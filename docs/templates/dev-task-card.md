# 开发任务卡（垂直切片）

> **用途**：把一条需求切成"从 Vue 到 MySQL 一条打穿"的最小可交付功能，并写清 AI 给了什么骨架、用户要写哪几段业务代码。
> **填写时机**：S2 实现阶段，**动手写第一行代码之前**。每张卡对应工序 13–18 的一段工作；G3 开发闸门按卡逐张判定。
> **产出位置**：`projects/P0N-<slug>/docs/modules/TASK-<模块>-<两位序号>.md`（模块多时按模块建子目录）。

## 1. 为什么本流水线用垂直切片

一张任务卡 = **一条端到端可用的小功能**，而不是"写完所有 Entity"这种水平切片。

| 维度 | 水平切片（反例） | 垂直切片（本流水线） |
|---|---|---|
| 切法 | 按技术层切：本周写完所有 Entity、下周写完所有 Mapper | 按功能切：一次只打通"新增学生"这一条链路 |
| 一张卡包含 | 一个层的全部类 | 一条链路的全部层：Vue → Controller → Service → Mapper XML → MySQL |
| 完成判据 | "这个包写完了" | "这个功能在浏览器里能跑通，且负向路径返回业务错误码" |
| 出错时 | 编译过了但没人能演示，问题在联调时集中爆发 | 问题立刻暴露在当次切片内，返工半径最小 |
| 演示价值 | 无法演示 | 每张卡都能现场演示给用户看 |
| 与门禁关系 | 只能在最后一次性过 G3，失败原因难定位 | 每张卡单独过 DoD，G3 只是汇总确认 |

**结论**：本流水线要求任何一张任务卡都能独立演示、独立测试、独立回滚，否则说明它切得不够垂直。

## 2. 任务卡字段

| 字段 | 必填 | 说明与填写要求 |
|---|---|---|
| 任务编号 | 是 | `TASK-<模块>-<两位序号>`，模块段全大写、序号两位补零，项目内唯一不复用 |
| 标题 | 是 | 动词开头，一句话说清用户可见的能力，例："新增学生" |
| 对应需求 | 是 | `REQ-<模块>-<两位序号>`；一个任务卡只服务一条需求 |
| 对应业务规则 | 是 | `BR-<模块>-<两位序号>`，逐个列出；没有对应规则说明这张卡里没有业务 |
| 验收条件 | 是 | `AC-<模块>-<两位序号>`，Given/When/Then 的编号；DoD 里的测试必须逐一覆盖 |
| 垂直切片边界 | 是 | 从 Vue 到 MySQL 的**文件路径级**清单，标注每项是 AI 产出还是用户填写 |
| 留给用户实现 | 是 | 逐条列出 `TODO(user)` 的位置与内容 |
| AI 已提供 | 是 | 骨架、配置、SQL 结构、签名、测试骨架、冒烟脚本 |
| 完成判据 DoD | 是 | 用第 6 节的 8 项清单，逐项勾选 |
| 估算 | 是 | 用"工序序号 + 小时"，例：`工序 13–16，约 4h（用户 3h / AI 1h）` |
| 依赖任务 | 是 | 前置任务编号；无依赖写 `无`，并说明为什么可以并行 |
| 状态 | 是 | `OPEN / IN_PROGRESS / BLOCKED / DONE`，与 `progress-ledger.md` 保持一致 |

> **编号口径**（与 `AGENTS.md` §3.7 一致）：模块段取项目 `requirements.md` 第 0 节**模块字典**里的短码，全大写、序号两位补零。
> 例：P01 学生模块 `STU` → `TASK-STU-01`、`REQ-STU-01`、`BR-STU-02`、`AC-STU-03`；P02 图书模块 `BOOK` → `TC-BOOK-BORROW-01`。
> 非功能编号 `NFR-<分类>-<两位序号>`，分类只用固定 8 类缩写：`PERF` / `SEC` / `AVAIL` / `RELIAB` / `MAINT` / `SCALE` / `COMPAT` / `OBS`。
> 不允许 `REQ-stu-1` 这类小写不补零写法，也不允许一个项目里 `student` 与 `STU` 混用。

## 3. 空白副本（可整段复制）

