# 类职责与调用链

> **用途**：固定每一层「只做什么、禁止做什么」，并把一个用例从 Vue 到 MySQL 的完整链路写成可核对的箭头图，让 G2 能判定设计可以直接落到代码。
> **填写时机**：S1 契约冻结之后、生成任务清单之前（逐类落到实现前的最后一步）。
> **产出位置**：`projects/P0N-<slug>/docs/class-responsibility.md`；代码落到 `backend/src/main/java/<组ID>/p0N/` 与 `frontend/src/`。

## 1. 分层职责与禁止事项

| 层 / 包 | 只做 | 禁止 |
|---|---|---|
| `controller` | 接收请求、`@Valid` 校验、调 Service、返回 `Result<T>` | 写 SQL、直接依赖 `mapper`、写业务判断、逐个 `try/catch`、做 DTO↔Entity 转换 |
| `service`（接口） | 声明业务能力与入参 / 出参类型 | 出现 MyBatis 类型、出现 `HttpServletRequest`、写具体实现 |
| `service/impl` | 业务规则、事务编排、DTO→Entity、Entity→VO、调 mapper | 拼 SQL、越过 mapper 访问数据库、注入另一个 ServiceImpl 形成环 |
| `mapper`（接口） | 方法签名与结果类型，一行一操作 | 写业务判断、一律返回 `Map<String,Object>`、继承 MyBatis-Plus 的 `BaseMapper` |
| `mapper/*.xml` | SQL 与动态条件、结果映射 | 写业务规则（如"状态必须是 BORROWED 才能归还"）、拼白名单外的排序字段、出现别的项目表前缀 |
| `entity` | 与表一一对应的字段，只加 `@Data` | 加 `@Builder`、写业务方法、出现在 Controller 入参 / 返回值 |
| `dto` | 请求参数 + 校验注解 | 新增场景带上 `id`、出现 `is_deleted` / `created_at` 这类库表字段 |
| `vo` | 响应字段 + 展示所需派生字段（如 `className`） | 出现 `is_deleted` / 密码等敏感字段、写业务方法、被当作数据库查询结果直接接收 |
| `query` | 查询条件 + 分页参数（`page` / `pageSize` / `sortBy` / `sortOrder`） | 出现写操作字段、被当作多用途容器到处传 |
| `exception` | 每个错误一个子类；基类持有 `code` / `message` / `httpStatus` | 通用 `ApiException` 包打天下、集中式静态工厂 |
| `handler` | 全局异常处理，把异常转成 `Result` | 逐个 Controller 写 `try/catch`、把堆栈写进 `message` |
| `storage(+impl)` | 文件读写与 URL 生成（P06 起才引入） | 业务规则、把文件路径散落在 Controller |

代码风格：注释只用 `//` 与 `/* */`，**不写 Javadoc**；Controller 不写业务分支，Service 不碰 Web 对象。

## 2. 类清单模板

| 全限定类名 | 所在包 | 职责一句话 | 依赖 | 对应需求 |
|---|---|---|---|---|
| `<组ID>.p01.controller.StudentController` | `controller` | 承接学生模块 HTTP 请求 | `StudentService` | `REQ-STU-01` |
| `<组ID>.p01.service.StudentService` | `service` | 声明学生模块业务能力 | — | `REQ-STU-01` |
| `<组ID>.p01.service.impl.StudentServiceImpl` | `service.impl` | 实现学生业务规则与事务 | `StudentMapper`、`ClassMapper` | `BR-STU-02` |
| `<组ID>.p01.mapper.StudentMapper` | `mapper` | 学生表数据访问 | — | `REQ-STU-03` |
| `<组ID>.p01.entity.Student` | `entity` | 映射 `p01_student` | — | `REQ-STU-01` |
| `<组ID>.p01.dto.StudentCreateDTO` | `dto` | 新增请求参数与校验 | — | `REQ-STU-02` |
| `<组ID>.p01.vo.StudentVO` | `vo` | 列表 / 详情响应 | — | `REQ-STU-03` |
| `<组ID>.p01.query.StudentQuery` | `query` | 列表查询条件与分页 | — | `REQ-STU-04` |
| `<组ID>.p01.exception.StudentNotFoundException` | `exception` | `STUDENT_NOT_FOUND` | `BusinessException` | `BR-STU-05` |

填写规则：

- 全限定名用项目组 ID 前缀（示例 `com.jfsl.p01`），接口与实现各占一行，不合并。
- 每行「对应需求」至少一个 `REQ-` / `BR-` / `NFR-` 编号；写不出来的类先别进清单。
- 「依赖」列只写类名（不写包），用于人工检查依赖方向；出现 `mapper → service` 或 `service ↔ service` 即为环，必须拆。
- `entity` / `dto` / `vo` / `query` 每个模块至少各一行；异常子类按错误码逐个列。
- 清单必须覆盖所有新增类：漏一类，判据 5 不通过。

