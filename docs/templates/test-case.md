# 测试用例

> **用途**：把一条验收条件变成一条**可执行、可判定真假、含负向断言**的测试用例，让"测过了"变成"有记录、能重跑"。
> **填写时机**：S3 验证与交付阶段、写测试代码**之前**；工序 19 分层测试的输入物，G4 发布闸门按用例结论判定。
> **产出位置**：`projects/P0N-<slug>/docs/testing/cases/TC-<模块>-<场景>-<两位序号>.md`（一个场景一份；同类小用例可合并成一个文件的多节）。

## 1. 编号与关联

- 编号：`TC-<模块>-<场景>-<两位序号>`，例：`TC-STU-CREATE-02`；模块与场景都**全大写**，场景用动词（`CREATE` / `QUERY` / `BORROW` / `RETURN`）。
- 每条用例至少关联一个 `REQ-<模块>-<两位序号>` 或 `BR-<模块>-<两位序号>`，并尽量关联一条 `AC-<模块>-<两位序号>`。
- 模块段取项目 `requirements.md` 第 0 节模块字典的短码（P01 → `STU`、P02 → `BOOK`），与代码里的模块名、表前缀一一对应，不写小写、不写中文。
- 一个 `AC-` 至少对应一条正向用例与一条负向用例；**只有正向用例的验收条件视为未覆盖**。

## 2. 字段

| 字段 | 必填 | 说明 |
|---|---|---|
| 用例编号 | 是 | 见上；与测试方法名里的编号保持一致，便于反查 |
| 关联需求/业务规则 | 是 | `REQ-` / `BR-` / `AC-` 编号，多个用逗号分隔 |
| 目标 | 是 | 一句话说清"证明什么"；不写"测试新增功能"这种空话 |
| 层级 | 是 | 单元 / 接口（Web 层）/ 集成 / 端到端；一层的用例不要断言另一层的事 |
| 环境与前置条件 | 是 | 用哪个库、哪个 profile、哪些数据必须先存在、时钟是否固定 |
| 测试数据 | 是 | 具体的值（含边界值），不写"一条合法数据"；标注是固定夹具还是随机生成 |
| 步骤 | 是 | 编号步骤；每步是"动作 → 观察点" |
| 预期结果 | 是 | 状态码 / 业务 `code` / 返回体关键字段 / 页面状态 |
| 不应发生的变化 | 是 | 负向断言：数据库不应新增的行、不应变化的字段、不应发出的调用 |
| 清理方式 | 是 | 删除插入的行 / 回滚事务 / 重建夹具；**必须写明**，否则测试不可重跑 |
| 实际结果 | 是 | 执行后回填：通过 / 失败原文（含断言消息） |
| 结论 | 是 | `PASS / FAIL / BLOCKED / SKIPPED`（SKIPPED 必须写理由） |

## 3. 用例设计六法（每种给一条 P01/P02 的具体例子）

| 方法 | 怎么用 | P01/P02 具体例子 |
|---|---|---|
| **等价类** | 把输入划成有效类与无效类，每类取一个代表值 | `age` 有效类 [18,120] 取 30；无效类"小于下界"取 10、"非数字"取 `"abc"`（`BR-STU-02`） |
| **边界值** | 在每个边界的内侧、边界上、外侧各取一点 | `age` 取 17/18/19 与 120/121；`pageSize` 取 1/10/100/101（`BR-STU-02`、`REQ-STU-02`） |
| **决策表** | 把多个条件组合成规则表，逐条覆盖 | P02 借书：`库存>0` × `读者未超借阅上限` × `书未挂失` → 允许借出；任一为否 → 拒绝且错误码不同（`BOOK/002/003`） |
| **状态迁移** | 列出合法状态与非法迁移 | P02 借阅单：`BORROWED → RETURNED` 合法；`RETURNED → RETURNED` 非法，应返回状态冲突而非静默成功（`BOOK`） |
| **场景法** | 串起一条真实用户旅程，验证端到端可用 | 管理员登录 → 录入学号 → 列表分页搜索到该生 → 编辑年龄 → 删除 → 列表查不到（P01 主流程） |
| **错误推测** | 凭经验列出最可能被写错的地方，优先测 | 学号大小写/前后空格、手机号 11 位、并发两次提交同一学号、`pageNum=0`、超长字符串、`null` 与空串的区别 |

## 4. 边界值清单（直接抄进测试数据）