```markdown
# TASK-<模块>-<两位序号> <标题>

| 项 | 值 |
|---|---|
| 任务编号 | TASK-<模块>-<两位序号> |
| 对应需求 | REQ-<模块>-<两位序号> |
| 业务规则 | BR-<模块>-<两位序号>, BR-<模块>-<两位序号> |
| 验收条件 | AC-<模块>-<两位序号>, AC-<模块>-<两位序号> |
| 估算 | 工序 <n>–<n>，约 <n>h（用户 <n>h / AI <n>h） |
| 依赖任务 | TASK-<模块>-<两位序号> / 无 |
| 状态 | OPEN |

## 垂直切片边界

| 层 | 文件 | 动作 | 归属 |
|---|---|---|---|
| MySQL | `projects/P0N-<slug>/database/migrations/V<n>__<说明>.sql` | 新表 / 加字段（只含 `p0N_` 前缀） | AI |
| Vue | `frontend/src/views/<模块>/<X>View.vue` | 页面骨架 | AI |
| ... | ... | ... | AI / USER |

## 留给用户实现（TODO）

1. `service/impl/<X>ServiceImpl.java#<method>` —— <要做什么> - <AC 编号>
2. `resources/mapper/<X>Mapper.xml#<statementId>` —— <要做什么> - <AC 编号>

## 完成判据（DoD）

见 [dev-task-card.md](../../../templates/dev-task-card.md) 第 6 节，8 项全部勾选才允许把状态改为 DONE。
```

## 4. 示例：TASK-STU-01 新增学生

### 4.1 基本信息

| 项 | 值 |
|---|---|
| 任务编号 | `TASK-STU-01` |
| 标题 | 新增学生 |
| 对应需求 | `REQ-STU-01`（管理员录入学生基础信息） |
| 业务规则 | `BR-STU-01`（学号全局唯一）、`BR-STU-02`（年龄 18–120）、`BR-STU-03`（手机号 11 位且唯一） |
| 验收条件 | `AC-STU-01`（成功入库）、`AC-STU-02`（学号重复返回冲突）、`AC-STU-03`（年龄越界返回参数错误） |
| 估算 | 工序 13–16，约 4h（用户 3h / AI 1h） |
| 依赖任务 | 无（本卡是 P01 第一条垂直切片，其余卡都依赖它建立的分层骨架） |
| 状态 | IN_PROGRESS |

### 4.2 垂直切片边界（文件路径级）

包根示例为 `com.jfsl.p01`，实际以项目 `pom.xml` 的 `groupId` 为准。

| 层 | 文件 | 动作 | 归属 |
|---|---|---|---|
| MySQL | `projects/P01-student-info/database/migrations/V1__init_student.sql` | 建 `p01_student`，含 `uk_p01_student_no`、`uk_p01_student_phone`（只允许出现 `p01_` 前缀） | AI |
| MySQL | `projects/P01-student-info/database/seed.sql` | 插 3 条合法种子 + 1 条重复手机号（供发现记录引用） | AI |
| Entity | `backend/src/main/java/com/jfsl/p01/entity/Student.java` | Lombok `@Data` POJO，无 `record` | AI |
| DTO | `.../dto/StudentCreateDTO.java` | 字段 + `@NotBlank/@Min/@Max/@Pattern` | AI |
| VO | `.../vo/StudentVO.java` | 返回体，不含 `deleted` 等内部字段 | AI |
| Mapper 接口 | `.../mapper/StudentMapper.java` | `int insert(Student student)`、`int countByStudentNo(String)` | AI（签名） |
| Mapper XML | `backend/src/main/resources/mapper/StudentMapper.xml` | `insert` 与 `countByStudentNo` 的 SQL | **USER** |
| Service 接口 | `.../service/StudentService.java` | `Long create(StudentCreateDTO dto)` | AI（签名） |
| Service 实现 | `.../service/impl/StudentServiceImpl.java` | 唯一性校验 + 组装 Entity + 事务边界 | **USER** |
| 异常 | `.../exception/StudentNoDuplicatedException.java` | 专属子类，携带 `code` 与 `httpStatus` | AI |
| Controller | `.../controller/StudentController.java` | `@PostMapping` + `@Valid` + 返回 `Result<Long>` | AI |
| 测试 | `backend/src/test/java/com/jfsl/p01/service/StudentServiceImplTest.java` | 用例骨架（`@Test` 与 Mock 已就位，断言为 TODO） | AI 骨架 / **USER 断言** |
| Vue | `frontend/src/views/student/StudentCreateView.vue` | 表单骨架 + 校验规则 + 提交按钮 | AI 骨架 / **USER 交互** |
| Vue | `frontend/src/api/student.ts` | `createStudent(payload)` 请求函数 | AI |
| Vue | `frontend/src/types/student.ts` | `StudentCreatePayload` 类型 | AI |

### 4.3 留给用户实现的部分

1. `StudentMapper.xml#insert`、`#countByStudentNo` —— 写 `#{}` 参数化 SQL，插入后回填自增主键；`countByStudentNo` 必须走唯一索引 - `AC-STU-02`
2. `StudentServiceImpl#create` —— 先查重、再插入；命中重复时抛 `StudentNoDuplicatedException`；整个方法一个事务 - `AC-STU-02`
3. `StudentServiceImplTest#shouldNotCreateStudentWhenStudentNoDuplicated` —— 写断言与 `verify` 次数 - `AC-STU-02`
4. `StudentCreateView.vue` —— 提交前表单校验、按钮 `submitting` 态、失败时展示后端 `message` - `AC-STU-01` / `AC-STU-03`

