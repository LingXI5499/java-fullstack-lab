# jfsl-conventions —— Java 全栈流水线工程约定

> 版本：1.0.0
> 负责工序：**全部**（任何写代码、建表、定接口之前先读本技能）
> 与 `AGENTS.md` 保持一致；两者冲突时以 `AGENTS.md` 为准，并立即修正本技能。

## 适用场景

- 在 java-fullstack-lab 仓库里新建或修改任何 Java / SQL / Vue 代码之前；
- 评审别人的实现是否符合本项目规范时；
- 需要判定"某个写法算不算违规"时。

不适用：与本仓库无关的其他 Java 项目（本技能包含大量本项目特有约定，不要外推到别处）。

---

## 1. 技术基线（不得串版本）

| 项 | 值 | 说明 |
|---|---|---|
| JDK | 编译目标 **17** | `maven.compiler.release=17`，本机 JDK 21 编译；产物 major version 必须是 61 |
| Spring Boot | 3.2.x | 由父 POM 统一 |
| ORM | MyBatis + **手写 Mapper XML** | **禁止引入 MyBatis-Plus** |
| 数据库 | MySQL 8.0 | 单库 `java_fullstack_lab`，`utf8mb4` / `utf8mb4_general_ci` |
| 前端 | Vue 3 + Vite + TypeScript + Pinia + Element Plus + ECharts | pnpm workspace |
| 测试 | JUnit 5 + Mockito + MockMvc | 集成测试用**真实 MySQL**，不用 H2 代替关键 SQL 验证 |
| 文档 | SpringDoc OpenAPI | 契约先行 |
| 部署 | Nginx + systemd | Docker Compose 只是可选交付物 |

**推迟引入**：OSS / `storage/` 抽象（P06 再说）、域名与 HTTPS（真实部署再说）、
Redis / MQ / 微服务 / K8s（流水线下一版）。

## 2. 硬性编码约定（违反即返工）

### 2.1 禁止 Java `record`

VO / DTO / Query 一律 Lombok POJO：

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class StudentVO {
    private Long studentId;
    private String studentNo;
    private String name;
}
```

实体只用 `@Data`（MyBatis 需要可变对象），不额外加 `@Builder`。

理由：`record` 的访问器是 `studentId()` 而不是 JavaBean 的 `getStudentId()`，
与 `BeanUtils.copyProperties`、序列化框架、模板引擎等按 JavaBean 约定工作的库不兼容。

### 2.2 业务异常用继承体系，每个错误一个子类

```java
// exception/ 包
public class StudentNoDuplicatedException extends BusinessException {
    public StudentNoDuplicatedException() {
        super("STUDENT_NO_DUPLICATED", "学号已存在", 409);
    }
}
```

**禁止**"集中式静态工厂 + 通用 `ApiException`"。错误码必须来自项目 `requirements.md` 的错误码清单。

### 2.3 包结构（`star-rain` 分层）

```text
com.jfsl.<project>.<module>/
├─ controller/      HTTP 接口
├─ service/         业务接口
├─ service/impl/    业务实现
├─ mapper/          数据访问接口
├─ entity/          数据库实体
├─ dto/             请求对象
├─ vo/              响应对象
├─ query/           查询条件对象
├─ config/          @Configuration
├─ constant/        常量
├─ enumeration/     枚举
├─ exception/       自定义异常
├─ handler/         全局异常处理器
├─ interceptor/     拦截器
├─ aspect/          切面
├─ utils/           无状态工具类
└─ storage/ + storage/impl/   基础设施抽象（需要时）
```

规则：

- **一个包只放一类东西**；禁止 `support`、`misc`、模块内 `common` 这类无语义包名；
- **接口与实现分离**：`service/` + `service/impl/`；没有接口时不要为了对称硬造接口；
- `@ConfigurationProperties` 放 `properties/`，`@Configuration` 放 `config/`；
- 跨模块共享的（`Result`、`PageResult`、`BusinessException`、`GlobalExceptionHandler`）放在项目根的
  `common/` 包下的对应子包中（`common/result/`、`common/exception/`、`common/handler/`）。

### 2.4 Controller 只做四件事

参数绑定、`@Valid` 校验、调 Service、返回 `Result<T>`。

不允许：写 SQL、写业务判断、直接操作数据库、逐个 `try/catch`、拼装跨模块的复杂 VO。

### 2.5 注释风格

用 `//` 与 `/* */`，**不写 Javadoc 文档注释**（用户明确要求）。

