# ADR 架构决策记录

> **用途**：把一次架构取舍的「背景 — 候选 — 决策 — 代价」固化成一份可复审、可引用的记录，避免同一件事在每个项目里被重新讨论一遍。
> **填写时机**：S1 设计阶段的**第一步**，必须在 `data-design.md` 与 `api-contract.md` 定稿之前完成——选型决定表结构与契约形状，定稿后再改代价最大。
> **产出位置**：`projects/P0N-<slug>/docs/adr/ADR-<序号>-<slug>.md`；同一项目多条决策时，按 `## ADR-0NN` 分节汇总进 `projects/P0N-<slug>/docs/adr.md`。

## 1. 什么时候需要写 ADR

命中下面 ≥2 条才写；一条都不命中的事不要写 ADR（那是变更记录，不是决策）。

| 判据 | 说明 | 本仓库例子 |
|---|---|---|
| 有取舍 | 至少 2 个候选方案都能满足需求，没有唯一正确答案 | 手写 MyBatis XML vs MyBatis-Plus |
| 有代价 | 选它就要放弃某个东西（开发速度 / 性能 / 可移植性 / 学习成本） | 手写 XML 换来 SQL 可见性，代价是代码量 |
| 影响后续多个项目 | 决策会被 P02…P10 复制，改一次要改十处 | `storage/` 抽象、异常继承体系 |
| 会被反复追问「为什么不用 X」 | 不写下来就要在每个项目里重新解释 | 为什么禁止 `record`、为什么不上 JPA |

**不需要写 ADR**：技术基线已钉死的事（见第 6 节表格）、纯个人偏好、可随手回滚且不影响其他项目的实现细节。

状态流转规则：

```text
PROPOSED ──G2 通过──> ACCEPTED ──被新 ADR 取代──> SUPERSEDED
                          └──不再适用且无替代──> DEPRECATED
```

- 推翻一份 `ACCEPTED` 的 ADR：**新开一个新编号的 ADR**，把旧的置为 `SUPERSEDED` 并写明「由 ADR-0NN 取代」。
- 已 `ACCEPTED` 的正文不得就地改写，只能追加「修订说明」+ 日期。

## 2. 字段定义

| 字段 | 必填 | 取值 / 格式 | 写法要求 |
|---|---|---|---|
| 编号 | 是 | `ADR-<三位序号>` | 仓库内全局递增，不按项目重置 |
| 标题 | 是 | 一句话 | 用「为什么 X」句式，不用「关于 X 的决策」 |
| 状态 | 是 | `PROPOSED` / `ACCEPTED` / `SUPERSEDED` / `DEPRECATED` | 大写英文，不写中文状态 |
| 日期 | 是 | `YYYY-MM-DD` | 决策日期；有复审时追加复审日期 |
| 关联需求 | 是 | `REQ-<模块>-<序号>`、`BR-<模块>-<序号>`、`NFR-<分类>-<序号>` | 至少一个编号，否则是空想 |
| 背景 | 是 | 3–8 行 | 只写事实与约束，不写结论 |
| 候选方案 | 是 | ≥2 个 | 每个方案都要写**优点**与**代价**；只写优点的方案是伪候选 |
| 决策 | 是 | 一句话 | 明确到「用 A 不用 B」，禁止「视情况而定」 |
| 决策理由 | 是 | 3–6 条 | 指定性理由优先（训练目标 / 基线 / CI 约束），性能理由要给数据 |
| 后果 | 是 | 正面 / 负面各 ≥1 条 | 负面后果必须写；写不出来说明代价没想清楚 |
| 复审触发条件 | 是 | 可观测的条件 | 「感觉不合适时」无效，要写「当 X 超过 N 时」 |

## 3. 空白模板（复制这一段）

```markdown
## ADR-<序号>：为什么 <做某件事>

- 状态：PROPOSED
- 日期：<YYYY-MM-DD>
- 关联需求：REQ-<模块>-<序号>、NFR-<分类>-<序号>
- 关联设计：`docs/data-design.md#<小节>`、`docs/api-contract.md#<小节>`

### 背景

<当前约束、问题、不决策的后果。3–8 行，只写事实。>

### 候选方案

| 方案 | 做法 | 优点 | 代价 |
|---|---|---|---|
| A <名称> | <怎么做> | <得到什么> | <放弃什么> |
| B <名称> | <怎么做> | <得到什么> | <放弃什么> |

### 决策

采用 <方案 A>。

### 决策理由

1. <硬约束类理由，如 CI 无密钥 / 训练目标>
2. <可验证类理由，如 EXPLAIN 可对照>

### 后果

- 正面：<获得什么>
- 负面：<放弃什么，用什么措施补偿>

### 复审触发条件

