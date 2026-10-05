# AGENTS.md — 本仓库的 AI 协作与工程约定入口

> 这份文件是**唯一约定源**（Single Source of Truth）。
> 技能库 `engineering/skills/jfsl-conventions/SKILL.md` 与本文件必须保持一致；
> 若出现冲突，以本文件为准，并立即修正技能库。

## 1. 这个仓库在做什么

用十个 Java 全栈项目作为样本，构建一条**可重复、可度量、自带质量门禁**的开发流水线。
流水线本身（阶段规范 + 技能库 + 工作流 + 脚手架）才是主交付物，项目是验证样本。

## 2. 技术基线（不得擅自串版本）

| 项 | 值 |
|---|---|
| JDK | 编译目标 17（`maven.compiler.release=17`），本机 JDK 21 可编译 |
| Spring Boot | 3.2.x |
| ORM | MyBatis，**手写 Mapper XML**，禁止引入 MyBatis-Plus |
| 数据库 | MySQL 8；**单库 `java_fullstack_lab`**，每项目表前缀 `p0N_` 做隔离 |
| 前端 | Vue 3 + Vite + TypeScript + Pinia + Element Plus + ECharts，pnpm workspace |
| 测试 | JUnit 5 + Mockito + MockMvc；集成测试用真实 MySQL |
| 文档 | SpringDoc OpenAPI（契约先行） |
| 部署 | Nginx + systemd；Docker Compose 仅作可选交付物 |

**推迟到真正需要时再引入的技术**（不要提前实现）：

- 云对象存储（OSS）与 `storage/` 抽象 —— 到 P06 需要文件上传、且拿到密钥时再做；
- 域名、HTTPS 证书、真实服务器 —— 到真实部署时再处理；
- 其余见第 8 节非目标。


## 3. 硬性编码约定

1. **禁止 Java `record`**。VO / DTO / 值对象一律 Lombok POJO：
   `@Data @Builder @NoArgsConstructor @AllArgsConstructor`。
   实体使用 `@Data`，不额外加 `@Builder`。
2. **业务异常用继承体系**：公共基类持有 `code / message / httpStatus`，每个错误一个子类放进 `exception` 包。
   禁止"集中式静态工厂 + 通用 ApiException"。
3. **包结构**采用 `star-rain` 分层规范：
   `controller / service(+impl) / mapper / entity / dto / vo / query / config / constant / enumeration / exception / handler / interceptor / aspect / utils / storage(+impl)`。
   禁止 `support`、`misc`、模块内 `common` 这类无语义包名。
4. **接口与实现分离**：`service/` + `service/impl/`、`storage/` + `storage/impl/`。
   没有接口时不要为了对称硬造接口。
5. **Controller 只做**：参数绑定、`@Valid`、调 Service、返回 `Result<T>`。
   不写 SQL、不写业务判断、不逐个 `try/catch`。
6. **注释风格**：用 `//` 与 `/* */`，**不写 Javadoc 文档注释**。
7. **命名规范**：需求 `REQ-模块-两位序号`、业务规则 `BR-模块-两位序号`、验收条件 `AC-模块-两位序号`、
   测试用例 `TC-模块-场景-两位序号`、非功能 `NFR-分类-两位序号`、任务 `TASK-模块-两位序号`、决策 `ADR-三位序号`；
   **模块段与场景段全大写、序号两位补零**（例 `REQ-STU-01`、`TC-STU-CREATE-02`，不写 `REQ-stu-1`）；
   模块段必须来自项目 `requirements.md` 第 0 节的模块字典。数据库约束前缀 `uk_` / `idx_` / `ck_`；表名业务单数（`p01_student`）。
8. **秘密不入库**：所有连接串、密钥走环境变量或 `application-local.yml`（已 gitignore），
   仓库内只保留 `*.example`。
9. **单库 + 前缀隔离**（数据库铁律）：
   - 全仓库只有一个库 `java_fullstack_lab`，十个项目共用，**不各自建库**；
   - 每个项目的所有表以 `p<两位项目号>_` 为前缀，例如 P01 的 `p01_student`、P02 的 `p02_book`；
   - 约束命名：`uk_` / `idx_` / `ck_` 前缀，字段 `snake_case`，布尔 `is_` 前缀，时间 `_at` 后缀；
   - **一个项目的 SQL 只允许出现自己的前缀**（门禁自动扫描），禁止跨项目 JOIN；
   - 迁移脚本：`database/migrations/V<版本>__<说明>.sql`。
