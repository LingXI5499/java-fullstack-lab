# 交付流水线规范

> **这份文档是流水线的本体。** 十个项目是它的验证样本，不是它的目的。
> 它解决的问题只有一个：**给定一个需求，如何稳定地、可复现地、一次比一次更快地做出一个能跑、能测、能讲清楚的 Java 全栈项目。**

---

## 0. 三条铁律

### 铁律一：串行开发，一次只有一个项目

**永远同时只开发一个项目。** 前一个项目通过 G4 发布闸门并完成复盘与技能升级之后，才允许创建下一个项目的目录。

这条规则不是效率偏好，而是本流水线的训练前提：

```text
串行：项目 N 的复盘 → 升级 skill → 项目 N+1 直接受益
并行：项目 N 的坑还在踩，项目 N+1 已经复制了同样的坑
```

具体约束：

- 同一时刻，`projects/` 下只允许存在一个处于 `IN_PROGRESS` 的项目。
- 当前项目未通过 G4 之前，不得创建下一个项目的骨架（新目录、新 pom、新 package.json）。
- 允许的例外只有两类：修当前项目的 bug、给已发布项目补文档或补测试。**新功能一律排队。**

`progress-ledger.md` 的「当前卡在哪里」一节与 `tools/lab.ps1 status` 都以此为准。

### 铁律二：每一步都必须留下可检查的产出物

没有产出物的工序不算完成。产出物必须是**文件**，不是"我觉得我想清楚了"。

反例（不算完成）：

- "我理解了需求" → 没有需求卡
- "设计我心里有数" → 没有 ER 和 API 契约
- "接口我测过了" → 没有验证记录

### 铁律三：经验必须回写到资产，文档不算

每个项目结束后，复盘结论必须落成对**具体资产**的改动之一：

| 资产类型 | 位置 | 改什么 |
|---|---|---|
| 技能 | `engineering/skills/jfsl-*/` | 增加规则、修正错误、补充模板、增加检查项 |
| 工作流 | `engineering/workflows/` | 调整阶段编排、增加自动校验 |
| 生成器 | `tools/gen/` | 让下次少写/少改一个文件 |
| 模板 | `docs/templates/` | 表格字段、检查清单 |
| 门禁 | `tools/lab.ps1`、`.github/workflows/` | 把人工检查变成自动检查 |

只写 `retrospective.md` 而没动任何资产 = **本轮复盘未完成**，不允许开工下一个项目。
这条规则的载体是 [skill-upgrade-proposal.md](../templates/skill-upgrade-proposal.md)。

---

## 1. 全仓库只有一份技能库

```text
engineering/
├─ skills/            <- 所有项目共用，只有这一份，不再分散到各项目
│  ├─ jfsl-conventions/      约定源头（与 AGENTS.md 同步）
│  ├─ jfsl-scaffold/         骨架生成
│  ├─ jfsl-data-design/      数据设计
│  ├─ jfsl-api-design/       接口契约
│  ├─ jfsl-crud-gen/         八件套代码生成
│  ├─ jfsl-frontend-page/    页面骨架
│  ├─ jfsl-test-layered/     分层测试
│  ├─ jfsl-release/          打包与部署
│  ├─ jfsl-review-gate/      评审与门禁
│  └─ jfsl-retro-skill-upgrade/ 复盘回写
└─ workflows/         <- 所有项目共用，只有这一份
   ├─ stage-spec.md           S0 编排
   ├─ stage-design.md         S1 编排
   ├─ stage-implement.md      S2 编排
   ├─ stage-accept.md         S3 编排
   ├─ project-kickoff.md      串起 S0–S3（一次一个项目）
   └─ pipeline-v2.md          盲测用端到端编排
```

**项目目录里没有 skills。** 项目目录只放这个项目自己的东西：

