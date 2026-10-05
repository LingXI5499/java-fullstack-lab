# 项目进度与阶段账本（Progress Ledger）

> **用途**：维护 S0–S3 共 23 个工序的状态表、四道闸门的判定记录、当前阻塞点、变更与返工度量——一眼看出"现在卡在哪一步、下一步该干什么"。
> **填写时机**：S0 开工即建，**每完成一个工序就更新一次**（不许攒着批量补）；G1–G4 判定当天必须回填闸门记录。
> **产出位置**：`projects/P0N-<slug>/docs/progress.md`（一个项目一份）；度量数字每轮回填到 `docs/pipeline/metrics.md`。
> 工序编号与产出物的权威定义见 [../pipeline/delivery-pipeline.md](../pipeline/delivery-pipeline.md) §2，本文件不得自行改编号。

> **与 `discovery-record.md` 的分工**：本文件回答"**走到哪一步、闸门过没过、返工几次**"；
> 发现类事实（脏数据、接口与契约不符）写在 `discovery.md`，本文件只在「变更记录」里引用其编号。

## 1. 填写口径（先统一，再填表）

| 字段 | 取值 | 判定标准 |
|---|---|---|
| 状态 | `OPEN` | 未开始；无产出物 |
| | `IN_PROGRESS` | 已动手，产出物未齐；**同一项目同时最多 2 个 IN_PROGRESS**（1 个主工序 + 1 个评审/返工） |
| | `DONE` | 产出物已提交，且该工序的"完成判据"逐条为真 |
| | `CANCELLED` | 本项目确定不做，必须写明理由与 ADR 编号 |
| 负责人 | `AI` / `USER` / `BOTH` | 按 `AGENTS.md` §5 的分工；`BOTH` 表示 AI 出骨架、用户写业务代码 |
| 产出物 | 路径 | 必须是仓库内真实存在的文件；没有文件不算完成（铁律二） |
| 门禁状态 | `G1 / G2 / G3 / G4` | 该工序归属哪道闸门的判定范围；非闸门工序填 `—` |
| 完成日期 | `YYYY-MM-DD` | 只在状态变为 `DONE` 时填；返工不清空原日期，另记在「变更记录」 |

**不变量**：任一时刻 `projects/` 下只能有**一个**处于 `IN_PROGRESS` 的项目（铁律一）。若本表出现第二个项目的账本，先停手并说明原因。

> **编号口径**（与 `AGENTS.md` §3.7 一致）：工序号 01–23 全仓库统一，不得自增自减；
> 表里引用的编号必须写全称——`REQ-STU-01`、`BR-STU-02`、`AC-STU-03`、`TC-STU-CREATE-02`、`TASK-STU-01`、`NFR-PERF-01`；
> 模块段取项目 `requirements.md` 第 0 节模块字典，全大写、序号两位补零（不写 `REQ-stu-1`，也不写光秃秃的 `STU`）。
> NFR 分类只用固定 8 类缩写：`PERF` / `SEC` / `AVAIL` / `RELIAB` / `MAINT` / `SCALE` / `COMPAT` / `OBS`。

> **路径约定**：S0–S1 的规格与设计文档按 `delivery-pipeline.md` §2 放 `projects/P0N-<slug>/docs/`；
> S2–S3 的产出物随代码放 `projects/P0N-<slug>/`（`backend/` `frontend/` `database/` `test/` `deploy/` `docs/`）。
> 同一份内容只留一处，禁止两处各写一份。

## 2. 阶段账本表（23 工序）