| 字段 | 下界外 | 下界 | 下界内 | 上界内 | 上界 | 上界外 |
|---|---|---|---|---|---|---|
| `age`（P01） | 17 | **18** | 19 | 120 | **120** | 121 |
| `pageSize`（通用） | 0 | **1** | 10 | 100 | **100** | 101 |
| `pageNum`（通用） | 0（应归一为 1 或报错，二选一但必须写死） | 1 | 2 | — | 上限值 | 上限 +1 |
| `stock`（P02 借阅时的库存） | -1（非法） | **0（不可借）** | 1 | 借阅上限 | — | — |
| `borrowDays`（P02 借期） | 0 | 1 | 29 | 30 | **30** | 31 |

> 规则：边界值写法必须**给出具体数字**。"测试了边界情况"不是用例，"`age = 17` 应返回 400 + `code=10001`"才是。

## 5. AAA 结构与命名

用例体一律 Arrange / Act / Assert 三段，段间空行分隔；断言全放在 Assert 段，禁止在 Act 段里加判断。

```java
@Test
void shouldNotCreateStudentWhenStudentNoDuplicated() {
    // Arrange
    StudentCreateDTO dto = StudentFixtures.validDto("S2025001", "13800000001", 20);
    when(studentMapper.countByStudentNo("S2025001")).thenReturn(1);

    // Act + Assert
    assertThatThrownBy(() -> studentService.create(dto))
            .isInstanceOf(StudentNoDuplicatedException.class)
            .hasMessageContaining("S2025001");
    verify(studentMapper, never()).insert(any(Student.class));
}

@Test
void createStudent_whenAgeBelowMinimum_returnsBadRequest() throws Exception {
    // Arrange
    String body = """
            {"studentNo":"S2025002","name":"Zhang San","age":17,"phone":"13800000002"}
            """;

    // Act
    mockMvc.perform(post("/api/p01/students").contentType(APPLICATION_JSON).content(body))
            // Assert
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value(10001))
            .andExpect(jsonPath("$.data").doesNotExist());
}
```

命名要求：**方法名要表达业务行为**，读起来像一句话。两种写法都允许，同一项目内保持统一：

- `shouldNotCreateStudentWhenStudentNoDuplicated`（should-when 式，负向用例可读性最好）
- `createStudent_whenAgeBelowMinimum_returnsBadRequest`（method_when_then 式，接口用例常用）

禁止：`test1`、`testCreate`、`testCreate2`、`createTest`。

## 6. 断言清单（每类用例至少勾一遍）

- [ ] **状态**：HTTP 状态码 + 业务 `code`（两者都要断言，只断言 200 会漏掉错误码被吞的情况）
- [ ] **数据库副作用**：成功路径断言**存在**且字段值正确；失败路径断言相应的表**行数不变**
- [ ] **事件数量**：发布领域事件的功能，断言 `ApplicationEventPublisher` 被调用的次数与载荷
- [ ] **外部调用次数**：`verify(mapper, times(1))` / `verify(storage, never())`；错误路径最关键是 `never()`
- [ ] **不应发生的变化**：`age` 校验失败时 `updated_at` 不得变化；重复提交时不得产生第二条记录；越权时不得删除他人数据
- [ ] **可读性**：失败时的断言消息包含定位信息（学号、ID、期望值 vs 实际值）

## 7. Mock 的边界：什么必须用真实 MySQL 验证

Mock 只能证明"我调用了什么"，不能证明"数据库会怎么反应"。以下必须在真实 MySQL 8 上跑（工序 19 的持久层/集成层）：

| 必须真实 MySQL | 原因 |
|---|---|
| 唯一索引冲突（`uk_p01_student_no`） | 冲突由索引决定；Mock 里永远返回 1，测不出"索引根本没建" |
| 逻辑外键的引用完整性（P02 借阅单 → 图书） | 本仓库**禁止物理外键**，引用完整性由应用层保证：`book_id` 指向不存在的图书必须返回业务错误码，Mock 里永远校验不出真实数据状态 |
| 删除父记录前的引用检查 | "删除前查无引用方"依赖真实数据，且每个关联字段都要有"引用不存在"路径的用例 |
| 孤儿数据排查 SQL（`test/` 内） | 每个项目必须提供一条能查出孤儿行的 SQL，并纳入发布前检查 |
| 动态 SQL 条件（`<if>/<where>/<foreach>`） | 只在真实方言下才暴露拼接错误与 `WHERE` 遗漏 |
| 分页与排序（`LIMIT/OFFSET`、`ORDER BY`） | Mock 的 `selectPage` 无法验证条数与顺序 |
| 事务回滚与隔离（P03 库存、P04 乐观锁） | 只有提交/回滚边界真实存在时才有意义 |
| `@Transactional` 传播与自调用失效 | 需要真实代理，Mock 测不出代理失效 |
| MySQL 方言特性（`utf8mb4`、`DATETIME(3)`、`ON DUPLICATE KEY UPDATE`） | H2 语法近似但语义不同 |

