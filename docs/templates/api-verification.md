# 接口验证记录（冒烟与契约核对）

> **用途**：用**真实启动的服务**逐条验证接口的输入、输出、状态码与错误码，并留下可复现的证据；同时确认"该失败的地方**确实**失败得体面"。
> **填写时机**：S3 验证与交付阶段，工序 19 之后、工序 20–23 之间**每批次填一份**；G4 发布闸门前必须存在最近一次全绿的记录。
> **产出位置**：`projects/P0N-<slug>/docs/testing/api-verification/AV-<批次>.md`；证据（响应体、截图、脚本输出）放 `projects/P0N-<slug>/test/evidence/AV-<批次>/`；脚本本体在 `tools/smoke/P0N.ps1`。

## 1. 头部字段

| 字段 | 必填 | 说明 |
|---|---|---|
| 验证批次 | 是 | `AV-<YYYYMMDD>-<序号>`，例：`AV-20250311-01`；同一批次内所有验证项共用环境与版本 |
| 环境 | 是 | 本机 / 测试库 / 预发布；写清库名（`java_fullstack_lab`）、端口（P01 为 8101）、profile |
| 服务版本 | 是 | `commit` 短哈希 + 里程碑 tag（如 `a1b2c3d` + `p01-v1.0.0`）；工作区有未提交改动时必须注明 |
| 启动方式 | 是 | 命令原文，例：`cd projects\P01-student-info\backend; .\mvnw.cmd spring-boot:run -Dspring-boot.run.profiles=local` |
| 依赖前置 | 是 | 迁移版本、种子数据、前端是否已起、外部依赖是否可用 |
| 验证人 / 日期 | 是 | `AI` / `USER` 或姓名；`YYYY-MM-DD` |
| 总体结论 | 是 | `PASS / FAIL`；FAIL 时写出阻断项编号 |

## 2. 验证项表格

| 端点 | 输入 | 期望 | 实际 | 结论 | 证据 |
|---|---|---|---|---|---|
| `GET /api/p01/students` | `page=1&pageSize=10` | `200`，`code=OK`，`data.total` 与库内行数一致 | `200`，`total=12`，与 `SELECT COUNT(*)` 一致 | PASS | `evidence/AV-20250311-01/list-page1.json` |
| `<METHOD> <path>` | `<参数或请求体>` | `<状态码 / code / 关键字段>` | `<实际>` | PASS/FAIL | `<文件路径>` |

要求：
- **一行一个验证点**，不要把"整个模块都测了"写成一行；
- 「输入」必须是可直接复制的字面量（含边界值：`pageSize=101`、`age=17`）；
- 「证据」必须是文件路径或命令原文，不接受"已确认"。

## 3. 冒烟命令清单（可直接照抄）

### 3.1 bash / Git Bash

```bash
BASE=http://localhost:8101
# 1) 健康检查
curl -s -o /dev/null -w "%{http_code}\n" "$BASE/actuator/health"

# 2) 新增学生（成功路径）
curl -s -X POST "$BASE/api/p01/students" \
  -H "Content-Type: application/json" \
  -d '{"studentNo":"S2025001","name":"Zhang San","age":20,"phone":"13800000001"}'

# 3) 条件分页查询
curl -s "$BASE/api/p01/students?page=1&pageSize=10&name=Zhang"

# 4) 单条查询
curl -s "$BASE/api/p01/students/1"

# 5) 修改
curl -s -X PUT "$BASE/api/p01/students/1" \
  -H "Content-Type: application/json" \
  -d '{"name":"Zhang San","age":21,"phone":"13800000001"}'

# 6) 删除
curl -s -X DELETE "$BASE/api/p01/students/1"
```

### 3.2 PowerShell（Windows 首选）