| 阶段 | 工序 | 工序名 | 状态 | 负责人 | 产出物 | 门禁 | 完成日期 |
|---|---|---|---|---|---|---|---|
| S0 | 01 | 项目章程 | OPEN | AI | `projects/P0N-<slug>/docs/charter.md` | G1 | — |
| S0 | 02 | 需求卡 | OPEN | BOTH | `projects/P0N-<slug>/docs/requirements.md` | G1 | — |
| S0 | 03 | 业务规则 | OPEN | BOTH | `projects/P0N-<slug>/docs/requirements.md`（规则节） | G1 | — |
| S0 | 04 | 验收条件 | OPEN | BOTH | `projects/P0N-<slug>/docs/requirements.md`（验收节） | G1 | — |
| S0 | 05 | 非功能需求 | OPEN | BOTH | `projects/P0N-<slug>/docs/requirements.md`（NFR 节） | G1 | — |
| S1 | 06 | 数据设计 | OPEN | AI | `projects/P0N-<slug>/docs/design.md`（数据节） | G2 | — |
| S1 | 07 | API 契约 | OPEN | AI | `projects/P0N-<slug>/docs/design.md`（接口节） | G2 | — |
| S1 | 08 | 模块与包结构 | OPEN | AI | `projects/P0N-<slug>/docs/design.md`（结构节） | G2 | — |
| S1 | 09 | 页面与状态 | OPEN | AI | `projects/P0N-<slug>/docs/design.md`（前端节） | G2 | — |
| S1 | 10 | 决策记录 | OPEN | AI | `projects/P0N-<slug>/docs/adr/ADR-*.md` | G2 | — |
| S2 | 11 | 脚手架 | OPEN | AI | `projects/P0N-<slug>/backend/` `frontend/` `deploy/` | — | — |
| S2 | 12 | 迁移与种子 | OPEN | AI | `database/migrations/V*.sql` `database/seed.sql` | — | — |
| S2 | 13 | Entity / DTO / VO / Query | OPEN | AI | `backend/src/main/java/<base>/entity|dto|vo|query/` | — | — |
| S2 | 14 | Mapper + XML | OPEN | USER | `mapper/` + `resources/mapper/` | — | — |
| S2 | 15 | Service + 事务 | OPEN | USER | `service/` + `service/impl/` | — | — |
| S2 | 16 | Controller | OPEN | AI | `controller/` | — | — |
| S2 | 17 | 异常 / 日志 / 认证 | OPEN | AI | `exception/` `handler/` `interceptor/` `aspect/` | — | — |
| S2 | 18 | 前端页面与联调 | OPEN | USER | `frontend/src/views/` `frontend/src/api/` | G3 | — |
| S3 | 19 | 分层测试 | OPEN | BOTH | `src/test/` `docs/testing/` | — | — |
| S3 | 20 | 冒烟脚本 | OPEN | AI | `tools/smoke/P0N.ps1` | — | — |
| S3 | 21 | 打包 | OPEN | AI | `backend/target/*.jar` `frontend/dist/` | — | — |
| S3 | 22 | 部署配置与 Runbook | OPEN | AI | `deploy/` | — | — |
| S3 | 23 | 打 tag 与归档 | OPEN | BOTH | 带注释 tag `p0N-vX.Y.Z` + 发布记录 | G4 | — |

复盘升级（24–26，不计入 23 工序，但 G4 之后必须完成才允许开下一个项目）：

| 步骤 | 名称 | 状态 | 产出物 |
|---|---|---|---|
| 24 | 项目复盘 | OPEN | `projects/P0N-<slug>/docs/retrospective.md` |
| 25 | 技能升级提案 | OPEN | `docs/pipeline/upgrade-log.md` 新增条目 |
| 26 | 度量更新 | OPEN | `docs/pipeline/metrics.md` |

## 3. 门禁记录表

| 闸门 | 判定范围 | 判定日期 | 结论 | 未通过项 | 整改动作 | 复验日期 |
|---|---|---|---|---|---|---|
| G1 规格 | 工序 01–05 | — | — | — | — | — |
| G2 设计 | 工序 06–10 | — | — | — | — | — |
| G3 开发 | 工序 13–18（DoD 8 项） | — | — | — | — | — |
| G4 发布 | 工序 19–23 + 发布阻断 5 条件 | — | — | — | — | — |

判定结论只有 `PASS` / `FAIL`；`FAIL` 必须填写"未通过项"（引用 `AC-` / `BR-` / `TC-` / `DISC-` 编号）与整改责任人。
闸门不通过就不进入下一阶段——不许用"先做着，回头补"绕过。

## 4. 当前卡在哪里

> 每次更新账本，必须把这一节重写一遍（它是给人看的唯一入口）。

- **一句话现状**：<当前处于哪个工序、在做什么、还差什么>
- **下一步动作**：<下一个要执行的具体指令或命令>
- **需要用户决定的问题**：<必须由用户拍板的事项；没有就写"无">
- **阻塞时长**：<本工序已耗时 / 预期耗时>

