# 发布记录（release-record）

> **用途**：为一次可交付版本留下「产物 ↔ 提交 ↔ tag」的完整证据链——半年后仍能回答"这个 JAR 里到底有什么、出事怎么回滚"。
> **填写时机**：S3 验证与交付阶段，**G4 发布闸门之前**必须填完第 1–9 节；执行 `.\tools\lab.ps1 tag` 之后再回填 tag 与 commit hash，并复查第 7 节。
> **产出位置**：`projects/P0N-<slug>/docs/release-record.md`，同时归档一份到 `docs/projects/P0N/release-record.md`。

## 1. 版本标识

| 项 | 值 |
|---|---|
| 项目 | P0N `<项目名>` |
| 版本 tag | `p0N-vX.Y.Z`（如 `p01-v1.0.0`） |
| commit hash | `<git rev-parse 输出的 40 位 hash>` |
| 分支 | `main`（由 `feat/P0N-<slug>` 以 `--no-ff` 合并） |
| 发布日期 | `<YYYY-MM-DD>` |
| 发布人 | `<角色代号，不写真名>` |
| 流水线资产版本 | `jfsl-scaffold@<X.Y.Z>`、`jfsl-verify@<X.Y.Z>` |

版本号自增规则：

| 变更性质 | 进位 | 例子 |
|---|---|---|
| 首版交付，或破坏兼容（API 契约 / 表结构 / 语义） | X | `p01-v1.0.0` → `p01-v2.0.0` |
| 新增需求（`REQ-*`）或新增接口 | Y | `p01-v1.0.0` → `p01-v1.1.0` |
| 缺陷修复、配置与文案修正 | Z | `p01-v1.1.0` → `p01-v1.1.1` |

tag 必须是**带注释 tag**（`git tag -a`），推送后**不得移动**；要改就发新版本。

## 2. 发布范围

只列本次真正包含的需求，未完成的写进「已知限制」。

| 需求编号 | 模块 | 内容摘要 | 对应测试用例 | 状态 |
|---|---|---|---|---|
| `REQ-<模块>-<序号>` | `<模块>` | `<一句话>` | `TC-<模块>-<场景>-<序号>` | 完成 |
| `BR-<模块>-<序号>` | `<模块>` | `<被实现的业务规则>` | `TC-<模块>-<场景>-<序号>` | 完成 |

排除项（本次**不做**，需显式登记）：`<REQ 编号 / 描述>`

## 3. 构建产物

| 产物 | 路径 | 大小 | 指纹（SHA256 前 12 位） |
|---|---|---|---|
| 后端 JAR | `backend/target/<artifactId>-<X.Y.Z>.jar` | `<NN.N> MB` | `<hash>` |
| 前端 dist | `frontend/dist/` | `<N.N> MB` | `<hash>` |
| 迁移脚本包 | `database/migrations/` | `<N KB>` | `<hash>` |

```powershell
# 记录产物指纹，防止"改了没重新打包"
Get-FileHash .\backend\target\p01-student-info-1.0.0.jar -Algorithm SHA256 |
  Select-Object -ExpandProperty Hash
(Get-ChildItem .\frontend\dist -Recurse -File | Measure-Object -Property Length -Sum).Sum / 1MB
```

```bash
# 确认字节码兼容 JDK 17（major version 必须是 61）
javap -v -cp backend/target/classes com.example.p01.P01Application.class | grep "major version"
```

## 4. 数据库迁移清单

全仓库共用一个库 `java_fullstack_lab`（不各自建库），本项目所有表以 `p0N_` 前缀隔离，
脚本放在 `database/migrations/V<版本>__<说明>.sql`，按版本号顺序执行、只增不改。
本项目的 SQL 中**不得出现其他项目的前缀**，也不得跨项目 JOIN 或建跨项目外键（门禁扫描这条）。

| 顺序 | 脚本 | 类型 | 作用对象 | 可回滚 | 空库验证 | 备注 |
|---|---|---|---|---|---|---|
| 1 | `V1__create_p01_tables.sql` | DDL | `p0N_*` 建表 | 是（DROP） | 通过 | 含索引与约束 |
| 2 | `V2__add_p0N_xxx_column.sql` | DDL | 加列 | 是（扩展阶段可留列） | 通过 | 见"扩展—迁移—收缩" |
| 3 | `V3__seed_p0N_dict.sql` | DML | 字典 seed | 是（DELETE） | 通过 | 幂等，可重复执行 |