```powershell
$base = 'http://localhost:8101'
$body = @{ studentNo = 'S2025001'; name = 'Zhang San'; age = 20; phone = '13800000001' } | ConvertTo-Json

# 注意：PowerShell 5.1 里 curl 是 Invoke-WebRequest 的别名，要真 curl 请写 curl.exe
$created = Invoke-RestMethod -Method Post -Uri "$base/api/p01/students" -ContentType 'application/json' -Body $body
$created | ConvertTo-Json -Depth 5   # { "code":0, "message":"ok", "data":1 }

$list = Invoke-RestMethod -Uri "$base/api/p01/students?page=1&pageSize=10"
$list.data.total
```

### 3.3 带 JWT 的请求（P06 起适用）

```bash
BASE=http://localhost:8106
TOKEN=$(curl -s -X POST "$BASE/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"<REDACTED>"}' | jq -r '.data.token')

curl -s "$BASE/api/p06/articles?page=1&pageSize=10" -H "Authorization: Bearer $TOKEN"

# 无 token：期望 401
curl -s -o /dev/null -w "%{http_code}\n" "$BASE/api/p06/articles"
```

```powershell
$login = Invoke-RestMethod -Method Post -Uri "$base/api/auth/login" -ContentType 'application/json' `
  -Body (@{ username = 'admin'; password = $env:LAB_ADMIN_PASSWORD } | ConvertTo-Json)
$headers = @{ Authorization = "Bearer $($login.data.token)" }
Invoke-RestMethod -Uri "$base/api/p06/articles?page=1&pageSize=10" -Headers $headers
```

> 口令与密钥**不写进本文件**：从环境变量（`$env:LAB_ADMIN_PASSWORD`）读取，记录里只写 `<REDACTED>`。

## 4. 认证与权限验证矩阵

| # | 场景 | 构造方式 | 期望状态码 | 期望业务 code | 期望副作用 |
|---|---|---|---|---|---|
| 1 | 未登录 | 不带 `Authorization` | `401` | `40100` 未认证 | 无数据变化 |
| 2 | 已登录无权限 | 普通用户 token 调管理端点 | `403` | `40300` 无权限 | 无数据变化 |
| 3 | 已登录有权限 | 管理员 token 调管理端点 | `200` | `0` | 按业务生效 |
| 4 | 越权访问他人数据 | 用户 A 的 token 访问用户 B 的资源 ID | `403`（或 `404` 以便不泄露存在性，**二选一并写死**） | `40300` / `40400` | **B 的数据必须原样不变** |
| 5 | token 过期 | 使用过期 token | `401` | `40101` token 过期 | 无数据变化 |
| 6 | token 被篡改 | 改签名末位 | `401` | `40102` token 无效 | 无数据变化 |

> P01–P05 尚未引入 JWT（README 说明从 P06 开始），其验证记录中本矩阵标 `N/A`，但必须写明理由，不能留空。

## 5. 错误路径验证清单

| 错误类型 | 构造输入 | 期望 | 重点核查 |
|---|---|---|---|
| 参数校验失败 | `age=17`、`name` 为空、`phone=138` | `400` + `code=PARAM_INVALID` + 字段级 `message` | 不是 500；错误信息不含堆栈与 SQL |
| 资源不存在 | `GET /api/p01/students/999999` | `404` + `code=STUDENT_NOT_FOUND` | 不返回 `200 + data:null` |
| 状态冲突 | P02 对已归还借阅单再次归还 | `409` + 状态冲突码 | 不静默成功；不重复加库存 |
| 重复提交 | 连续两次相同 `studentNo` | 第二次 `409` + `code=STUDENT_NO_DUPLICATED` | 库里只有 1 行；不产生第二条 |
| 超大分页 | `pageSize=101` | `400` + `code=PARAM_INVALID`（或按约定截断为 100，**必须写死一种**） | 不允许把 `pageSize=100000` 直接打到 SQL |
| 非法分页 | `page=0`、`page=-1` | `400` 或归一为 1（**写死一种**） | 不出现 `LIMIT -10` 类异常 |
| 类型错误 | `age="abc"`、`id="xyz"` | `400` + `code=PARAM_INVALID` | 不是 `500 MethodArgumentTypeMismatchException` 裸抛 |
| 逻辑外键引用不存在 | 新增借阅时 `bookId=999999` | `404` + 引用不存在业务码 | 不是 500；**不产生孤儿行**（无物理外键兜底，必须应用层拦住） |
| 删除仍被引用的父记录 | 删除仍有在借记录的图书 | `409` + 引用检查业务码 | 不静默删除、不留下孤儿 `p02_borrow` 行 |
| 非法状态迁移 | P02 `RETURNED → BORROWED` | `409` | 状态机被真正执行 |
| 超长字符串 | `name` 200 字符 | `400` | 不被数据库截断后静默入库 |

## 6. 「必须失败也要验证」

**只验证成功路径等于没验证。** 每个失败用例要确认三件事：

1. 返回的是**业务错误码**（`Result.code != 0`），而不是 `500` 加一段 Java 堆栈；
2. HTTP 状态码与错误类别匹配（校验 400 / 未认证 401 / 无权限 403 / 不存在 404 / 冲突 409）；
3. 数据库**没有留下半成品数据**（事务回滚生效），且**没有孤儿行**（本项目逻辑外键的 `orphan-check.sql` 返回 0 行），日志中**没有**手机号、口令等敏感明文。

判定反例（一票否决）：

```text
HTTP/1.1 500
{"timestamp":"2025-03-11T10:00:00Z","status":500,"error":"Internal Server Error",
 "trace":"com.mysql.cj.jdbc.exceptions.SQLError: Duplicate entry 'S2025001' ..."}