### 4.4 AI 已提供的部分

- 可启动骨架、统一 `Result<T>`、全局异常处理器、日志配置、`application-local.yml.example`
- 上表所有类的**结构、字段、签名**与 XML 骨架（`<insert>` 框架已给，SQL 为空）
- 测试类骨架（`@ExtendWith(MockitoExtension.class)`、`@Mock`、`@InjectMocks` 已就位）
- 冒烟脚本 `tools/smoke/P01.ps1` 中"新增学生"一节
- 本任务卡与对应的讲解

### 4.5 完成判据（DoD）

见第 6 节 8 项；本卡若未覆盖 `AC-STU-03` 的负向断言，不得标记 DONE。

## 5. TODO(user) 的写法

必须写成 `// TODO(user): <要做什么> - <验收条件编号>`，让"要做什么"和"拿什么判"在同一行。

```java
@Override
public Long create(StudentCreateDTO dto) {
    // TODO(user): 校验学号唯一（先 count 再 insert），重复时抛 StudentNoDuplicatedException - STU
    // TODO(user): 校验手机号唯一，重复时抛 PhoneDuplicatedException - STU
    Student student = StudentConverter.toEntity(dto);
    // TODO(user): 补全审计字段（createdAt 用固定 Clock 注入，禁止 System.currentTimeMillis()）- NFR-MAINT-01
    studentMapper.insert(student);
    return student.getId();
}
```

反例（不允许出现在交付中）：

```java
// 校验一下                        <- 没说要校验什么、拿什么判
// TODO: 这里以后再说               <- 没有验收条件编号，无法判定完成
```

## 6. DoD 8 项清单（G3 开发闸门按此判）

1. [ ] **编译通过**：`mvnw.cmd -q clean compile` 与 `pnpm build` 均无错误（警告需在卡里说明）。
2. [ ] **迁移在空库可执行**：`database/migrations/V*.sql` 按版本顺序在全新 `java_fullstack_lab` 上从零跑通，`seed.sql` 可重复执行，无手工补丁。
3. [ ] **权限与异常路径已实现**：该卡涉及的 401/403/404/409/400 路径都有对应业务异常子类，且被全局处理器接住；卡片涉及的每个关联字段都有"引用不存在"路径（禁用物理外键，引用完整性由应用层保证）。
4. [ ] **自动化测试通过**：本卡对应 `AC-` 编号至少一条正向 + 一条负向用例，`mvnw.cmd test` 全绿。
5. [ ] **前后端联调完成**：浏览器里手工走通一次成功路径与一次失败路径，失败时页面显示后端 `message` 而非 500 页面。
6. [ ] **配置文档更新**：新增的环境变量 / 端口 / 迁移步骤已写入项目 `README.md` 与 `deploy/runbook.md`。
7. [ ] **无遗留阻断缺陷**：本卡相关缺陷状态为 `CLOSED`，未关闭项已记入 `discovery-record` 或发布记录。
8. [ ] **门禁命令通过**：`.\tools\lab.ps1 verify --project P0N` 返回成功（编译 + 测试 + 前端构建 + 秘密扫描）。

## 填写检查清单

- [ ] 任务编号、需求编号、业务规则编号、验收条件编号四者齐全且格式合规
- [ ] 垂直切片边界列到了**文件路径级**，每项标了 AI / USER 归属
- [ ] 每个 `TODO(user)` 都有验收条件编号，没有"以后再补"式描述
- [ ] DoD 8 项逐项可判定（不是"差不多""基本完成"）
- [ ] 估算写了工序区间与小时数，且与 `progress-ledger.md` 记录一致
- [ ] 依赖任务明确，或写清了并行不冲突的理由
- [ ] 本卡完成后，`progress-ledger.md` 的对应工序状态与门禁状态同步更新
