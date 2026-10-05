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
| 数据库 | MySQL 8，库 `java_fullstack_lab`，每项目表前缀 `p0N_` |
| 前端 | Vue 3 + Vite + TypeScript + Pinia + Element Plus + ECharts，pnpm workspace |
| 测试 | JUnit 5 + Mockito + MockMvc；集成测试用真实 MySQL |
| 文档 | SpringDoc OpenAPI（契约先行） |
| 部署 | Nginx + systemd；Docker Compose 仅作可选交付物 |

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
7. **命名规范**：需求 `REQ-模块-序号`、业务规则 `BR-模块-序号`、非功能 `NFR-分类-序号`、测试用例 `TC-模块-场景-序号`；
   数据库约束前缀 `fk_` / `ck_` / `idx_`；表名业务单数（`p01_student`）。
8. **秘密不入库**：所有连接串、密钥走环境变量或 `application-local.yml`（已 gitignore），
   仓库内只保留 `*.example`。

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