```

期望形态：

```text
HTTP/1.1 409
{"code":10003,"message":"student no already exists: S2025001","data":null}
```

## 7. 示例：P01 学生模块完整验证记录

### AV-20250311-01 P01 学生模块冒烟

| 项 | 值 |
|---|---|
| 环境 | 本机 Windows，库 `java_fullstack_lab`，端口 `8101`，profile `local` |
| 服务版本 | commit `a1b2c3d`，tag `p01-v1.0.0`（工作区无未提交改动） |
| 启动方式 | `cd projects\P01-student-info\backend; .\mvnw.cmd spring-boot:run -Dspring-boot.run.profiles=local` |
| 依赖前置 | 迁移 `V1__init_student.sql` 已执行；`seed.sql` 已导入 12 行；前端 `pnpm dev` 已起（联调用例用） |
| 分层测试前置 | `.\tools\lab.ps1 verify --project P01` 已通过（工序 19 结论 PASS） |
| 总体结论 | PASS（认证矩阵 N/A：P01 未引入 JWT，见 README「新增技术点」） |

| # | 端点 | 输入 | 期望 | 实际 | 结论 | 证据 |
|---|---|---|---|---|---|---|
| 1 | `GET /actuator/health` | 无 | `200`，`status=UP` | `200`，`UP`，MySQL 连通 | PASS | `evidence/AV-20250311-01/health.txt` |
| 2 | `POST /api/p01/students` | `S2025001, Zhang San, 20, 13800000001` | `200`，`code=OK`，`data=13` | `200`，`code=OK`，`data=13` | PASS | `create-ok.json` |
| 3 | `POST /api/p01/students` | 同一 `studentNo` 再提交 | `409`，`code=STUDENT_NO_DUPLICATED` | `409`，`code=STUDENT_NO_DUPLICATED`，库内仍 13 行 | PASS | `create-dup.json` |
| 4 | `POST /api/p01/students` | `age=17` | `400`，`code=PARAM_INVALID`，message 指向 `age` | `400`，`code=PARAM_INVALID`，`age must be between 18 and 120` | PASS | `create-age17.json` |
| 5 | `GET /api/p01/students` | `page=1&pageSize=10` | `200`，`total=13` | `200`，`total=13`，10 条 | PASS | `list-page1.json` |
| 6 | `GET /api/p01/students` | `pageSize=101` | `400`，`code=PARAM_INVALID` | `400`，`code=PARAM_INVALID`，`pageSize must be <= 100` | PASS | `list-oversize.json` |
| 7 | `GET /api/p01/students` | `page=0` | 归一为第 1 页，`200` | `200`，返回第 1 页 | PASS | `list-page0.json` |
| 8 | `GET /api/p01/students/999999` | 不存在的 id | `404`，`code=STUDENT_NOT_FOUND` | `404`，`code=STUDENT_NOT_FOUND` | PASS | `get-404.json` |
| 9 | `PUT /api/p01/students/13` | `age=21` | `200`，`code=OK`，库内 `age=21` | `200`，库内 `age=21`，`updated_at` 变化 | PASS | `update-ok.json` |
| 10 | `PUT /api/p01/students/13` | `age=121` | `400`，`code=PARAM_INVALID`，**`updated_at` 不变** | `400`，`updated_at` 未变 | PASS | `update-age121.json` |
| 11 | `DELETE /api/p01/students/13` | 已存在 id | `200`，`code=OK`，库内行数 12 | `200`，行数 12 | PASS | `delete-ok.json` |
| 12 | `DELETE /api/p01/students/13` | 重复删除 | `404`，`code=STUDENT_NOT_FOUND` | `404`，`code=STUDENT_NOT_FOUND` | PASS | `delete-again.json` |
| 13 | `POST /api/p01/students` | `age="abc"` | `400`，`code=PARAM_INVALID`，无堆栈 | `400`，`code=PARAM_INVALID`，日志无堆栈 | PASS | `create-badtype.json` |
| 14 | 认证矩阵 #1–#6 | — | — | — | N/A | P01 无 JWT，见 `README.md` 技术新增点 |

**越权与敏感信息核查**：日志中手机号按 `138****0001` 脱敏输出（`handler` 层已配置），响应体不含 `deleted`/`updated_at` 等内部字段。抽查 3 条日志确认。

**逻辑外键核查**：P01 的 `class_id` 为逻辑外键（无物理外键），已执行"新增时 `classId=999999`"→ `404` + 引用不存在业务码；`test/orphan-check.sql` 返回 0 行。

**未通过项与整改**：无。

## 8. 空白副本（可整段复制）

```markdown
# <AV-YYYYMMDD-序号> <模块> 接口验证记录