```text
projects/P0N-<slug>/
├─ README.md            项目说明 + 陌生机器可复现的 Quick Start
├─ docs/                本项目全部文档（见下方路径裁决）
│  ├─ charter.md          01 项目章程
│  ├─ requirements.md     02–05 需求 / 业务规则 / 验收条件 / 非功能需求
│  ├─ design.md           06–09 数据设计 / API 契约 / 模块结构 / 页面与状态
│  ├─ adr/ADR-<NNN>-<slug>.md   10 架构决策（多条时另加 adr.md 汇总）
│  ├─ progress.md         23 工序账本 + 门禁记录（progress-ledger 模板）
│  ├─ discovery.md        开发中的发现记录
│  ├─ testing.md          19–20 测试计划与用例
│  ├─ deployment.md       22 部署说明（索引到 deploy/runbook.md）
│  ├─ release.md          23 发布记录
│  └─ retrospective.md    24 复盘
├─ database/            schema.sql / seed.sql / migrations/
├─ backend/             Maven 工程
├─ frontend/            Vue 工程
├─ test/                冒烟脚本与跨层测试（含 orphan-check.sql、evidence/）
└─ deploy/              nginx.conf / systemd unit / docker-compose.yml / runbook.md
```

### 路径裁决（此前模板之间有过冲突，以此为准）

| 内容 | 唯一位置 | 不放在 |
|---|---|---|
| 项目章程与全部规格/设计文档 | `projects/P0N-<slug>/docs/` | 不再放 `projects/P0N-<slug>/docs/` |
| 工序账本、发现记录、复盘 | `projects/P0N-<slug>/docs/` | —— |
| 接口验证批次记录与证据 | `.../docs/testing/api-verification/AV-*.md` 与 `.../test/evidence/AV-*/`，最新全绿结论汇总到 `.../test/api-verification.md` | —— |
| 后端单测 | `.../backend/src/test/java/` | `test/` |
| 前端单测 | `.../frontend/src/**/*.spec.ts` | `test/` |
| 冒烟脚本（跨层、需起服务） | `tools/smoke/P0N.ps1` | 项目内 |
| 跨项目共享件（流水线、技能、工作流、模板） | `docs/pipeline/`、`docs/templates/`、`engineering/` | 项目目录 |

理由：**一个项目的东西全部在一起**。串行开发的模式下，"找到这个项目的全部资料"比"把所有项目的同类文档放在一起"更重要；
跨项目的资产本来就只有一份（技能库与流水线规范），不需要通过目录去共享。

`docs/projects/` 只保留给**尚未建立代码目录的项目**做草稿区；一旦进入工序 11，文档就移入 `projects/P0N-<slug>/docs/`。

技能库通过 `engineering/sync-skills.ps1` 同步到用户的技能目录后，在任意会话中可用；
**项目之间共享技能，不复制技能。**

技能数量有上限：**12 个**。要新增技能，必须先在 [skills-map.md](skills-map.md) 立项，
说明它为什么不能被现有技能覆盖。上限的作用是防止"技能库膨胀成第二个需要维护的项目"。

---

## 2. 阶段与工序（7 阶段 / 23 工序 / 4 闸门）

工序编号是全仓库统一的，用于 `progress-ledger.md` 与提交信息。

### 标识符约定（全仓库统一，不得各自发明）

所有可追踪对象都用"`<类型>-<模块>-<两位序号>`"三段式。**模块段与场景段一律全大写，序号一律两位补零**，
因为不同大小写与不同宽度会让 grep 与追踪矩阵立刻失效。