## 5. 变更记录（范围 / 设计 / 返工）

| # | 日期 | 类型 | 说明 | 影响工序 | 返工原因分类 | 关联编号 | 处理 |
|---|---|---|---|---|---|---|---|
| 1 | <YYYY-MM-DD> | 范围变更 / 设计变更 / 返工 | <发生了什么> | <工序号> | 规格不清 / 设计缺失 / 编码错误 / 环境问题 / 需求变更 | `DISC-…` / `ADR-…` | <动作与结论> |

- **范围变更**：需求增删（必须同步改 `requirements.md` 与验收条件，并重跑 G1 相关检查）。
- **设计变更**：ER / 契约 / 包结构变化（必须有 ADR，不允许口头改）。
- **返工**：同一工序第二次动工即算一次返工；原因分类固定用上表五类，不得自创（否则无法汇总到 `metrics.md`）。

## 6. 时间与返工度量（每轮回填 `docs/pipeline/metrics.md`）

| 指标 | 值 | 说明 |
|---|---|---|
| 计划起止 | `<YYYY-MM-DD>` ~ `<YYYY-MM-DD>` | S0 开工到 G4 通过 |
| 实际起止 | `<YYYY-MM-DD>` ~ `<YYYY-MM-DD>` | 同上，按实际填写 |
| 工序 11–23 实际耗时 | `<n> h` | **这才是被流水线影响的部分**，必须单独统计 |
| 实际工时 | AI `<n> h` / USER `<n> h` | 用于下一轮估算校准 |
| 返工次数 | `<n>` | 与第 5 节「返工」行数一致；不一致就是有一处没记 |
| 返工原因分布 | 规格不清 `<n>` / 设计缺失 `<n>` / 编码错误 `<n>` / 环境问题 `<n>` / 需求变更 `<n>` | 合计必须等于返工次数 |
| 门禁失败次数 | G1 `<n>` / G2 `<n>` / G3 `<n>` / G4 `<n>` | 每次 FAIL 记一次 |
| 缺陷发现阶段 | 单元 `<n>` / PR `<n>` / 联调 `<n>` / 发布前 `<n>` / 发布后 `<n>` | 越靠后越贵；发布后必须为 0 |
| 用例数 | `TC-` 总数 `<n>`，其中负向 `<n>` | 负向占比 < 30% 需说明原因 |
| 资产回写条数 | L2 `<n>` / L3 `<n>` / L4 `<n>` | 来自 `discovery.md` 的分级 |

**收敛目标**：返工次数与骨架耗时（工序 11–23）应逐项目下降。若第 N 个项目没有下降，回查 24–26 是否真的改了资产。

## 7. 示例：P01 走到 S2 工序 14

**一句话现状**：P01 已完成脚手架、迁移与种子、八件套骨架；正在写 `p01_student` 的 Mapper XML 与动态查询条件，`insert` 已完成，条件分页 SQL 未完成。

