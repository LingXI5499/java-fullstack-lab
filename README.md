# Java Full-Stack Lab

> **一条可重复、可度量、自带质量门禁的 Java 全栈工程流水线。**
> 十个 Spring Boot + MyBatis + MySQL + Vue 项目是这条流水线的实验样本；
> 每跑完一个项目，流水线资产（Skill / 模板 / Agent 工作流）必须比上一轮更少返工。

## 这个仓库是什么

这不是"十个项目的源码堆放处"，而是**一套工程流水线 + 它的十个验证样本**：

```text
规格（需求卡 / 业务规则 / 验收条件）
  ↓
设计（ER / API 契约 / 包结构 / ADR）
  ↓
实现（脚手架生成 + 你写业务代码 + AI 评审）
  ↓
验证（JUnit / MockMvc / 冒烟脚本 / 四道闸门）
  ↓
交付（打包 / Nginx + systemd / Runbook / 里程碑 tag）
  ↓
复盘（把踩到的坑写回 Skill，版本 +1，下一轮更快）
  ↑______________________________________________________|
```

四道闸门（G1 规格 / G2 设计 / G3 开发 / G4 发布）不通过就不进入下一阶段。
每个项目的经验必须落到**具体资产**（Skill 文本、模板文件、生成器规则），不允许"只写文档不改资产"。

## 技术基线

| 层 | 技术 | 约定 |
|---|---|---|
| 语言 | Java 17（`maven.compiler.release=17`） | 本机可用 JDK 21 编译，产物必须兼容 17 |
| 构建 | Maven 3.9 + Maven Wrapper | 每个项目独立 `pom.xml`，上游 BOM 由父 POM 统一 |
| 后端 | Spring Boot 3.2.x / Spring MVC / MyBatis(XML) / MySQL 8 | 手写 XML，**不引入 MyBatis-Plus** |
| 认证 | JWT + Interceptor + Filter | 从 P06 开始引入 |
| 文档 | SpringDoc OpenAPI | 接口契约先行 |
| 测试 | JUnit 5 / Mockito / MockMvc | 与验收条件一一对应 |
| 前端 | Vue 3 + Vite + TypeScript + Pinia + Element Plus + ECharts | pnpm workspace 统一管理 |
| 部署 | Nginx + systemd + Docker Compose（可选） | 本地可复现为验收底线 |
| 版本管理 | Git + Conventional Commits + 带注释里程碑 tag | 禁止压平历史 |

**包结构**统一采用 `star-rain` 分层规范：
`controller / service(+impl) / mapper / entity / dto / vo / query / config / exception / interceptor / aspect / utils`。

**硬性编码约定**：禁止使用 Java `record`（VO/DTO 一律 Lombok POJO）；业务异常使用"每个错误一个子类"的继承体系。

## 十个项目

| # | 项目 | 训练重点 | 新增技术点 | 端口 |
|---|---|---|---|---|
| P01 | 学生信息管理系统 | CRUD + 条件/分页查询 | 脚手架、统一 `Result`、全局异常、日志 | 8101 |
| P02 | 图书管理系统 | 借阅/归还 + 多表 | 事务、`LEFT JOIN`、状态机 | 8102 |
| P03 | 药店管理系统 | 库存/销售 | 事务回滚、库存一致性 | 8103 |
| P04 | 仓库管理系统 | 入库/出库/盘点 | 并发更新、乐观锁、索引 | 8104 |
| P05 | 课程管理系统 | 选课/成绩/课表 | 排课冲突校验、聚合统计 | 8105 |
| P06 | 博客系统 | 用户/文章/评论/上传 | JWT 全链路、文件存储抽象 | 8106 |
| P07 | 招聘管理系统 | 企业/岗位/简历/投递 | 外部依赖降级、AOP 操作日志 | 8107 |
| P08 | 订单管理系统 | 订单/订单项/库存 | 事务边界、幂等、报表统计 | 8108 |
| P09 | 项目任务管理系统 | 项目/成员/任务/进度 | 复杂业务编排、看板、ECharts | 8109 |
| P10 | 个人知识平台 + 作品集导出 | 整合验证 | 星雨笔录作品集对接、部署验收 | 8110 |