明确禁止：**用 H2 代替 MySQL 验证关键 SQL**。H2 可以用于不涉及方言的单元级冒烟，但结论不得写进 G4 判定。

## 8. 示例（完整填充）

## TC-STU-CREATE-02 学号重复时拒绝新增

| 项 | 值 |
|---|---|
| 关联需求/规则 | `REQ-STU-01`, `BR-STU-01`, `AC-STU-02` |
| 目标 | 证明学号唯一性在 **Service 业务规则**与**数据库唯一索引**两层都生效，且失败不留脏数据 |
| 层级 | 单元（Service，Mock Mapper）+ 集成（真实 MySQL 触发 `uk_p01_student_no`）两条，结论取两者 |
| 环境与前置条件 | 库 `java_fullstack_lab`，profile `test`；`p01_student` 已存在且 `uk_p01_student_no` 已建；固定 `Clock` 为 `2025-03-11T10:00:00Z` |
| 测试数据 | 既有行：`student_no = 'S2025001'`；本次请求：`student_no = 'S2025001'`，`name = 'Zhang San'`，`age = 20`，`phone = '13800000009'` |
| 步骤 | 1. 先插入 `S2025001` 夹具（集成用例）<br>2. `POST /api/p01/students` 提交重复学号<br>3. 查询 `p01_student` 中 `student_no='S2025001'` 的行数与 `phone='13800000009'` 的行数 |
| 预期结果 | HTTP `409`；`code = 10003`；`message` 含学号；`data = null`；日志 `WARN` 中含学号且**不含**手机号明文 |
| 不应发生的变化 | `p01_student` 行数仍为 1；`phone='13800000009'` 的行数为 0；无 `INSERT` 语句执行；`updated_at` 未变化；未发出任何领域事件 |
| 清理方式 | 集成用例在 `@AfterEach` 执行 `DELETE FROM p01_student WHERE student_no IN ('S2025001','S2025009')`；单元用例无需清理 |
| 实际结果 | 单元：`assertThatThrownBy` 通过；集成：`DuplicateKeyException` 被 `GlobalExceptionHandler` 转换为 409 |
| 结论 | PASS |

**证据片段**

```text
POST /api/p01/students -> 409
{"code":10003,"message":"student no already exists: S2025001","data":null}
mysql> SELECT COUNT(*) FROM p01_student WHERE phone='13800000009';  -> 0
```

## 9. 空白副本（可整段复制）

```markdown
## TC-<模块>-<场景>-<两位序号> <一句话标题>

| 项 | 值 |
|---|---|
| 关联需求/规则 | <REQ-…>, <BR-…>, <AC-…> |
| 目标 | <证明什么> |
| 层级 | 单元 / 接口 / 集成 / 端到端 |
| 环境与前置条件 | <库、profile、夹具、Clock> |
| 测试数据 | <具体值，含边界值> |
| 步骤 | 1. <动作 → 观察点> 2. … |
| 预期结果 | <状态码 / code / 关键字段 / 页面状态> |
| 不应发生的变化 | <行数、字段、调用次数、事件> |
| 清理方式 | <删除 / 回滚 / 重建夹具> |
| 实际结果 | <PASS 或失败原文> |
| 结论 | <PASS / FAIL / BLOCKED / SKIPPED + 理由> |
```

## 填写检查清单

- [ ] 编号格式为 `TC-<模块>-<场景>-<两位序号>`（全大写、两位补零），且与测试方法名/用例文件能对应上
- [ ] 关联了 `REQ-`/`BR-`/`AC-` 编号；该 `AC-` 已有配对的正向与负向用例
- [ ] 测试数据是具体值，边界用例写出了具体数字（17/18/19、100/101 之类）
- [ ] 「不应发生的变化」非空，且是可执行的断言（不是"数据应保持一致"）
- [ ] 清理方式写明，用例可重复执行且结果稳定
- [ ] 涉及唯一索引 / 动态 SQL / 事务 / 方言的用例已标注走真实 MySQL
- [ ] 「实际结果」与「结论」在执行后回填，失败原文保留，不做美化
