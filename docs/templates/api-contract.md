# API 契约

> **用途**：把「页面要什么、后端给什么」写成双向可核对的表，让 G2 能判定契约可以冻结，S2 期间前后端不再互相猜字段。
> **填写时机**：S1 设计阶段，与 `page-state.md` **并行互校**；G2 通过即冻结（冻结方式见第 1 节）。
> **产出位置**：`projects/P0N-<slug>/docs/api-contract.md`；需要机器可读时同步导出 `docs/openapi.yaml`（SpringDoc 反查核对）。

## 1. 契约冻结规则

冻结对象（四样，缺一不可）：**路径与方法**、**请求 / 响应字段名**、**错误码**、**权限**。

```text
S1 起草 ──> 与 page-state.md 互校 ──> G2 评审 ──> 冻结（记录 commit hash）
                                       ↑                    │
                          变更影响卡 ───┴──── 必须改时解冻 ←───┘
```

- 冻结后 S2 不得私自改路径、字段名、错误码；要改必须走变更影响卡，并同步 4 处：本文件、`page-state.md`、`openapi.yaml`、`docs/tasks.md`。
- 字段名大小写即契约：后端 VO 用 `camelCase`（`studentNo`），数据库列用 `snake_case`（`student_no`），前端一律用 `camelCase`。
- 契约以「页面动作」为最小粒度：**每个端点至少被一个页面动作使用**（判据见第 9 节）。

## 2. 资源命名（RESTful）

规则：路径用**名词复数** + 全小写 + 连字符（`/student-profiles`）；资源标识放路径段（`/{id}`）；查询条件放查询参数（`?classId=12`）；动作型状态迁移用子资源（`POST /borrows/{id}/return`）；查询参数用 `camelCase`。

| 反例 | 正例 | 为什么 |
|---|---|---|
| `POST /addUser` | `POST /users` | 动词进路径，方法和路径语义重复 |
| `GET /getUserById?id=3` | `GET /users/{id}` | 标识应在路径段，便于缓存与权限匹配 |
| `POST /user/update` | `PUT /users/{id}` | 更新用 HTTP 方法表达，路径保持名词 |
| `GET /deleteUser?id=3` | `DELETE /users/{id}` | `GET` 不允许有副作用 |
| `GET /getBorrowListByBook?bookId=1` | `GET /borrows?bookId=1` | 过滤是查询参数，不是新资源 |
| `POST /borrow/return?id=1` | `POST /borrows/{id}/return` | 状态迁移建模为子资源动作 |
| `GET /student_list` | `GET /students` | 下划线 + 单复数混用，风格不统一 |

统一前缀 `/api/p0N`（项目号进路径：P01 → `/api/p01`，P02 → `/api/p02`），例如 `GET /api/p01/students`；该前缀之外只放 `/files/**` 静态映射与 `/actuator/health`。

## 3. 端点表模板

| 方法 | 路径 | 用途 | 请求 DTO | 响应 VO | 成功码 | 主要错误码 | 权限 |
|---|---|---|---|---|---|---|---|
| `GET` | `/api/p0N/<资源>` | <列表 / 分页查询> | `<XxxQuery>` | `PageResult<XxxVO>` | 200 | `COMMON_PARAM_INVALID` | `<ANONYMOUS>` |
| `GET` | `/api/p0N/<资源>/{id}` | <详情> | 路径参数 | `<XxxVO>` | 200 | `<XXX>_NOT_FOUND` | `<ANONYMOUS>` |
| `POST` | `/api/p0N/<资源>` | <新增> | `<XxxCreateDTO>` | `Long`（新主键） | 201 | `COMMON_PARAM_INVALID`、`<XXX>_NO_DUPLICATED` | `<ROLE_ADMIN>` |
| `PUT` | `/api/p0N/<资源>/{id}` | <修改> | `<XxxUpdateDTO>` | `Void` | 200 | `<XXX>_NOT_FOUND`、`<XXX>_STATE_CONFLICT` | `<ROLE_ADMIN>` |
| `DELETE` | `/api/p0N/<资源>/{id}` | <删除（逻辑）> | 路径参数 | `Void` | 200 | `<XXX>_NOT_FOUND` | `<ROLE_ADMIN>` |

填表规则：

