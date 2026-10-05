# 部署与回滚 Runbook（runbook）

> **用途**：一份"换一台机器也能照做"的部署与回滚手册——照抄命令即可从零到可访问，出事能按同一份文档退回去。
> **填写时机**：S3 验证与交付阶段，**G4 发布闸门之前**写完并在干净环境实测一遍；发布有配置变更时同步更新。
> **产出位置**：`projects/P0N-<slug>/deploy/runbook.md`（同目录放 `nginx.conf`、`<unit>.service`、`docker-compose.yml`）。
> **验收底线是本地可复现**：照本文件在干净机器上做完能访问即算通过，**不要求真实服务器**；域名与 HTTPS 推迟。

## 1. 前置条件
| 组件 | 最低版本 | 检查命令 | 说明 |
|---|---|---|---|
| JDK | 17 | `java -version` | 编译目标 17，运行 17+ 均可 |
| MySQL | 8.0 | `mysql --version` | 单库 `java_fullstack_lab`，`utf8mb4` |
| Nginx | 1.24 | `nginx -v` | 静态资源 + `/api` 反向代理 |
| Node.js / pnpm | 20 / 9 | `node -v` / `pnpm -v` | 仅构建前端时需要 |

账号要求（**只写权限要求，不写任何真实凭据**）：

- 部署账号 `<deploy-user>`：普通用户，`sudo` 只放行 `systemctl restart/status <unit>` 与 `nginx -s reload`；**不得用 `root` 直接跑应用**。
- 数据库账号 `<db-user>`：只对 `java_fullstack_lab` 有权限、只操作本项目 `p0N_*` 表；不得授 `ALL ON *.*`。
- 应用以专用低权限账号 `jfsl-p0N`（无登录 shell）运行；配置目录 `600` 且归属该账号。

## 2. 环境变量清单（示例值均为占位，**不写真实值**）
| 变量名 | 用途 | 示例值 | 是否必需 |
|---|---|---|---|
| `DB_URL` | JDBC 连接串 | `jdbc:mysql://127.0.0.1:3306/java_fullstack_lab?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai` | 是 |
| `DB_USERNAME` | 数据库账号 | `<db-user>` | 是 |
| `DB_PASSWORD` | 数据库口令 | `<从密钥来源注入>` | 是 |
| `SERVER_PORT` | 本项目端口（`81NN`） | `8101` | 是 |
| `SPRING_PROFILES_ACTIVE` | 生效环境 | `local` | 是 |
| `FILE_STORAGE_ROOT` | 本地文件存储根目录 | `/var/lib/jfsl/p01/files` | 否（用到上传才需要） |

真实值只写进已被 gitignore 的 `backend/src/main/resources/application-local.yml`，或权限 `600` 的
`/etc/jfsl/p0N/p0N.env`；仓库内只保留 `*.example`。日志必须脱敏，禁止打印口令与 Token。

## 3. 数据库初始化与迁移
全仓库**只有一个库** `java_fullstack_lab`，十个项目共用（不各自建库）；本项目所有表以 `p0N_` 前缀隔离。

```sql
CREATE DATABASE IF NOT EXISTS java_fullstack_lab
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

CREATE USER IF NOT EXISTS '<db-user>'@'127.0.0.1' IDENTIFIED BY '<password-from-secret-store>';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX
  ON java_fullstack_lab.* TO '<db-user>'@'127.0.0.1';
```

迁移脚本放在 `database/migrations/V<版本>__<说明>.sql`，按版本号顺序执行：

```bash
for f in $(ls database/migrations/V*.sql | sort -V); do
  mysql -h127.0.0.1 -u"$DB_USERNAME" -p java_fullstack_lab < "$f"
done
```

铁律与自查：

- 本项目 SQL **不得出现其他项目的前缀**，禁止跨项目 JOIN（门禁自动扫描）；
- 约束前缀 `uk_` / `idx_` / `ck_`，字段 `snake_case`，布尔 `is_` 前缀，时间 `_at` 后缀；
- **禁止物理外键**：不得出现 `FOREIGN KEY` / `REFERENCES` / `ON DELETE CASCADE`；关联用 `<被引用表单数>_id`
  （如 `p01_student.class_id`）表达并**必须建普通索引**；