10. **禁止物理外键，只用逻辑外键**（数据库铁律）：
    - `schema.sql` 中**不得出现 `FOREIGN KEY` 与 `REFERENCES`**，不允许 `ON DELETE CASCADE`；
    - 关联通过 `<被引用表单数>_id` 字段（如 `p01_student.class_id`）表达，该字段**必须建普通索引**；
    - 引用完整性由**应用层保证**：写入前校验被引用记录存在，删除前校验无引用方；
    - 三条强制要求：① 删除父记录前显式检查子记录并返回业务错误码；
      ② 每个关联字段都要有"引用不存在"路径的用例；③ 每个项目在 `test/` 提供孤儿数据排查 SQL。

## 4. 流水线与闸门

```text
S0 立项与规格 ──G1──> S1 设计 ──G2──> S2 实现 ──G3──> S3 验证与交付 ──G4──> 复盘升级
```

- S0 立项与规格：项目章程、需求卡、业务规则、验收条件（Given/When/Then）、非功能指标
- S1 设计：概念-逻辑-物理数据设计、API 契约、包结构、页面与状态、ADR
- S2 实现：脚手架、迁移与 seed、Entity/DTO/VO、Mapper+XML、Service+事务、Controller、异常/日志/认证、前端联调
- S3 验证与交付：分层测试、冒烟脚本、打包、部署配置与 Runbook、打 tag + 归档作品集

**闸门不通过就不进入下一阶段。** 详细定义见 `docs/pipeline/delivery-pipeline.md`。

## 5. AI 协作边界（重要）

本项目采用"**AI 搭骨架 + 用户写业务代码**"模式：

### 5.0 需求只能来自用户（最高优先级的边界）

**在用户给出需求之前，不得创建项目目录、不得设计表结构、不得编写业务规则或验收条件。**

- 母版文档里的项目清单只是**候选顺序**，不构成需求；把它当成需求就是编造。
- 需求缺口只能以**问题清单**的形式返回给用户，不允许用"合理推测"填补。
- 任何由 AI 推测出来的业务内容（表字段、规则、错误码、种子数据）都必须显式标注为推测，
  并放在用户可一键否决的位置（分支或草稿目录）。
- 需求接收方式与确认流程见 [docs/requirements-intake.md](docs/requirements-intake.md)：
  用户给需求 → AI 整理成 `REQ-*` / `BR-*` / `AC-*` → **用户确认** → 设计 → **用户确认** → 才写代码。

**AI 负责**
- 阶段规范、需求卡、ER、API 契约、ADR
- 可编译的工程骨架、配置、模板、建表与种子 SQL
- CRUD 全链路的**接口与骨架**（八件套的结构、签名、XML 骨架）
- 测试骨架、冒烟脚本、CI、部署配置
- 代码评审、纠错、补测试、把经验写回技能库

**用户负责**
- `Service` 实现里的业务规则与事务编排
- `Mapper XML` 的 SQL 与动态条件
- Vue 页面的交互与状态
- 业务测试的断言与用例设计

**执行要求**
- 凡是留给用户的实现位，必须写 `// TODO(user): <要做什么> - <验收条件编号>`，并在阶段文档里给出讲解。
- 不得为了让代码"跑起来"而替用户实现业务规则，除非用户明确要求。
- 每次交付后必须运行门禁命令，并把失败原因如实报告。

## 6. 提交与分支

- 主干：`main`（始终可构建、通过门禁）
- 阶段分支：`feat/P0N-<slug>`，合并使用 `--no-ff`，禁止压平历史
- Commit：Conventional Commits + 项目作用域，例如
  `feat(p03): add sale transaction with stock rollback`
- 一次提交一个目标，且必须可构建、可解释
- 里程碑使用带注释 tag：`p0N-vX.Y.Z`，推送后不得移动

## 7. 门禁命令

```powershell
.\tools\lab.ps1 verify --project P01     # 编译 + 测试 + 前端构建 + 秘密扫描
.\tools\lab.ps1 tag    --project P01 --version 1.0.0
```

## 8. 非目标

Redis、RabbitMQ、Kafka、Elasticsearch、Spring Cloud、微服务、分布式事务、Kubernetes、
分库分表、真实支付/短信、复杂前端组件库体系。