- 「请求 DTO」只能是 `dto` 包的类；「响应 VO」只能是 `vo` 包的类；查询条件用 `query` 包的类。**实体不得出现在这两列**。
- 「权限」列每行必须有值：`ANONYMOUS` / `ROLE_ADMIN` / `OWNER`（数据归属校验，落点在 Service → Mapper 的 `WHERE`）。
- 一个端点只写「主要错误码」（1–3 个），完整错误码在第 5 节表里。
- 端点数与 `page-state.md` 的页面动作数必须相等，否则判据 4 不通过。

## 4. 统一响应形状

```json
// Result<T>：所有端点
{ "code": "OK", "message": "OK", "data": { } }

// PageResult<T>：所有分页端点，放在 Result.data 内
{
  "code": "OK",
  "message": "OK",
  "data": {
    "page": 1,
    "pageSize": 10,
    "total": 37,
    "totalPages": 4,
    "list": [ { "id": 1, "studentNo": "2026001", "name": "张三" } ]
  }
}
```

```java
// <组ID>.common 约定包名：result 放在 vo 包外的公共位置由脚手架决定；禁止 record
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Result<T> {
    private String code;       // "OK" 表示成功；失败取第 5 节错误码表里的错误码字符串
    private String message;    // 用户可见文案，前端可直接展示
    private T data;            // 失败时可为 null 或字段错误列表
}

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class PageResult<T> {
    private Integer page;      // 从 1 开始
    private Integer pageSize;  // 上限见第 7 节
    private Long total;        // 满足条件的总行数
    private Integer totalPages;// 由 total 与 pageSize 计算，前端不自己算
    private List<T> list;      // 当前页数据
}
```

字段说明：`code` 是业务错误码字符串（成功固定 `"OK"`），与 HTTP 状态码**同时**表达结果（HTTP 给网关与浏览器，`code` 给业务分支）；`message` 是用户可见文案，禁止把异常堆栈或 SQL 片段放进来；分页的 `total` 必须在同一次查询里用 `COUNT(*)` 得到，不允许前端累加。

## 5. 错误码表模板

| 错误码 | HTTP 状态 | 触发条件 | 用户可见文案 | 异常子类（`exception` 包） |
|---|---|---|---|---|
| `COMMON_PARAM_INVALID` | 400 | `@Valid` 校验失败 / 类型转换失败 | 参数校验失败，请检查输入 | `ParamInvalidException` |
| `<模块>_NOT_FOUND` | 404 | 按 id 查不到或已逻辑删除 | <该记录不存在或已被删除> | `<模块>NotFoundException` |
| `<模块>_NO_DUPLICATED` | 409 | 业务唯一键重复（唯一索引冲突） | <该编号已存在> | `<模块>NoDuplicatedException` |
| `<模块>_STATE_CONFLICT` | 409 | 当前状态不允许该操作（如已归还再归还） | <当前状态不支持该操作> | `<模块>StateConflictException` |
| `<模块>_REFERENCED` | 409 | 被其它记录引用，不可删除 | <存在关联数据，无法删除> | `<模块>ReferencedException` |
| `COMMON_INTERNAL_ERROR` | 500 | 未捕获异常 | 服务器开小差了，请稍后再试 | `CommonInternalErrorException` |

规则：

- 错误码 = **大写下划线**，模块前缀 + 语义后缀（`TICKET_STATE_CONFLICT`、`STUDENT_NO_DUPLICATED`），**不把 HTTP 状态写进名字**。
- `Result.code` 用的就是本表的错误码字符串（成功固定 `"OK"`）；HTTP 状态由响应行与异常基类的 `httpStatus` 决定，两者不互相替代、也不写进彼此的名字。
- **一个错误码对应一个异常子类**，落在 `exception` 包；禁止「集中式静态工厂 + 通用 ApiException」。
- 基类持有 `code / message / httpStatus`，由 `handler` 包的全局异常处理器统一转成 `Result`；Controller 不写 `try/catch`。
- 同一个错误码的文案只有一处来源（异常子类的构造），前端不再改字。

## 6. 请求校验规则

```java
// dto/StudentCreateDTO.java —— VO/DTO 一律 Lombok POJO，禁止 record
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class StudentCreateDTO {

    @NotBlank(message = "学号不能为空")
    @Pattern(regexp = "^[0-9]{8}$", message = "学号必须是 8 位数字")
    private String studentNo;

    @NotBlank(message = "姓名不能为空")
    @Size(max = 30, message = "姓名长度不能超过 30")
    private String name;

    @NotNull(message = "班级不能为空")
    @Positive(message = "班级 id 必须为正数")
    private Long classId;

    @NotNull(message = "入学年份不能为空")
    @Min(value = 2000, message = "入学年份不合法")
    @Max(value = 2100, message = "入学年份不合法")
    private Integer enrollYear;
}
```