| 项 | 值 |
|---|---|
| 环境 | <本机 / 测试库 / 预发布，库名 + 端口 + profile> |
| 服务版本 | commit <短哈希> + tag <p0N-vX.Y.Z>（工作区<有/无>未提交改动） |
| 启动方式 | <命令原文> |
| 依赖前置 | <迁移版本、种子、前端、外部依赖> |
| 验证人 / 日期 | <AI / USER> / <YYYY-MM-DD> |
| 总体结论 | <PASS / FAIL + 阻断项编号> |

| # | 端点 | 输入 | 期望 | 实际 | 结论 | 证据 |
|---|---|---|---|---|---|---|
| 1 | <METHOD /path> | <字面量输入> | <状态码 / code / 字段> | <实际> | <PASS/FAIL> | `<证据文件>` |

**认证与权限矩阵**：<逐行填，或写 N/A + 理由>
**错误路径核查**：<参数校验 / 不存在 / 冲突 / 重复提交 / 超大分页 的结果>
**必须失败也要验证**：<是否确认是业务错误码、是否回滚、是否脱敏>
**未通过项与整改**：<编号 + 动作 + 责任人 + 复验日期>
```

## 填写检查清单

- [ ] 批次、环境、commit + tag、启动命令四要素齐全，他人可照抄复现
- [ ] 每个验证项一行，输入是字面量，证据是可打开的文件路径
- [ ] 认证与权限矩阵逐行填写，或写明 `N/A` 与理由（P01–P05 属此类）
- [ ] 错误路径至少覆盖：校验失败、资源不存在、状态冲突、重复提交、超大分页
- [ ] 「必须失败也要验证」已确认：业务错误码、事务回滚、日志脱敏三项
- [ ] 敏感信息（口令、完整手机号、密钥）在记录中为 `<REDACTED>` 或已脱敏
- [ ] 逻辑外键已核查：引用不存在路径有记录，`orphan-check.sql` 返回 0 行
- [ ] 未通过项写明了整改动作与复验日期，且与 `progress-ledger.md` 的闸门记录一致