| 类型 | 格式 | 示例 | 说明 |
|---|---|---|---|
| 需求 | `REQ-<模块>-<NN>` | `REQ-{0}-0{1}` | 一张需求卡一个编号 |
| 业务规则 | `BR-<模块>-<NN>` | `BR-{0}-0{1}` | 一条规则一个编号 |
| 验收条件 | `AC-<模块>-<NN>` | `AC-{0}-0{1}` | **一条验收条件只描述一个场景**，场景号不嵌套在编号里 |
| 测试用例 | `TC-<模块>-<场景>-<NN>` | `TC-{0}-{1}-0{2}` | 场景是动词，全大写 |
| 非功能需求 | `NFR-<分类>-<NN>` | `NFR-{0}-0{1}` | 分类段全大写 |
| 开发任务 | `TASK-<模块>-<NN>` | `TASK-{0}-0{1}` | 垂直切片任务 |
| 架构决策 | `ADR-<NN>` | `ADR-001` | 三位补零，因为会跨项目累计 |
| 技能版本 | `<技能名>@<语义化版本>` | `jfsl-scaffold@1.2.0` | —— |

**模块段必须来自项目自己的模块字典**（写在 `requirements.md` 第 0 节），例如 P01 的
`MAJ` / `CLS` / `STU` / `DASH` / `UI` / `COM` / `AUTH`。禁止中途新增未登记的模块段。

### S0 立项与规格

| 工序 | 名称 | 产出物 | 完成判据 |
|---|---|---|---|
| 01 | 项目章程 | `projects/P0N-<slug>/docs/charter.md` | 定位、技术新增点（1–2 个）、不做范围、成功判据齐备 |
| 02 | 需求卡 | `projects/P0N-<slug>/docs/requirements.md` | 每条需求有 `REQ-` 编号、角色、触发场景、期望行为、优先级 |
| 03 | 业务规则 | 同上，规则清单节 | 每条规则有 `BR-` 编号，写成"当…且…时，必须…" |
| 04 | 验收条件 | 同上，验收条件节 | 每条能用 Given/When/Then 写出来，且能判定真假 |
| 05 | 非功能需求 | 同上，NFR 节 | 指标含数值与测量方法，而不是"要快""要安全" |

**G1 规格闸门**：需求无歧义、业务规则可判定、验收条件可执行、范围边界清晰。

### S1 设计

| 工序 | 名称 | 产出物 | 完成判据 |
|---|---|---|---|
| 06 | 数据设计 | `projects/P0N-<slug>/docs/design.md` 数据节 | 概念-逻辑-物理三层；表和字段含约束与索引；迁移可写出来 |
| 07 | API 契约 | 同上，接口节 | 每个端点的请求/响应/状态码/错误码明确；与页面动作一一对应 |
| 08 | 模块与包结构 | 同上，结构节 | 类清单到全限定名级别；依赖方向无环 |
| 09 | 页面与状态 | 同上，前端节 | 每个页面列出 `empty/loading/loaded/error/submitting` 表现 |
| 10 | 决策记录 | `projects/P0N-<slug>/docs/adr/ADR-*.md` | 有取舍的地方都有 ADR，含代价与复审触发条件 |

**G2 设计闸门**：契约冻结、权限落点明确、设计可映射到任务（用 [design-completion.md](../templates/design-completion.md) 逐条勾选）。

### S2 实现

| 工序 | 名称 | 产出物 | 完成判据 |
|---|---|---|---|
| 11 | 脚手架 | `backend/` `frontend/` `deploy/` | 可直接启动；`docs/dev-env.md` 的步骤能跑通 |
| 12 | 迁移与种子 | `database/schema.sql` `seed.sql` | 空库可执行；重复执行幂等或有明确说明 |
| 13 | Entity / DTO / VO / Query | 对应包 | 字段与设计一致；DTO 有校验注解；无 `record` |
| 14 | Mapper + XML | `mapper/` + `resources/mapper/` | 参数化 SQL（`#{}`）；动态条件用 `<if>/<where>`；无 `${}` 拼接用户输入 |
| 15 | Service + 事务 | `service/` + `service/impl/` | 业务规则在 Service；事务边界与设计一致 |
| 16 | Controller | `controller/` | 只有参数绑定、`@Valid`、调用、返回 `Result<T>` |
| 17 | 异常 / 日志 / 认证 | `exception/` `handler/` `interceptor/` `aspect/` | 业务异常有专属子类；日志脱敏；401/403 路径可验证 |
| 18 | 前端页面与联调 | `frontend/` | 页面状态齐全；分页与搜索联动正确；联调通过 |