## 3. 调用链模板

```text
StudentList.vue（点击"新增"并提交表单）
  ↓ api/student.ts: createStudent(dto)
  ↓ axios（请求拦截器加 token；响应拦截器拆 Result，非 200 抛错）
  ↓ POST /api/v1/students
StudentController.create(@Valid StudentCreateDTO dto)
  ↓ studentService.create(dto)
StudentServiceImpl.create(dto)        ↓ 【事务开始】
  ↓ 业务校验：班级是否存在？学号是否重复？
Student entity（@Data，Builder 组装）
  ↓ studentMapper.insert(entity)
StudentMapper.xml <insert id="insert">
  ↓ INSERT INTO p01_student (...) VALUES (...)   ← 列名 snake_case，#{} 占位
MySQL（uk_p01_student_student_no 兜底学号唯一） ↓ 【事务结束】
  ↓ 返回主键
Result<Long>（Controller 包装）
  ↓ axios 响应拦截器
页面：关闭弹窗 → page = 1 → 重新查询列表
```

分层转换规则：

| 转换 | 在哪里做 | 不在这里做 |
|---|---|---|
| `DTO → Entity` | `ServiceImpl`（Builder 组装，忽略库表审计字段） | Controller、Mapper XML |
| `Entity → VO` | `ServiceImpl` | Controller、XML 的 `resultType`（统计类查询除外，需在类清单说明） |
| `List<Entity> → PageResult<VO>` | `ServiceImpl`（含 `total` / `totalPages` 计算） | 前端、Controller |
| `Query → SQL 参数` | `ServiceImpl` 计算 `offset` 后把 `Query` 直接传给 Mapper | XML 里再算一遍分页偏移 |

## 4. 事务边界标注

- **从哪开始 / 到哪结束**：从 `ServiceImpl` 的 public 业务方法进入开始（`@Transactional(rollbackFor = Exception.class)`），到方法返回或抛出未捕获异常结束。在链路图上用 `【事务开始】` / `【事务结束】` 标出来。
- **必须在同一事务**：多表写（借阅记录 + 扣减可借数量）、状态迁移 + 流水记录、唯一性校验 + 插入（配合 `uk_` 唯一索引兜底）。
- **不能放进事务**：文件读写 / 上传、HTTP 远程调用、消息发送、大批量导入（分批提交）、长耗时计算。
- **只读查询**：不加事务；确需一致性快照时用 `@Transactional(readOnly = true)`，且同一事务里不得写。
- **自调用失效**：`this.other()` 不走代理，事务不生效；要么拆到另一个 Service，要么保证是外部代理调用。
- **异常与回滚**：默认只有 `RuntimeException` 才回滚；业务异常基类必须继承 `RuntimeException`；捕获后不再抛 = 不回滚，所以捕获后必须转成业务异常再抛。
- **并发与重复请求**：事务内先查后插并不能防并发，统一由 `uk_` 唯一索引兜底并映射成业务错误码（如 `STUDENT_NO_DUPLICATED`）。

## 5. 示例：P01「新增学生」完整调用链（标注留给用户的实现位）

```java
// <组ID>.p01.controller.StudentController —— AI 提供，完整可编译
@RestController
@RequestMapping("/api/v1/students")
@RequiredArgsConstructor
public class StudentController {

    private final StudentService studentService;

    @PostMapping
    public Result<Long> create(@Valid @RequestBody StudentCreateDTO dto) {
        return Result.success(studentService.create(dto), "新增成功");
    }
}
```

```java
// <组ID>.p01.service.impl.StudentServiceImpl —— 骨架由 AI 提供，业务规则留给用户
@Service
@RequiredArgsConstructor
public class StudentServiceImpl implements StudentService {

    private final StudentMapper studentMapper;
    private final ClassMapper classMapper;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long create(StudentCreateDTO dto) {
        // TODO(user): 用 classMapper 校验班级存在且未删除，不存在抛 ClassNotFoundException(CLASS_NOT_FOUND) - TC-STU-CREATE-04
        Student entity = Student.builder()
                .studentNo(dto.getStudentNo())
                .name(dto.getName())
                .gender(dto.getGender())
                .classId(dto.getClassId())
                .enrollYear(dto.getEnrollYear())
                .isDeleted(0)
                .build();
        // TODO(user): 调 studentMapper.insert(entity)，捕获 DuplicateKeyException 转抛 StudentNoDuplicatedException(STUDENT_NO_DUPLICATED) - TC-STU-CREATE-02
        return entity.getId();
    }
}
```

```java
// <组ID>.p01.mapper.StudentMapper —— AI 提供接口签名
@Mapper
public interface StudentMapper {
    int insert(Student entity);
}
```