- 引用完整性由应用层保证：写入前校验被引用记录存在，删除父记录前检查子记录并返回业务错误码；
- 迁移必须在**全新空库**上从零跑通；seed 必须幂等，连跑两次结果一致。

```sql
-- 孤儿数据排查（无物理外键，只能靠 SQL 兜底；本项目 test/orphan-check.sql 保留一份）
SELECT s.id, s.class_id FROM p01_student s
  LEFT JOIN p01_class c ON c.id = s.class_id
 WHERE s.class_id IS NOT NULL AND c.id IS NULL;
```

## 4. 后端部署
目录规划：`/opt/jfsl/p0N/app/`（JAR，文件名带版本号，**不覆盖旧文件**）、`/etc/jfsl/p0N/`（配置与 `p0N.env`，`600`）、
`/var/www/p0N/`（前端）、`/var/log/jfsl/p0N/`（日志）。

```bash
cd projects/P01-student-info/backend
./mvnw -q clean package          # 产物必须来自 main 上的目标 tag
```

先前台试跑一次（`set -a` 把 `p0N.env` 读进环境），确认能起再交给 systemd：

```bash
set -a; . /etc/jfsl/p01/p01.env; set +a
/usr/bin/java -jar /opt/jfsl/p01/app/p01-student-info-1.0.0.jar --spring.config.additional-location=/etc/jfsl/p01/application-local.yml
```

`/etc/systemd/system/jfsl-p01.service`（专用低权限账号 + 配置与 JAR 分离）；改完执行
`sudo systemctl daemon-reload && sudo systemctl enable --now jfsl-p01`：

