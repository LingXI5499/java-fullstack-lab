# 参考与需求来源

本目录存放**只读**的外部输入：它们是流水线的需求来源，不是流水线本身。
修改参考文档前请先确认是否需要同步修改 `docs/pipeline/` 下的规范。

| 文件 | 来源 | 用途 |
|---|---|---|
| [Java全栈开发体系V1_项目训练母版.md](Java全栈开发体系V1_项目训练母版.md) | 用户既有资产（V1 母版） | 技术基线、标准项目目录、请求链路、23 步开发顺序、十个训练项目的选择依据 |

## 这份母版在本仓库中的角色

它定义了本流水线的**能力范围与训练梯度**：

- 技术边界：JDK + Maven + Spring Boot + Spring MVC + MyBatis + MySQL + JWT + Vue + Element Plus + ECharts + JUnit + Apifox + OSS + Linux/Nginx
- 训练顺序（母版 §55）：学生 → 图书 → 药店 → 仓库 → 课程 → 博客 → 招聘 → 订单 → 项目任务 → 个人知识平台
- 能力等级（母版 §48）：L0 听过 → L1 能解释 → L2 能看懂 → L3 能独立开发 → L4 能设计/Debug/优化 → L5 能讲清楚
- 23 步开发顺序（母版 §47）：本仓库把它收敛为 7 大阶段 / 23 工序 / 4 道闸门，见 [../pipeline/delivery-pipeline.md](../pipeline/delivery-pipeline.md)

## 与母版的三处有意偏离

| 母版 | 本仓库 | 原因 |
|---|---|---|
| §39 标准目录把 `common/` 放在后端根下 | `AGENTS.md` 规定的 `star-rain` 分层（`constant/enumeration/exception/handler/result/utils`） | 沿用用户既有工程的真实包规范，避免两套并行 |
| §13 建议后端直连 OSS | `storage/` 抽象 + `LocalDiskStorage`（默认）+ `OssStorage`（配置启用） | 开发与 CI 环境没有云密钥，必须能离线跑通 |
| §50 列了 24 个候选项目 | 只做 §55 推荐的 10 个 | 每个项目只新增 1–2 个技术点，防止技能点稀释 |