| 规则 | 做法 |
|---|---|
| 触发入口 | Controller 方法参数上加 `@Valid`（嵌套对象加 `@Valid`），不加 `@Validated` 分组时默认全量校验 |
| 分组校验 | 新增用 `Save.class`、修改用 `Update.class`，`id` 只在 `Update.class` 里 `@NotNull` |
| 校验失败结构 | `code = "COMMON_PARAM_INVALID"`，`message = "参数校验失败"`，`data` 为 `List<FieldErrorVO>`（`vo` 包） |
| 统一出口 | `MethodArgumentNotValidException` / `ConstraintViolationException` 在 `GlobalExceptionHandler` 里转成上面的结构 |
| 禁止 | 在 Controller 里写 `if (dto.getXxx() == null) return Result.fail(...)`；在 Service 里重复做格式校验（业务规则校验归 Service） |

```json
// 校验失败的响应
{
  "code": "COMMON_PARAM_INVALID",
  "message": "参数校验失败",
  "data": [
    { "field": "studentNo", "message": "学号必须是 8 位数字" },
    { "field": "name", "message": "姓名不能为空" }
  ]
}
```

## 7. 分页、排序与查询参数约定

| 参数 | 类型 | 默认 | 约束 | 说明 |
|---|---|---|---|---|
| `page` | `int` | 1 | ≥1，**从 1 开始** | 小于 1 视为 1 |
| `pageSize` | `int` | 10 | 1–100 | 超过 100 取 100（不做静默变 0） |
| `sortBy` | `String` | `id` | **白名单**：`id` / `studentNo` / `name` / `createdAt` | 白名单外的值返回 `COMMON_PARAM_INVALID` |
| `sortOrder` | `String` | `asc` | `asc` / `desc` | 大小写不敏感，转小写后再判断 |
| `keyword` | `String` | 无 | 去首尾空格，长度 ≤ 50 | 模糊匹配，是否走索引见 `data-design.md#7` |

```xml
<!-- Mapper XML：排序字段用 choose 映射到列名，绝不出现 ORDER BY ${sortBy} -->
<choose>
  <when test="sortBy == 'studentNo'">ORDER BY student_no</when>
  <when test="sortBy == 'name'">ORDER BY name</when>
  <when test="sortBy == 'createdAt'">ORDER BY created_at</when>
  <otherwise>ORDER BY id</otherwise>
</choose>
<if test="sortOrder == 'desc'">DESC</if>
LIMIT #{offset}, #{pageSize}
```

- 所有条件值一律 `#{}`；`${}` 只允许出现在**排序方向 / 列名白名单**这种无法参数化的位置，且必须来自第 7 节白名单。
- `offset` 由 Service 计算（`(page - 1) * pageSize`），不传给前端计算。
- 分页必须「一次列表查询 + 一次 count 查询」，count 不带 `ORDER BY`、不做 `JOIN` 以外多余的列。

## 8. 示例：P01 学生模块契约（6 个端点）

本阶段（P01）无认证，权限列统一 `ANONYMOUS`，P06 引入 JWT 后把写操作改为 `ROLE_ADMIN`。

| 方法 | 路径 | 用途 | 请求 DTO | 响应 VO | 成功码 | 主要错误码 | 权限 |
|---|---|---|---|---|---|---|---|
| `GET` | `/api/p01/students` | 列表分页 + 条件筛选 | `StudentQuery` | `PageResult<StudentVO>` | 200 | `COMMON_PARAM_INVALID` | `ANONYMOUS` |
| `GET` | `/api/p01/students/{id}` | 详情 | 路径参数 | `StudentVO` | 200 | `STUDENT_NOT_FOUND` | `ANONYMOUS` |
| `POST` | `/api/p01/students` | 新增 | `StudentCreateDTO` | `Long` | 201 | `COMMON_PARAM_INVALID`、`STUDENT_NO_DUPLICATED` | `ANONYMOUS` |
| `PUT` | `/api/p01/students/{id}` | 修改 | `StudentUpdateDTO` | `Void` | 200 | `STUDENT_NOT_FOUND`、`STUDENT_NO_DUPLICATED` | `ANONYMOUS` |
| `DELETE` | `/api/p01/students/{id}` | 删除（逻辑） | 路径参数 | `Void` | 200 | `STUDENT_NOT_FOUND` | `ANONYMOUS` |
| `GET` | `/api/p01/students/options` | 条件搜索（下拉补全用） | `keyword`、`limit` | `List<StudentOptionVO>` | 200 | `COMMON_PARAM_INVALID` | `ANONYMOUS` |