```sql
-- 迁移脚本头部统一写明：目标库、前缀、可回滚方式
-- USE java_fullstack_lab;
-- 回滚：ALTER TABLE p0N_xxx DROP COLUMN yyy;
```

⚠️ 破坏性语句（`DROP TABLE` / `DROP COLUMN` / 类型收窄）**不得**与加列写在同一个版本里；先在上一版完成数据迁移，本版只收缩。

## 5. 配置变更清单

| 配置键 | 来源 | 默认值 | 是否必需 | 影响 |
|---|---|---|---|---|
| `DB_URL` | 环境变量 | 本地 `java_fullstack_lab` | 是 | 数据库地址 |
| `DB_USERNAME` / `DB_PASSWORD` | 环境变量 / `application-local.yml` | 无 | 是 | **不入库**，仓库只留 `.example` |
| `SERVER_PORT` | `application.yml` | `81NN` | 是 | 每项目独立端口 |
| `<新增业务开关>` | `application.yml` | `<值>` | 否 | `<说明>` |

## 6. 验证结果

| 验证项 | 命令 / 文件 | 结果 |
|---|---|---|
| 质量门禁 | `.\tools\lab.ps1 verify --project P0N` | 编译 / 测试 / 前端构建 / 秘密扫描 全绿 |
| 接口逐条核对 | `test/api-verification.md` | `<N>/<M> 通过` |
| 冒烟脚本 | `tools/smoke/p0N-smoke.ps1` | 通过 |
| 前端构建 | `pnpm build` | 无 error |
| 秘密扫描 | `lab.ps1 verify` 内含 | 命中 0 |

原始输出（脱敏后）粘贴在此，或写明日志归档路径：

```text
<verify 输出的尾部摘要>
```

## 7. 可追溯关系：产物 ↔ 提交 ↔ tag

三者必须能互相证明，缺一不可：

- **tag → 提交**：tag 指向的 commit 就是构建源码状态；
- **提交 → 产物**：产物由该 commit 的工作区构建，且 `git status --porcelain` 为空（无未提交改动参与构建）；
- **产物 → 提交**：产物指纹记在本文档，重新构建可得到同一 hash（Maven 需去时间戳）。

```bash
git cat-file -t p01-v1.0.0                    # 输出 tag 才正确；输出 commit 说明标签打错了
git rev-parse p01-v1.0.0^{commit}             # tag 解引用到的 commit hash
git show --stat p01-v1.0.0                    # tag 附注 + 本次变更文件清单
git tag -n99 -l "p01-v*"                      # 列出 P01 全部里程碑及注释
git log --oneline --no-merges p01-v1.0.0 -20  # 该里程碑之前的提交
```

变更文件清单必须与第 3、4 节的产物与迁移脚本对得上；对不上说明有未提交内容或被 `--amend` 的历史。

## 8. 已知限制

| 编号 | 限制 | 影响面 | 计划处理版本 |
|---|---|---|---|
| `LIM-01` | `<未实现的需求或已知缺陷>` | `<模块/用户>` | `p0N-vX.Y.Z` |

## 9. 回滚触发条件与操作入口

触发条件（满足任一即回滚，先执行再复盘）：

1. 冒烟脚本失败，或"登录 → 查询 → 新增"三个动作任一不可用；
2. 5xx 比例 > 1% 且持续 5 分钟以上；
3. 关键接口 P95 超过上一版本基线的 3 倍；
4. 出现数据写坏、重复扣减、状态机不可逆等正确性问题；
5. 迁移脚本执行后校验行数/唯一约束失败。

操作入口：见 `projects/P0N-<slug>/deploy/runbook.md` 的「回滚」一节（含命令、数据库回滚策略与演练记录）。

## 10. 发布前检查清单（G4 闸门）