```ini
[Unit]
Description=JFSL P01 student-info backend
After=network-online.target mysql.service

[Service]
Type=simple
User=jfsl-p01
WorkingDirectory=/opt/jfsl/p01/app
EnvironmentFile=/etc/jfsl/p01/p01.env
ExecStart=/usr/bin/java -Xms256m -Xmx512m -jar /opt/jfsl/p01/app/p01-student-info-1.0.0.jar --spring.config.additional-location=/etc/jfsl/p01/application-local.yml
SuccessExitStatus=143
Restart=on-failure
StandardOutput=append:/var/log/jfsl/p01/app.log
StandardError=append:/var/log/jfsl/p01/app.err.log
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

`/etc/logrotate.d/jfsl-p01`（配合 `StandardOutput=append` 用 `copytruncate`，否则日志永不清理）：

```conf
/var/log/jfsl/p01/*.log {
    daily
    rotate 14
    compress
    copytruncate
}
```

## 5. 前端部署
```bash
cd projects/P01-student-info/frontend
pnpm install --frozen-lockfile && pnpm build
sudo mkdir -p /var/www/p01/releases/1.0.0 && sudo cp -r dist/* /var/www/p01/releases/1.0.0/
sudo ln -sfn /var/www/p01/releases/1.0.0 /var/www/p01/dist   # 回滚只改软链
```

`/etc/nginx/conf.d/p01.conf`（占位域名；静态资源 + `/api` 反代 + 上传大小 + gzip），改完执行
`sudo nginx -t && sudo nginx -s reload`：

```nginx
server {
    listen 80;
    server_name p01.<your-domain>;

    root /var/www/p01/dist;
    client_max_body_size 10m;        # 与后端 multipart 上限保持一致

    gzip on;

    location / {
        try_files $uri $uri/ /index.html;      # 前端 history 路由
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8101/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 30s;
    }
}
```

## 6. 可选：Docker Compose
**这是可选交付物**：CI 与本地验收都不依赖它，它只是让没装 MySQL/Nginx 的机器能快速起一套，不要让它成为唯一入口。

```yaml
services:
  mysql:
    image: mysql:8.0
    environment: { MYSQL_ROOT_PASSWORD: "${DB_PASSWORD:?set-in-env}", MYSQL_DATABASE: java_fullstack_lab }
    volumes: ["mysql-data:/var/lib/mysql"]
  backend:
    image: eclipse-temurin:17-jre
    command: ["java", "-jar", "/app/app.jar"]
    environment:
      DB_URL: jdbc:mysql://mysql:3306/java_fullstack_lab?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
      DB_USERNAME: ${DB_USERNAME:?set-in-env}
      DB_PASSWORD: ${DB_PASSWORD:?set-in-env}
    ports: ["8101:8101"]
    volumes: ["./backend/target/p01-student-info-1.0.0.jar:/app/app.jar:ro"]
volumes:
  mysql-data:
```

## 7. 健康检查（存活与就绪分开）
| 探针 | 端点 | 判据 | 失败时的动作 |
|---|---|---|---|
| 存活 liveness | `/api/actuator/health/liveness` | `{"status":"UP"}` | 进程假死 → `systemctl restart` |
| 就绪 readiness | `/api/actuator/health/readiness` | `{"status":"UP"}` | 依赖（DB）未就绪 → 不接流量，先查 DB |
| 反向代理 | 同上端点经 Nginx 访问 | 200 | 代理配置或上游端口问题 |

```bash
curl -fsS http://<app-domain>/api/actuator/health/liveness  && echo " liveness OK"
curl -fsS http://<app-domain>/api/actuator/health/readiness && echo " readiness OK"
```

`application.yml` 需开启 `management.endpoint.health.probes.enabled=true` 与 `management.endpoints.web.exposure.include=health`。

## 8. 验证步骤（从真实入口走三个动作）
```bash
BASE=http://<app-domain>

# 1) 登录 -> 期望 code=200 且 data.token 非空；失败先看 /var/log/jfsl/p0N/app.log
curl -fsS -X POST "$BASE/api/auth/login" -H 'Content-Type: application/json' \
  -d '{"username":"<demo-user>","password":"<demo-pass>"}'

# 2) 查询 -> 期望 data.total > 0，data.list 长度 <= 10
curl -fsS "$BASE/api/students?page=1&pageSize=10&status=1" -H "Authorization: Bearer $TOKEN"

# 3) 新增 -> 期望 code=200；重复学号返回业务错误码（唯一约束 uk_p01_student_no）
curl -fsS -X POST "$BASE/api/students" -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' -d '{"studentNo":"<S0001>","name":"<示例姓名>","classId":1}'
```

三个动作还要在浏览器里手工走一遍（打开 `$BASE` → 登录 → 列表能搜 → 新增后列表出现新记录），
这是 G4 的验收动作，不能用 curl 代替。另需验证失败路径：`classId` 传不存在的班级时返回
"引用不存在"业务错误码且不写入任何行（逻辑外键由应用层保证）。

## 9. 回滚
触发条件（满足任一即回滚，事后补 `incident-record.md`）：

1. 冒烟或第 8 节三个动作任一失败，且 10 分钟内定位不到可立即修复的点；
2. 5xx 比例 > 1% 持续 5 分钟；
3. 关键接口 P95 超过上一版本基线的 3 倍；
4. 数据正确性受损（重复扣减、状态不可逆、唯一约束被绕过、出现孤儿数据）；
5. 迁移后校验失败，或 SQL 里出现其他项目的前缀。

```bash
# 后端回滚：新旧 JAR 并存，只改 unit 指向旧版本
sudo systemctl stop jfsl-p01
sudo sed -i 's#p01-student-info-1\.0\.0\.jar#p01-student-info-0.9.3.jar#' /etc/systemd/system/jfsl-p01.service
sudo systemctl daemon-reload && sudo systemctl start jfsl-p01

# 前端回滚：只改软链
sudo ln -sfn /var/www/p01/releases/0.9.3 /var/www/p01/dist && sudo nginx -s reload
```

数据库回滚策略——与「扩展—迁移—收缩」配合，**禁止在回滚里删列删表**：

| 阶段 | 做法 | 回滚动作 |
|---|---|---|
| 扩展 | 先加新列/新表，允许为空 | 无需回滚，留列即可（老代码不读它） |
| 迁移 | 双写或批量刷数据，校验一致 | 保留新列，数据不动 |
| 收缩 | 下一个版本才删旧列/旧表 | 只回滚代码，不回滚结构 |

回滚演练记录：`<YYYY-MM-DD> 在 <环境> 演练 JAR 回退 + 软链回退，耗时 <N> 秒，结果 <通过>`。

## 10. 常见故障对照表
| 现象 | 优先检查哪一层 | 处置 |
|---|---|---|
| 页面 404 / 刷新路由 404 | Nginx 静态根与 `try_files` | 确认 `root` 指向当前软链；补 `try_files $uri $uri/ /index.html` |
| 页面能开但接口 502 | Nginx → 后端进程 | `systemctl status`；`curl 127.0.0.1:81NN/api/...` 判断是进程还是代理 |
| 接口 404 但后端在跑 | 上下文路径 / 代理前缀 | 对齐 `/api/` 与后端 `server.servlet.context-path`，注意结尾斜杠 |
| 接口 401 / 403 | Token 与拦截器 | 看过期时间、`Authorization` 头、拦截器放行清单 |
| 启动即退出 | 环境变量与配置 | `journalctl -u jfsl-p01 -n 100`；检查 `EnvironmentFile` 是否被读到 |
| `Access denied for user` | 数据库账号授权 | 核对账号主机段（`@'127.0.0.1'`）与 `GRANT` 范围 |
| 上传 413 | Nginx `client_max_body_size` | 与后端 `spring.servlet.multipart.max-file-size` 一起调 |
| 删除父记录后出现孤儿数据 | 应用层引用校验 | 跑 `test/orphan-check.sql`；补删除前的子记录校验与错误码 |
| 列表越翻越慢 | 索引与 SQL | 看 `EXPLAIN`，补联合索引（详见 `incident-record.md` 示例） |

## 11. 示例（P01 学生信息管理系统，已填好）
- 应用目录 `/opt/jfsl/p01/app/p01-student-info-1.0.0.jar`；配置目录 `/etc/jfsl/p01/`（`600`）；静态目录
  `/var/www/p01/dist` → `releases/1.0.0`；端口 `8101`；占位域名 `p01.<your-domain>`；单元 `jfsl-p01.service`
  （`User=jfsl-p01`）；库 `java_fullstack_lab` / 前缀 `p01_`。
- 实测记录：干净机器上从建库到第 8 节三个动作全部通过，约 25 分钟（`pnpm install` 占 6 分钟）；空库执行
  `V1__create_p01_tables.sql` 与 `V3__seed_p01_major.sql` 通过；`nginx -t` 通过；JAR 回退演练 40 秒。
  已知偏差：Nginx 运行账号对 `/var/www/p01/releases` 需可读执行权限，否则 403（步骤已补 `chmod`）；
  `classId` 引用不存在的班级时后端原本直接写库产生孤儿数据，已改为应用层校验并补 `TC-STU-CREATE-04`。

## 填写检查清单
- [ ] 第 1 节写的是"权限要求"，没有任何真实凭据
- [ ] 第 2 节变量表里**没有真实值**，示例值全部为占位
- [ ] 第 3 节含建库、授权、迁移执行、空库验证、前缀自查与孤儿数据排查 SQL
- [ ] 第 3 节确认 `schema.sql` 无 `FOREIGN KEY` / `REFERENCES` / `ON DELETE CASCADE`
- [ ] 第 4 节 systemd 单元用专用低权限账号，配置与 JAR 分离，日志有轮转
- [ ] 第 5 节 `nginx.conf` 含静态资源、`/api` 反代、上传大小、gzip 四项
- [ ] 第 6 节明确标注 Docker Compose 为**可选**交付物
- [ ] 第 7 节存活与就绪是两个端点，各有处置动作
- [ ] 第 8 节验证步骤能在浏览器里从头走通，含"引用不存在"的失败路径
- [ ] 第 9 节回滚触发条件 ≥ 4 条，数据库回滚与"扩展—迁移—收缩"一致
- [ ] 第 10 节对照表覆盖 Nginx / 应用 / DB 三层
- [ ] 全文无真实主机名、真实 IP、真实域名、真实账号