**G3 开发闸门**：DoD 8 项（见 [dev-task-card.md](../templates/dev-task-card.md)）。

### S3 验证与交付

| 工序 | 名称 | 产出物 | 完成判据 |
|---|---|---|---|
| 19 | 分层测试 | `test/` `src/test/` | 验收条件 → 用例一一对应；关键 SQL 用真实 MySQL 验证 |
| 20 | 冒烟脚本 | `tools/smoke/P0N.ps1` | 一条命令跑完核心链路；失败时给出可读原因 |
| 21 | 打包 | `backend/target/*.jar` `frontend/dist/` | 构建可复现；产物与提交可关联 |
| 22 | 部署配置与 Runbook | `deploy/` | Nginx + systemd + 可选 compose + runbook 步骤可照做 |
| 23 | 打 tag 与归档 | 带注释 tag `p0N-vX.Y.Z` + 发布记录 | 发布记录含产物/迁移/配置/验证/回滚；作品集导出已生成 |

**G4 发布闸门**：发布阻断 5 条件——核心流程失败、存在越权、迁移不可恢复、数据一致性错误、关键配置未知。

### 复盘升级（G4 之后，开工下一个项目之前）

| 步骤 | 产出物 | 完成判据 |
|---|---|---|
| 24 | 项目复盘 | `projects/P0N/docs/retrospective.md` | 四问齐备；度量表填了真实数字 |
| 25 | 技能升级提案 | `docs/pipeline/upgrade-log.md` 新增条目 | 每条提案能落到 diff 级别，并已实际修改资产 |
| 26 | 度量更新 | `docs/pipeline/metrics.md` | 更新 T(N)、返工次数、缺陷分布 |

只有 24–26 全部完成后，才允许开始下一个项目的工序 01。

---

---

## 3. 单库 + 前缀隔离（数据库铁律）

**全仓库只有一个数据库：`java_fullstack_lab`。** 十个项目不各自建库，而是在同一个库里通过**命名前缀**做表级隔离。

```text
java_fullstack_lab
├─ p01_student            学生信息管理系统（P01）
├─ p01_class              班级
├─ p02_book               图书管理系统（P02）
├─ p02_borrow_record      借阅记录
├─ p03_medicine           药店管理系统（P03）
├─ ...
└─ p10_knowledge_doc      个人知识平台（P10）
```

### 命名规则（可自动检查）

| 对象 | 规则 | 示例 |
|---|---|---|
| 表 | `p<两位项目号>_<业务单数名>` | `p01_student`、`p05_course` |
| 主键 | `<表名后缀>_id`，类型 `bigint unsigned auto_increment` | `student_id` |
| **逻辑外键字段** | `<被引用表单数>_id`，**必须建索引**，**不建物理外键** | `class_id` → `p01_class` |
| 唯一约束 | `uk_<表名>_<字段名>` | `uk_p01_student_student_no` |
| 普通索引 | `idx_<表名>_<字段名列表>` | `idx_p02_borrow_record_status_created_at` |
| 检查约束 | `ck_<表名>_<条件名>` | `ck_p01_student_age_range` |
| 视图 / 存储过程 | 本流水线不使用 | —— |
| 字段命名 | `snake_case`，布尔用 `is_` 前缀，时间用 `_at` 后缀 | `is_deleted`、`created_at` |

### 禁止物理外键，只用逻辑外键（硬规则）

**`schema.sql` 中不得出现 `FOREIGN KEY` 与 `REFERENCES`，也不允许 `ON DELETE CASCADE`。**
关联只通过"逻辑外键字段 + 索引 + 应用层校验"表达。