- [ ] `.\tools\lab.ps1 verify --project P0N` 全绿，且输出已归档
- [ ] 字节码确认为 `major version: 61`（JDK 17 目标）
- [ ] 单元 / 集成测试与 `TC-*` 编号一一对应，无"只写不断言"的用例
- [ ] 前端 `pnpm build` 通过，dist 由**同一 commit** 产出
- [ ] 秘密扫描命中 0；`application-local.yml` 未被 `git ls-files` 命中
- [ ] `schema.sql` 无 `FOREIGN KEY` / `REFERENCES`；每个关联字段 `p0N_xxx_id` 都有普通索引与"引用不存在"用例
- [ ] 迁移脚本在**全新空库**上从零跑通（不是只在旧库上补跑）
- [ ] seed 脚本幂等：重复执行两次结果一致
- [ ] 前后端同一基线：产物由同一 commit 构建，无本地未提交修改
- [ ] 项目 `README.md` 的 Quick Start 在**陌生目录**实测过一遍
- [ ] 冒烟脚本通过，输出已粘贴到第 6 节
- [ ] 日志脱敏抽查：无密码、Token、连接串明文
- [ ] `CHANGELOG.md` 已按 Added / Changed / Fixed / Learning Focus 更新
- [ ] `test/api-verification.md` 已填并复核
- [ ] `deploy/runbook.md` 与 `deploy/nginx.conf`、systemd 单元文件随版本入库
- [ ] 回滚路径演练过一次（至少演练 JAR 回退）
- [ ] tag 已用 `git tag -a` 创建，注释含版本与需求范围

## 11. 示例（P01 学生信息管理系统 p01-v1.0.0）

| 项 | 值 |
|---|---|
| 项目 | P01 学生信息管理系统 |
| 版本 tag | `p01-v1.0.0` |
| commit hash | `4b7d3e1c9a2f5d8e0b6c4a7f1d3e5b9c2a8f0d64` |
| 分支 | `main`（由 `feat/P01-student-crud` 以 `--no-ff` 合并） |
| 发布日期 | `2025-03-18` |
| 流水线资产版本 | `jfsl-scaffold@1.2.0`、`jfsl-verify@1.1.0` |

发布范围：`REQ-STU-01` 学生 CRUD、`REQ-STU-02` 条件分页查询、`REQ-STU-03` 班级与专业下拉、
`BR-STU-04` 学号唯一（`UNIQUE KEY uk_p01_student_no`）；对应 `TC-STU-CREATE-01`、
`TC-STU-LIST-01`、`TC-STU-QUERY-02`。排除项：`REQ-STU-05` 成绩录入（见 `LIM-01`）。

产物：`backend/target/p01-student-info-1.0.0.jar` 42.7 MB，指纹 `9f2c1ab7d3e5`；
`frontend/dist/` 1.8 MB，指纹 `c81a04e7b62f`。

迁移：`V1__create_p01_tables.sql`（`p01_student` / `p01_class` / `p01_major`，含
`idx_p01_student_status_created` 联合索引）、`V3__seed_p01_major.sql`（8 条专业字典，幂等）；
两条脚本在空库 `java_fullstack_lab` 上从零执行通过。

配置：`SERVER_PORT=8101`、`DB_URL` / `DB_USERNAME` / `DB_PASSWORD` 走环境变量，
仓库内只有 `application-local.yml.example`。

验证：`verify --project P01` 全绿（38 个测试 / 0 失败 / 秘密扫描 0 命中）；
`test/api-verification.md` 24/24 通过；`tools/smoke/p01-smoke.ps1` 通过。

追溯：

```bash
git cat-file -t p01-v1.0.0     # -> tag
git show --stat p01-v1.0.0     # 变更 61 个文件，含 sql/V1__create_p01_tables.sql、deploy/runbook.md
```

已知限制：`LIM-01` 未实现成绩录入，计划 `p01-v1.1.0`；`LIM-02` 上传大小上限 5 MB，未做分片。

回滚触发条件：冒烟失败、5xx > 1%、列表接口 P95 > 600 ms、学号唯一约束被绕过。
入口：`deploy/runbook.md` §9；本次演练了 JAR 回退（从 v1.0.0 退到上一个构建，耗时 40 秒）。

## 填写检查清单

- [ ] 第 1 节 tag / commit / 日期与实际状态一致，无 `<...>` 残留
- [ ] 第 2 节的每个 `REQ-*` 都有对应 `TC-*`，未做的进了「已知限制」
- [ ] 第 3 节产物路径真实存在，指纹是本次构建算出来的
- [ ] 第 4 节每条迁移都注明"可回滚 / 不可回滚"与空库验证结论
- [ ] 第 7 节的 `git show --stat` 文件清单与第 3、4 节对得上
- [ ] 第 9 节回滚入口指向的 Runbook 章节真实存在
- [ ] 第 10 节清单全部勾上才允许执行 `lab.ps1 tag`