每个项目**恰好新增 1–2 个技术点**，其余全部复用。跨梯度技术（Redis / MQ / 微服务 / K8s）见 [非目标](#非目标)。

## 目录结构

```text
java-fullstack-lab/
├─ AGENTS.md                  # AI 协作与工程约定入口
├─ docs/
│  ├─ pipeline/               # 流水线核心资产（阶段规范 / Skill 注册表 / 度量 / 升级日志）
│  ├─ templates/              # 需求卡 / 任务卡 / ADR / 设计完成卡 / 用例 / 发布记录 …
│  ├─ projects/P01…P10/       # 每个项目的需求、设计、数据库、API、测试、部署文档
│  ├─ reference/              # Java 全栈开发体系 V1 母版
│  └─ dev-env.md              # 本机环境与版本约定
├─ engineering/
│  ├─ skills/jfsl-*/          # 技能库（DSH 结构：SKILL.md + references/ + templates/ + scripts/）
│  ├─ workflows/              # Agent 工作流：stage-spec / stage-design / stage-implement / stage-accept
│  └─ sync-skills.ps1         # 把技能同步到 ~/.dsh/skills/
├─ tools/
│  ├─ lab.ps1                 # new / gen-module / verify / tag
│  ├─ templates/              # Maven + Vue 骨架模板
│  ├─ gen/                    # 代码生成器
│  └─ smoke/                  # 每个项目的冒烟脚本
├─ projects/P01-student-info/ … P10-knowledge-platform/
│   ├─ backend/  frontend/  database/  test/  deploy/  docs/  README.md
└─ .github/workflows/         # backend.yml / frontend.yml / docs.yml
```

## 快速开始

前置：JDK 17+、Maven 3.9+、Node 20+、pnpm、MySQL 8。

```powershell
# 1. 建库（只做一次）
mysql -uroot -p -e "CREATE DATABASE IF NOT EXISTS java_fullstack_lab DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;"

# 2. 配一个项目的本地配置
Copy-Item projects\P01-student-info\backend\src\main\resources\application-local.yml.example `
          projects\P01-student-info\backend\src\main\resources\application-local.yml
# 编辑 application-local.yml 填入本机数据库账号

# 3. 导入该项目的表结构与种子数据
Get-Content projects\P01-student-info\database\schema.sql | mysql -uroot -p java_fullstack_lab
Get-Content projects\P01-student-info\database\seed.sql   | mysql -uroot -p java_fullstack_lab

# 4. 起后端
cd projects\P01-student-info\backend; .\mvnw.cmd spring-boot:run

# 5. 起前端
cd projects\P01-student-info\frontend; pnpm install; pnpm dev
```

每个项目的 `README.md` 都有一份可在**陌生机器**上复现的 Quick Start（受 AC-8 约束）。

## 版本与 tag

主干 `main` 始终代表"最新通过验收的状态"。阶段开发使用 `feat/P0N-<slug>` 分支，合并用 `--no-ff` 保留演进痕迹。

```bash
git tag -l                    # 查看里程碑
git checkout p01-v1.0.0       # 检出某个项目里程碑时期的全部源码
```

| 流程 | 命令 |
|---|---|
| 新建骨架 | `.\tools\lab.ps1 new student-info --project P01` |
| 生成模块 | `.\tools\lab.ps1 gen-module --project P01 --spec docs\projects\P01\modules\student.md` |
| 质量门禁 | `.\tools\lab.ps1 verify --project P01` |
| 打里程碑 | `.\tools\lab.ps1 tag --project P01 --version 1.0.0` |

## 非目标

主线**不引入**：Redis、RabbitMQ、Kafka、Elasticsearch、Spring Cloud、微服务、分布式事务、Kubernetes、分库分表、真实支付/短信、复杂前端组件库体系。

理由：先把单体 Web 工程练到"脱口而出"，再升级架构。这些能力留给本流水线的下一版。

## 与星雨笔录的关系

本仓库产出的项目会汇入 [星雨笔录](https://github.com/LingXI5499/star-rain-notes) 的作品集模块：
流水线只输出**可导入的作品集数据**（JSON / 静态页），不改动星雨笔录自身的库表与接口。

## 许可

MIT，见 [LICENSE](LICENSE)。