```xml
<!-- backend/src/main/resources/mapper/StudentMapper.xml —— 骨架由 AI 提供 -->
<insert id="insert" parameterType="<组ID>.p01.entity.Student"
        useGeneratedKeys="true" keyProperty="id">
  <!-- TODO(user): 写出 INSERT INTO p01_student 的列（snake_case）与对应 #{} 占位 - TC-STU-CREATE-01 -->
</insert>
```

```ts
// frontend/src/api/student.ts —— AI 提供
export function createStudent(dto: StudentCreateDTO) {
  return request.post<number>('/api/v1/students', dto)
}
```

```vue
<!-- frontend/src/components/StudentFormDialog.vue —— 交互与状态留给用户 -->
<script setup lang="ts">
// TODO(user): 按 api-contract.md#6 写校验规则；提交中禁用按钮；成功后 emit('success') 并关闭弹窗 - TC-STU-CREATE-01
</script>
```

分工对照（AI 搭骨架 + 用户写业务代码）：

| 位置 | 谁负责 | 交付标准 |
|---|---|---|
| Controller / DTO / Entity / VO / Query / Mapper 接口 / XML 骨架 / `api.ts` | AI | 可编译、签名与契约一致 |
| `ServiceImpl` 的业务规则与事务编排 | 用户 | 每个 `TODO(user)` 被替换，`TC-` 用例通过 |
| Mapper XML 的 SQL 与动态条件 | 用户 | `EXPLAIN` 命中预期索引，SQL 只出现本项目前缀 |
| Vue 页面交互与状态 | 用户 | 五个状态齐备，联调无字段缺失 |
| 测试断言与用例设计 | 用户 | 覆盖 `TC-` 编号，含「引用不存在」路径 |

## 6. 分层越界反例

| 反例 | 为什么错 | 正确做法 |
|---|---|---|
| Controller 注入 `StudentMapper` 直接查库 | 绕过业务规则与事务，权限与校验没有落点 | Controller 只调 Service，Mapper 只被 Service 调用 |
| Service 里用 `HttpServletRequest` 取 `userId` / IP | 与 Web 容器耦合，无法单元测试，无法复用到批处理 | 在 Controller 或 Interceptor 取好，作为方法参数传进 Service |
| Mapper XML 里写业务判断（用 `<if>` 决定能不能归还） | 业务规则散进 SQL，测试覆盖不到，多入口不一致 | 状态判断在 Service；XML 只负责按条件取数与更新 |
| Entity 直接作为 Controller 入参 / 返回值 | 暴露 `is_deleted` 等库表字段，前端可改审计字段，校验无处可加 | 入参 DTO、出参 VO，转换在 ServiceImpl |
| VO / DTO 用 `record`，或实体加 `@Builder` | 违反基线，实体被当传输对象用 | VO/DTO 用 Lombok 四注解；实体只 `@Data` |
| Mapper 查询一律返回 `Map<String,Object>` | 字段名拼错到运行期才暴露，契约悄悄漂移 | 返回明确的 Entity / VO；统计类查询建专用 VO |
| Controller 里逐个 `try/catch` 返回 `Result` | 响应结构不统一，错误码散落各处 | 抛业务异常，交 `GlobalExceptionHandler` 统一转换 |
| `ServiceImpl` 之间互相注入 | 形成循环依赖，事务嵌套难推理 | 抽一个上层编排 Service，或把查询下沉到 Mapper；依赖保持单向 |
| 事务加在 Controller 方法上，或自调用 `this.xxx()` | 事务范围错、代理失效导致不回滚 | 事务只在 `ServiceImpl` 的 public 方法上，跨方法调用走代理 |
| `utils` 里塞业务规则并做静态注入 | 规则藏进工具类，评审看不见 | 规则写在 `ServiceImpl`；工具类只做无状态转换 |

## 填写检查清单

- [ ] 十二层的「只做」「禁止」都已对照本项目检查过，没有例外未记录。
- [ ] 类清单覆盖所有新增类（含异常子类），每行有全限定名、包、职责、依赖、需求编号。
- [ ] 依赖方向单向：无 `mapper → service`、无 `service ↔ service` 互调、无 Controller → Mapper。
- [ ] 至少一条完整用例的链路写到 XML 与 SQL 粒度，含 `Result` 回到页面的收尾。
- [ ] 事务边界在链路图上标了起点与终点，并写明哪些写操作必须同事务、哪些必须放在事务外。
- [ ] 业务异常基类继承 `RuntimeException`（否则不回滚），已核对。
- [ ] 并发与重复请求由唯一索引兜底，且映射到具体错误码。
- [ ] 所有留给用户的实现位都写成 `// TODO(user): <做什么> - <TC- 或验收编号>`，没有替用户实现业务规则。
- [ ] 第 6 节反例逐条核对过本项目代码与设计，没有命中项。
- [ ] 调用链里出现的类都能在第 2 节清单里找到（清单与链路互不矛盾）。