- <当 X 超过 N 时>
- <当出现某类需求时>
```

## 4. 示例一：ADR-001 为什么用 MyBatis 手写 XML 而不是 MyBatis-Plus

- 状态：ACCEPTED
- 日期：2026-01-05
- 关联需求：NFR-可维护性-001、STU
- 关联设计：`docs/data-design.md#5-索引设计`、`docs/class-responsibility.md#1-分层职责与禁止事项`

### 背景

本仓库的训练目标是「把一条请求链路讲清楚」，其中最关键的一环是**一眼看懂 SQL**：能解释 `LEFT JOIN` 为什么这么写、联合索引为什么命中、`EXPLAIN` 里 `type=ALL` 意味着什么。十个项目的后端都用 MyBatis 访问 MySQL 8，而 MyBatis 有两种差别极大的用法：手写 Mapper XML 与 MyBatis-Plus 的 `QueryWrapper`。选哪一种会直接决定每个模块的实现骨架、代码生成器规则与评审清单。

### 候选方案

| 方案 | 做法 | 优点 | 代价 |
|---|---|---|---|
| A 手写 Mapper XML | 每个 Mapper 接口配一份 XML，SQL 显式写在 `<select>/<insert>/<update>/<delete>` 中，动态条件用 `<where>/<if>/<choose>` | SQL 与执行计划一一对应；索引可用 `EXPLAIN` 直接验证；动态 SQL 可读；与 `PageResult` 分页约定天然匹配；不引入额外依赖 | 每个实体要维护 4 条以上语句，代码量大；XML 拼错只在运行期报错；新人上手慢 |
| B MyBatis-Plus | 继承 `BaseMapper`，条件用 `QueryWrapper` 链式拼装，配内置分页插件 | 单表 CRUD 零 SQL，出活快；内置逻辑删除、乐观锁、分页 | SQL 被框架封装，训练目标被架空；多表与复杂条件最终仍要写 XML，形成两套风格；框架升级牵动全仓库；与基线冲突 |
| C Spring Data JPA | 方法名派生查询 + `@Query` | 对象映射彻底，跨库移植性好 | 与索引 / 执行计划距离更远；`findByXxxAndYyy` 在复杂查询下失控；与既有 MyBatis 资产不兼容 |

### 决策

采用**方案 A：MyBatis 手写 Mapper XML**；全仓库禁止引入 MyBatis-Plus（含 `mybatis-plus-boot-starter` 及其分页插件）。

### 决策理由

1. **训练目标是 SQL 可见性**，不是交付速度：这是能力训练样本，SQL 被框架藏起来等于训练没发生。
2. **索引与门禁可度量**：`data-design.md` 要求每个高频查询给出 `EXPLAIN` 结论；手写 XML 时 SQL 就在眼前，评审可逐条对照。
3. **故障可定位**：慢查询日志里的 SQL 能直接映射到唯一一份 XML，不需要「反推框架生成了什么」。
4. **依赖面小**：只依赖 `mybatis-spring-boot-starter`，升级 Spring Boot 3.2.x 时不牵动 ORM 框架。

### 后果

- 正面：SQL、索引、执行计划三者互相印证；评审清单可精确到「这条 `<if>` 有没有对应索引」。
- 负面：单表 CRUD 样板代码量上升。补偿措施：`tools/gen` 生成 XML 骨架，用户只写 `<where>` 条件与业务 SQL；把「全表扫描」列为 G3 硬伤。

### 复审触发条件

- 连续 3 个项目出现同一类手写 XML 缺陷（如漏 `is_deleted = 0`），说明该改生成器规则或 `<sql>` 片段，而不是换框架。
- 某项目单表 CRUD 的 XML 超过 200 行且无复杂查询，手写成本不再换取训练收益。
- 出现真正需要数据库无关性的需求（当前为非目标）。

## 5. 示例二：ADR-002 为什么文件存储要抽象 `storage/`，默认本地磁盘

- 状态：ACCEPTED
- 日期：2026-01-05
- 关联需求：FILE、NFR-可测试性-002
- 关联设计：`docs/class-responsibility.md#1-分层职责与禁止事项`

### 背景

母版 §13 建议后端直连对象存储（OSS）上传文件，但本仓库的硬约束是：**CI 与陌生机器上没有云密钥**，`docs/dev-env.md` 要求「一台陌生机器可在 5 分钟内跑通」，验收底线是本地可复现。同时 P06 博客封面、P07 简历附件、P10 作品集导出都需要文件读写，接口形状必须统一。

### 候选方案

| 方案 | 做法 | 优点 | 代价 |
|---|---|---|---|
| A 直连 OSS | `FileController` 直接调 `OssUtils`，返回公网 URL | 实现最短；生产即真实形态；无需自建静态映射 | 本地与 CI 无密钥即跑不通，集成测试只能打桩或跳过；上传逻辑写死在 Controller，难替换；联调依赖外网 |
| B `storage/` 抽象 + 本地默认 + OSS 可选 | 定义 `StorageService`（`store / load / delete / url`），默认实现写本地磁盘并映射 `/files/**`，OSS 实现用 `@ConditionalOnProperty` 启用 | 本地与 CI 完全离线；接口契约稳定；上传校验集中一处；换存储只改配置 | 多一层接口与两套实现；本地 URL 与 OSS URL 形状不同，前端要按相对路径处理；多一套配置开关与文档 |
| C 只用本地磁盘，不做接口 | `FileController` 直接 `Files.copy` 到固定目录 | 最简单 | 生产无法切 OSS；测试难以替换；目录路径散落多处 |