| 关注点 | 用物理外键 | 用逻辑外键（本流水线） |
|---|---|---|
| 引用完整性 | 数据库强制 | **Service 层强制**：写入前校验被引用记录存在 |
| 删除父记录 | `ON DELETE RESTRICT / CASCADE` 自动处理 | Service 层显式检查子记录，返回业务错误码（如 `CLASS_HAS_STUDENTS`） |
| 迁移与批量导入 | 受约束顺序限制，导数据要临时关闭外键检查 | 无顺序限制，导入脚本简单 |
| 分库分表与归档 | 约束成为障碍 | 天然可拆 |
| 插入性能 | 每次写入多一次约束校验 | 无额外开销，但失去数据库兜底 |
| 孤儿数据 | 数据库兜底 | **靠检查手段兜底**（见下） |

由此产生三条**强制要求**（缺一不可）：

1. **所有逻辑外键字段必须建索引**（`idx_<表>_<字段>`），否则关联查询退化为全表扫描；
2. **删除父记录前必须显式检查引用方**并返回专属业务错误码；P01 的对应规则是 `CLS` 与 `MAJ`；
3. **"引用不存在"的路径必须有测试用例**。例：用不存在的 `class_id` 新增学生，必须返回 `CLASS_NOT_FOUND`，而不是插入成功。

孤儿数据检测：每个项目在 `test/` 下提供排查 SQL，供 Runbook 与巡检使用：

```sql
-- P01 示例：找出班级已不存在的学生
SELECT s.student_id, s.student_no, s.name, s.class_id
FROM p01_student s
LEFT JOIN p01_class c ON c.class_id = s.class_id
WHERE c.class_id IS NULL;
```


### 为什么单库而不是十库

| 单库 + 前缀 | 十库 |
|---|---|
| 一处连接配置，切库零成本 | 十个连接串、十套本地配置 |
| 对比十个时期的设计差异只需 `SHOW TABLES LIKE 'p0N_%'` | 需要跨库来回切 |
| 项目阶段快照（tag）与表结构演进一一对应，好回溯 | 快照与库的对应关系容易丢 |
| 缺点：需要靠规则防越界 | 缺点：本地环境维护成本高 |

### 越界防范（三条硬规则）

1. **一个项目的 SQL 只允许出现自己的前缀。** P03 的 `schema.sql` 里出现 `p01_` 就是错误。
   这条由门禁自动检查（`lab.ps1 verify` 内含前缀扫描）。
2. **禁止跨项目 JOIN。** 需要复用数据时，通过导入脚本或作品集导出适配器传递，不在 SQL 里直接连表。
3. **不允许任何物理外键**（跨项目更不允许）。关联只靠 `<被引用表单数>_id` 字段 + 索引 + 应用层校验，
   详见 [禁止物理外键，只用逻辑外键](#禁止物理外键只用逻辑外键硬规则)。

### 与迁移的关系

- 每个项目自带 `database/schema.sql`（结构）与 `database/seed.sql`（种子），**只含本项目前缀的表**。
- 迁移脚本命名：`database/migrations/V<版本>__<说明>.sql`，例如 `V2__add_renew_flag.sql`。
- 执行顺序：先跑 P0N 的 `schema.sql`，再跑 `seed.sql`；升级时只跑增量迁移。
- 变更采用「扩展—迁移—收缩」三步：先加字段 → 新代码兼容旧数据 → 回填 → 切换读写 → 清理。
- 建库语句（只做一次）：

```sql
CREATE DATABASE IF NOT EXISTS java_fullstack_lab
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

### 连接配置

只有一个连接串，写在每个项目**已 gitignore** 的 `application-local.yml` 里：

```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:mysql://localhost:3306/java_fullstack_lab?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true}
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:}
```

`application-local.yml.example` 入库，真实的 `application-local.yml` 不入库。

---

## 4. AI 与用户的分工

本流水线采用 **"AI 搭骨架 + 用户写业务代码"**。分界线是**业务规则**，不是文件类型。

| 归属 | 内容 |
|---|---|
| **AI** | 阶段规范、需求卡、ER、API 契约、ADR、可编译骨架、配置、建表与种子 SQL、接口与签名、XML 骨架、测试骨架、冒烟脚本、CI、部署配置、评审与纠错、把经验写回技能库 |
| **用户** | `Service` 实现里的业务规则与事务编排、`Mapper XML` 的 SQL 与动态条件、Vue 页面的交互与状态、业务测试的断言与用例设计 |

执行要求：

1. 留给用户的实现位必须写成 `// TODO(user): <要做什么> - <验收条件编号>`。
2. 不得为了让程序"跑起来"而替用户实现业务规则，除非用户明确要求。
3. 每次交付后必须运行门禁命令，并如实报告失败原因，不得跳过。
4. 交付时同时给出**讲解**：这段骨架为什么这么分层、用户要实现的那部分影响哪条规则。