`// TODO(user): <要做什么> - <验收条件编号>` 用于标记留给用户实现的位置。

### 2.6 分层越界检查（提交前自查）

- Controller 里有没有 `if` 判断业务状态？
- Service 里有没有直接写 SQL 字符串？
- Mapper 里有没有业务规则（如"库存不足则返回 false"）？
- 实体类有没有直接被用作修改接口的入参？
- Service 有没有返回实体给 Controller 当响应？

以上任一为"有"，就是分层越界。

## 3. 命名与编号

### 3.1 标识符

| 类型 | 格式 | 示例 |
|---|---|---|
| 需求 | `REQ-<模块>-<NN>` | `REQ-{0}-0{1}` |
| 业务规则 | `BR-<模块>-<NN>` | `BR-{0}-0{1}` |
| 验收条件 | `AC-<模块>-<NN>` | `AC-{0}-0{1}` |
| 测试用例 | `TC-<模块>-<场景>-<NN>` | `TC-{0}-{1}-0{2}` |
| 非功能需求 | `NFR-<分类>-<NN>` | `NFR-{0}-0{1}` |
| 开发任务 | `TASK-<模块>-<NN>` | `TASK-{0}-0{1}` |
| 架构决策 | `ADR-<NNN>` | `ADR-001` |

**模块段与场景段全大写，序号两位补零**（`ADR` 三位）。模块段必须来自项目 `requirements.md` 第 0 节的模块字典。

### 3.2 错误码

格式 `<领域>_<条件>`，领域用**完整单词**的全大写（`STUDENT`、`CLASS`、`MAJOR`），不用缩写。
例：`STUDENT_NO_DUPLICATED`、`CLASS_HAS_STUDENTS`、`PARAM_INVALID`、`SYSTEM_ERROR`。

注意三者书写形式不同，不要互相替换：

| 形式 | 写法 | 例 |
|---|---|---|
| 表名 | snake_case 带项目前缀 | `p01_student` |
| 编号模块段 | 三字母全大写 | `STU` |
| 错误码领域段 | 完整单词全大写 | `STUDENT` |

### 3.3 数据库对象

| 对象 | 规则 | 示例 |
|---|---|---|
| 表 | `p<两位项目号>_<业务单数>` | `p01_student` |
| 主键 | `<表名后缀>_id`，`bigint unsigned auto_increment` | `student_id` |
| 逻辑外键字段 | `<被引用表单数>_id`，**必须建索引** | `class_id` |
| 唯一约束 | `uk_<表>_<字段>` | `uk_p01_student_student_no` |
| 普通索引 | `idx_<表>_<字段列表>` | `idx_p01_student_class_id` |
| 检查约束 | `ck_<表>_<条件>` | `ck_p01_student_age_range` |
| 字段 | `snake_case`；布尔 `is_` 前缀；时间 `_at` 后缀 | `is_deleted`、`created_at` |

### 3.4 Java 命名

- 类 `UpperCamelCase`；方法/字段 `lowerCamelCase`；常量 `UPPER_SNAKE_CASE`；
- 布尔字段不加 `is` 前缀（用 `active` 而非 `isActive`），避免 Lombok 生成的 `isActive()` 与 Jackson 序列化歧义；
- 枚举类名 `XxxStatus` / `XxxType`，枚举值全大写。

## 4. 数据库铁律

### 4.1 单库 + 前缀隔离

全仓库只有一个库 `java_fullstack_lab`，十个项目共用，**不各自建库**。
每项目表以 `p0N_` 为前缀。**一个项目的 SQL 只允许出现自己的前缀**，禁止跨项目 JOIN。

### 4.2 禁止物理外键，只用逻辑外键