### 决策

采用**方案 B**：`storage/StorageService` + `storage/impl/LocalDiskStorage`（`@ConditionalOnProperty(name = "app.storage.type", havingValue = "local", matchIfMissing = true)`）作为默认实现，`storage/impl/OssStorage` 仅在配置显式启用时装配。

### 决策理由

1. **CI 无密钥**是硬约束：默认实现必须离线可跑，OSS 只能是可选实现。
2. **接口稳定**：`store(MultipartFile) → storageKey`、`url(storageKey) → String` 在 P06/P07/P10 保持一致，前端不必感知后端换了存储。
3. **可测试**：Service 层依赖 `StorageService` 接口，集成测试注入指向临时目录的 `LocalDiskStorage` 即可，无需打桩云 SDK。
4. **校验集中**：后缀白名单、大小上限、文件名清洗只写在实现类入口，避免每个 Controller 各写一遍。

### 后果

- 正面：本地与 CI 离线可跑通；换存储只改 `app.storage.type`；上传校验有唯一落点。
- 负面：本地 URL 与生产 OSS URL 形状不同。补偿措施：`url()` 返回**可被前端直接使用的字符串**，并在 `api-contract.md` 写明该字段是相对路径还是绝对 URL。

### 复审触发条件

- 出现签名 URL、断点续传、多租户目录隔离等本地实现无法等价模拟的能力。
- 上传平均耗时超过 2s 且证据指向本地磁盘 IO，说明应默认启用远端存储。

## 6. 本仓库已确立、不需要重复讨论的决策

写 ADR 时遇到下面这些，直接引用本表，**不要**再开一份 ADR 论证一遍；要改基线请走 S3 复盘 → `docs/pipeline/upgrade-log.md`。

| 决策 | 内容 | 依据 | 需要开 ADR 吗 |
|---|---|---|---|
| 统一返回 | 所有接口返回 `Result<T>`，分页返回 `Result<PageResult<T>>` | `AGENTS.md` §3 | 否 |
| 分页形状 | `page` 从 1 开始，`pageSize` 有上限 | `api-contract.md` | 否 |
| 禁止 `record` | VO/DTO 用 Lombok POJO（`@Data @Builder @NoArgsConstructor @AllArgsConstructor`），实体只用 `@Data` | `AGENTS.md` §3 | 否 |
| 包结构 | `controller / service(+impl) / mapper / entity / dto / vo / query / config / constant / enumeration / exception / handler / interceptor / aspect / utils / storage(+impl)` | `AGENTS.md` §3 | 否 |
| 异常体系 | 公共基类持有 `code / message / httpStatus`，每个错误一个子类 | `AGENTS.md` §3 | 否 |
| ORM | MyBatis 手写 Mapper XML，禁止 MyBatis-Plus | ADR-001 | 否（已决） |
| 数据库 | 库 `java_fullstack_lab`，表前缀 `p0N_`，`utf8mb4` / `utf8mb4_general_ci` | `docs/dev-env.md` | 否 |
| 文件存储 | `storage/` 抽象 + 本地磁盘默认实现 | ADR-002 | 否（已决） |
| 秘密管理 | 连接串与密钥只走环境变量或 `application-local.yml` | `AGENTS.md` §3 | 否 |
| 注释风格 | 用 `//` 与 `/* */`，不写 Javadoc | `AGENTS.md` §3 | 否 |

## 填写检查清单

- [ ] 编号在仓库内全局唯一，未被其他项目占用。
- [ ] 状态是 `PROPOSED` / `ACCEPTED` / `SUPERSEDED` / `DEPRECATED` 之一，没有中文状态。
- [ ] 背景只写事实与约束，没有提前写结论。
- [ ] 候选方案 ≥2 个，且每个方案都写了代价。
- [ ] 决策是一句话，明确到「用 A 不用 B」。
- [ ] 决策理由至少一条是可验证的硬约束（CI 无密钥 / 训练目标 / 基线冲突）。
- [ ] 负面后果至少 1 条，并写明补偿措施。
- [ ] 复审触发条件可观测（有阈值或事件），不是「感觉不合适时」。
- [ ] 至少关联一个 `REQ-` / `BR-` / `NFR-` 编号。
- [ ] 若推翻历史决策：新 ADR 已开，旧 ADR 状态已改为 `SUPERSEDED` 并注明取代者。
- [ ] 已确认本决策不在第 6 节基线表内（若在，删掉这份 ADR，改为引用）。