---

## 5. 项目推进的标准动作

一次完整推进（以 P0N 为例）：

```powershell
# 1. 确认当前没有别的项目在进行
.\tools\lab.ps1 status

# 2. 确认上一个项目已经复盘完毕（升级日志有本轮的条目）
git log --oneline -5

# 3. 生成骨架（S2 工序 11）
.\tools\lab.ps1 new <slug> --project P0N

# 4. 按工序推进，每完成一个工序就更新账本
#    projects/P0N-<slug>/docs/progress.md

# 5. 生成一个业务模块（工序 13–16）
.\tools\lab.ps1 gen-module --project P0N --spec docs\projects\P0N\modules\<module>.md

# 6. 门禁（G3 / G4）
.\tools\lab.ps1 verify --project P0N

# 7. 打里程碑
.\tools\lab.ps1 tag --project P0N --version 1.0.0
```

---

## 6. 与 Git 的绑定

| 动作 | 约定 |
|---|---|
| 主干 | `main`，始终可构建、通过门禁 |
| 阶段分支 | `feat/P0N-<slug>`，合并用 `--no-ff`，禁止压平历史 |
| 提交 | Conventional Commits + 项目作用域，一次一个目标，如 `feat(p03): add sale transaction with stock rollback` |
| 工序级提交 | 每个工序完成时提交一次，提交信息带上工序号，如 `test(p02): add borrow/return cases (step 19)` |
| 里程碑 tag | 带注释 tag `p0N-vX.Y.Z`，推送后不得移动 |
| 复盘提交 | 技能升级与项目复盘放在同一个提交里，如 `chore(skills): tighten pagination guard from p02 retro` |

---

## 7. 度量：怎么证明"越来越快"

`docs/pipeline/metrics.md` 记录每个项目的：

- 计划工期 vs 实际工期
- 返工次数与原因分类（规格不清 / 设计缺失 / 编码错误 / 环境问题 / 需求变更）
- 门禁失败次数与失败原因
- 测试用例数、缺陷数与发现阶段（越早发现越便宜）
- 从工序 11 到工序 23 的实际耗时（这才是被流水线影响的部分）

**收敛目标是这两条曲线同时下降**：

```text
返工次数  T1 > T2 > T3 > ... > T10
骨架耗时  T1 > T2 > T3 > ... > T10
```

如果第 N 个项目的返工次数没有下降，说明第 N-1 轮的复盘没有落到资产上——回到铁律三。

---

## 8. 什么时候允许偏离本规范

只有三种情况：

1. **用户明确要求**（例如要求 AI 代为实现某个业务规则）。
2. **技术现实不允许**（例如某库在 JDK 17 下无法运行）——此时必须写 ADR 记录。
3. **本规范自身有缺陷**——此时先改本规范并提交，再按新版本执行；不允许"先做再说"。

偏离必须留下痕迹：ADR、`discovery-record.md` 或 `change-impact.md` 三者之一。