- `schema.sql` 不得出现 `FOREIGN KEY` / `REFERENCES` / `ON DELETE CASCADE`；
- 关联用 `<被引用表单数>_id` + 索引表达，引用完整性由 Service 层校验；
- 三条强制要求：
  1. 所有逻辑外键字段必须建索引；
  2. **删除父记录前必须显式检查引用方**并返回专属错误码（如 `CLASS_HAS_STUDENTS`）；
  3. **"引用不存在"路径必须有测试用例**（用不存在的 `class_id` 新增学生必须返回 `CLASS_NOT_FOUND`）。

### 4.3 迁移

- `database/schema.sql`（结构）+ `database/seed.sql`（种子，必须幂等）；
- 增量迁移 `database/migrations/V<版本>__<说明>.sql`；
- 变更走「扩展—迁移—收缩」三步：加字段 → 新代码兼容旧数据 → 回填 → 切换读写 → 清理；
- 每个项目在 `test/` 提供一条孤儿数据排查 SQL。

### 4.4 连接与秘密

```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:mysql://localhost:3306/java_fullstack_lab?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true}
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:}
```

真实凭据只写在已 gitignore 的 `application-local.yml`；仓库内只保留 `application-local.yml.example`。

## 5. 统一契约

### 5.1 响应

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Result<T> {
    private Integer code;      // 0 = 成功，非 0 = 业务错误（用字符串错误码更利于排查，见下）
    private String message;
    private T data;
}
```

本项目约定：`code` 字段承载**字符串错误码**（如 `STUDENT_NOT_FOUND`），成功时为空或 `OK`；
HTTP 状态码按语义设置（200 / 400 / 404 / 409 / 500），不一律返回 200。

分页统一为：

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class PageResult<T> {
    private Long total;
    private Integer page;
    private Integer pageSize;
    private Integer pages;
    private List<T> list;
}
```

### 5.2 分页与排序

- `page` 从 1 开始；`pageSize` 1–100，越界返回 `PARAM_INVALID`，不静默纠正；
- 页码超出总页数返回空列表 + 真实 `total`，不报错；
- 排序字段走**白名单**，禁止把用户输入直接拼进 `ORDER BY`；
- 模糊查询关键词必须转义 `%` 与 `_`。

### 5.3 SQL 参数化

- 一律 `#{}`；**禁止**用 `${}` 拼接用户输入；
- 动态条件用 `<if>` + `<where>`；
- 列表查询必须显式写列名，不用 `SELECT *`。

## 6. 门禁与提交

- 主干 `main`，阶段分支 `feat/P0N-<slug>`，合并用 `--no-ff`；
- Conventional Commits + 项目作用域：`feat(p03): add sale transaction with stock rollback`；
- 一次提交一个目标，且必须可构建；
- 门禁命令：`.\tools\lab.ps1 verify --project P0N`（编译 + 测试 + 前端构建 + 秘密扫描 + 前缀扫描 + 字节码版本）；
- 里程碑：带注释 tag `p0N-vX.Y.Z`，推送后不移动。

## 7. 自检清单

写代码或评审前逐条过：

- [ ] 没有 `record`；VO/DTO 是 Lombok POJO；实体只有 `@Data`
- [ ] 业务异常有专属子类，错误码在本项目错误码清单里
- [ ] 包结构符合 `star-rain` 分层，没有 `support` / `misc` / 模块内 `common`
- [ ] Controller 只做四件事，没有业务判断与 SQL
- [ ] 注释用 `//` / `/* */`，没有 Javadoc
- [ ] 标识符大小写与补零正确，模块段来自模块字典
- [ ] SQL 只用 `#{}`；动态条件用 `<if>` / `<where>`；列表不 `SELECT *`
- [ ] 表名带 `p0N_` 前缀；没有物理外键；逻辑外键字段有索引
- [ ] 删除父记录前检查了引用方，并返回专属错误码
- [ ] 分页参数越界、页码超范围、排序白名单、模糊转义都处理了
- [ ] 没有把真实密码 / 密钥写进被跟踪的文件
- [ ] 改动之后跑过门禁命令