```json
// GET /api/p01/students?page=1&pageSize=10&classId=12&keyword=张&sortBy=studentNo&sortOrder=asc
{
  "code": "OK",
  "message": "OK",
  "data": {
    "page": 1,
    "pageSize": 10,
    "total": 37,
    "totalPages": 4,
    "list": [
      {
        "id": 1001,
        "studentNo": "20260001",
        "name": "张三",
        "gender": "MALE",
        "classId": 12,
        "className": "软件 2601",
        "enrollYear": 2026,
        "createdAt": "2026-01-05 10:20:30"
      }
    ]
  }
}

// POST /api/p01/students 请求体
{ "studentNo": "20260001", "name": "张三", "gender": "MALE", "classId": 12, "enrollYear": 2026 }

// POST /api/p01/students 成功响应（201）
{ "code": "OK", "message": "新增成功", "data": 1001 }

// GET /api/p01/students/999999 失败响应（404）
{ "code": "STUDENT_NOT_FOUND", "message": "学生不存在或已被删除", "data": null }

// POST 学号重复 失败响应（409）
{ "code": "STUDENT_NO_DUPLICATED", "message": "学号 20260001 已存在", "data": null }
```

P01 错误码对应关系：`STUDENT_NOT_FOUND` → `StudentNotFoundException`；`STUDENT_NO_DUPLICATED` → `StudentNoDuplicatedException`（由 `uk_p01_student_student_no` 唯一索引冲突触发，Service 捕获 `DuplicateKeyException` 后抛出，禁止先查后插做判断）；`COMMON_PARAM_INVALID` → `ParamInvalidException`。

## 9. 与页面状态的一致性自检

| 检查 | 做法 | 不通过的表现 |
|---|---|---|
| 端点有页面用 | 第 3 节每行都能在 `page-state.md#6` 找到引用 | 某个端点无人调用 → 删掉或补页面动作 |
| 页面动作有端点 | `page-state.md#6` 每行都能在第 3 节找到行 | 某动作找不到接口 → 补端点 |
| 字段名一致 | 两侧按 `camelCase` 逐字比对（`studentNo` ≠ `student_no`） | 前端拿到 `undefined` |
| 错误码有处理 | 第 5 节每个错误码在页面状态里有对应界面表现（表单错误 / 顶部提示 / 空态） | 只弹「请求失败」 |

```powershell
# 端点计数（与 page-state.md#6 的动作数对比）
Select-String -Path docs\api-contract.md -Pattern '^\| `(GET|POST|PUT|DELETE)`' | Measure-Object
```

## 填写检查清单

- [ ] 路径全为名词复数 + `/api/p0N` 前缀，没有动词进路径、没有下划线。
- [ ] 端点表「请求 DTO」「响应 VO」列只出现 `dto` / `vo` / `query` 包的类，没有实体。
- [ ] 「权限」列每行有值，`OWNER` 场景写明了归属过滤落在哪一层。
- [ ] `Result<T>` 与 `PageResult<T>` 的字段与本节 JSON 完全一致，没有额外自定义包装。
- [ ] `Result.code` 是业务错误码字符串（成功为 `"OK"`），HTTP 状态与错误码没有混用。
- [ ] 每个非 200 的错误都有错误码行，且对应一个 `exception` 包里的子类。
- [ ] 错误码是大写下划线、模块前缀，没有把 HTTP 状态塞进名字。
- [ ] 校验规则写在 DTO 注解上，Controller 里没有手写 `if (xxx == null)`。
- [ ] 校验失败结构（`List<FieldErrorVO>`）与前端展示方式已对齐。
- [ ] `page` 从 1 开始，`pageSize` 有上限，`sortBy` 有白名单且能拒绝非法值。
- [ ] Mapper XML 中不存在 `ORDER BY ${...}` 直拼；`${}` 的每一次出现都有白名单来源说明。
- [ ] 端点数与 `page-state.md` 的页面动作数相等，字段名逐字比对通过。
- [ ] 契约冻结版本（commit hash）已记录到 `design-completion.md`。