| 阶段 | 工序 | 工序名 | 状态 | 负责人 | 产出物 | 门禁 | 完成日期 |
|---|---|---|---|---|---|---|---|
| S0 | 01 | 项目章程 | DONE | AI | `projects/P01-student-info/docs/charter.md` | G1 | 2025-03-03 |
| S0 | 02 | 需求卡 | DONE | BOTH | `projects/P01-student-info/docs/requirements.md` | G1 | 2025-03-03 |
| S0 | 03 | 业务规则 | DONE | BOTH | 同上（`BR-STU-01`…`BR-STU-04`） | G1 | 2025-03-04 |
| S0 | 04 | 验收条件 | DONE | BOTH | 同上（`AC-STU-01`…`AC-STU-06`） | G1 | 2025-03-04 |
| S0 | 05 | 非功能需求 | DONE | BOTH | 同上（`NFR-PERF-01`、`NFR-MAINT-01`） | G1 | 2025-03-04 |
| S1 | 06 | 数据设计 | DONE | AI | `projects/P01-student-info/docs/design.md` §3 | G2 | 2025-03-05 |
| S1 | 07 | API 契约 | DONE | AI | 同上 §5（5 个端点） | G2 | 2025-03-06 |
| S1 | 08 | 模块与包结构 | DONE | AI | 同上 §6 | G2 | 2025-03-06 |
| S1 | 09 | 页面与状态 | DONE | AI | 同上 §7（列表 / 新建 / 编辑） | G2 | 2025-03-07 |
| S1 | 10 | 决策记录 | DONE | AI | `projects/P01-student-info/docs/adr/ADR-001~003.md` | G2 | 2025-03-07 |
| S2 | 11 | 脚手架 | DONE | AI | `projects/P01-student-info/backend/` `frontend/` | — | 2025-03-08 |
| S2 | 12 | 迁移与种子 | DONE | AI | `database/migrations/V1__init_student.sql` `seed.sql` | — | 2025-03-11 |
| S2 | 13 | Entity / DTO / VO / Query | DONE | AI | `entity/Student.java`、`dto/StudentCreateDTO.java`、`query/StudentQuery.java` | — | 2025-03-11 |
| S2 | 14 | Mapper + XML | IN_PROGRESS | USER | `mapper/StudentMapper.java` + `resources/mapper/StudentMapper.xml` | — | — |
| S2 | 15 | Service + 事务 | OPEN | USER | `service/StudentService.java` + `impl/` | — | — |
| S2 | 16 | Controller | OPEN | AI | `controller/StudentController.java` | — | — |
| S2 | 17 | 异常 / 日志 / 认证 | OPEN | AI | `exception/` `handler/` | — | — |
| S2 | 18 | 前端页面与联调 | OPEN | USER | `frontend/src/views/student/` | G3 | — |
| S3 | 19–23 | 分层测试 → 打 tag | OPEN | BOTH | 见第 2 节 | G4 | — |

| 闸门 | 判定日期 | 结论 | 未通过项 | 整改动作 |
|---|---|---|---|---|
| G1 规格 | 2025-03-04 | PASS | 无 | — |
| G2 设计 | 2025-03-07 | PASS | 无（`ADR-002` 明确选择物理删除而非逻辑删除） | — |
| G3 开发 | — | — | 待工序 18 完成后判定 | — |
| G4 发布 | — | — | 未开始 | — |

- **当前卡在哪里**：工序 14 进行中。`insert` 与 `countByStudentNo` 已写完；`selectPage` 的 `<where>` 动态条件（`name` 模糊 + `age` 区间 + `studentNo` 精确）尚未完成，`pageSize` 上限校验约定为 100（见 `design.md` §5）。
- **下一步动作**：补齐 `StudentMapper.xml#selectPage` 与 `#countByPage`，然后跑 `mvnw.cmd -Dtest=StudentMapperTest test`。
- **需要用户决定的问题**：`page<=0` 是"归一为 1"还是"返回 400"？P01 的验证记录（`AV-*`）要求二者写死其一，请拍板。
- **变更记录**：`2025-03-11` 返工 1 次，原因分类 **设计缺失**，关联 `DISC-STU-01`（seed 手机号重复导致 `uk_p01_student_phone` 建索引失败），处理为清洗种子数据 + 迁移前置探测。
- **度量**：计划 2025-03-03 ~ 2025-03-18；工序 11–13 实际 8h（AI 8h）；返工 1 次；G1/G2 一次通过。

## 填写检查清单

- [ ] 23 个工序一行不缺，编号与 `delivery-pipeline.md` 一致（不自行增删编号）
- [ ] 每个 `DONE` 都有仓库内真实存在的产出物路径，且完成判据逐条为真
- [ ] 同时处于 `IN_PROGRESS` 的工序不超过 2 个；`projects/` 下只有一个进行中的项目
- [ ] G1–G4 每道闸门有判定日期与 `PASS/FAIL`；`FAIL` 有未通过项编号与整改动作
- [ ] 「当前卡在哪里」三问齐备（现状 / 下一步 / 待用户决定），且每次更新都重写过
- [ ] 每次返工都在「变更记录」留行，原因分类取自固定五类
- [ ] 第 6 节度量数字与 `docs/pipeline/metrics.md` 一致（返工次数 = 返工行数）
- [ ] `CANCELLED` 的工序写明了理由与 ADR 编号
