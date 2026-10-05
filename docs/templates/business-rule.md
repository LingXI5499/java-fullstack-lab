# 业务规则卡

> **用途**：把散落在需求描述里的约束拆成一条条可独立测试的规则，让 Service 实现、异常定义与测试断言有共同依据。
> **填写时机**：`S0 立项与规格`，每张需求卡写完立即补它的规则卡；`G1` 闸门前全部规则必须填完。
> **产出位置**：`projects/P01-student-info/docs/business-rules.md` 的「规则清单」一节，按 `BR-STU-*` 顺序排列。

## 1. 卡片骨架

一条规则一张卡，直接复制下面整块。

### BR-\<模块\>-\<序号\> \<规则名称\>

| 字段 | 内容 |
|---|---|
| 规则编号 | `BR-<模块>-<序号>`，例 `STU`；一经分配不得复用 |
| 规则名称 | <能当句子的短标题，例：学号全局唯一> |
| 规则分类 | 数据校验 / 状态流转 / 权限 / 计算 / 并发 / 外部依赖 |
| 规则陈述 | 当 <条件> 且 <条件> 时，系统必须 <可观察的动作> |
| 反面情况 | <违反时系统做什么：拒绝并返回什么错误码；还是修正、回滚、降级> |
| 触发时机 | <在哪个用例、哪一步执行：Controller 入参校验 / Service 事务内 / 提交前> |
| 可测试性说明 | <一句话说明如何测出它被违反，例：提交两次相同学号，第二次必须返回 409 且表内仍为 1 行> |
| 关联需求编号 | `STU` |

规则陈述必须用「当…且…时，系统必须…」句式。写不出这个句式的，多半是设计决策而不是业务规则，应移到 ADR。

## 2. 规则分类与落点

| 分类 | 典型形态 | 代码落点 | 失败信号 |
|---|---|---|---|
| 数据校验 | 唯一、必填、长度、枚举、区间 | DTO 上的 `@Valid` + Service 内的唯一性查询 | 400 / 409 |
| 状态流转 | 只允许特定状态间迁移 | Service 内的状态判断 + `enumeration` 状态枚举 | 409 |
| 权限 | 角色能做什么、数据可见范围 | `interceptor` / `aspect` + Service 归属校验 | 401 / 403 |
| 计算 | 金额、分值、统计口径与舍入 | Service 内独立方法，禁止写在 Controller | 结果比对失败 |
| 并发 | 同一资源被同时修改、库存扣减 | 乐观锁 `version` 或 `WHERE` 条件更新 + 影响行数判断 | 409 |
| 外部依赖 | 第三方超时、重试、降级 | `storage` 实现内的超时与降级分支 | 504 或降级态 |

## 3. 示例一：数据校验规则

### STU 学号全局唯一

| 字段 | 内容 |
|---|---|
| 规则编号 | `STU` |
| 规则名称 | 学号全局唯一 |
| 规则分类 | 数据校验 |
| 规则陈述 | 当新增或修改学生档案且提交了 `student_no` 时，系统必须先按 `student_no` 查询 `p01_student`，若已存在同号记录则拒绝写入，并返回错误码 `STUDENT_NO_DUPLICATED` |
| 反面情况 | 返回 HTTP 409，`Result.code=STUDENT_NO_DUPLICATED`、`Result.message=学号已存在`；事务不产生任何行；表内记录数不变 |
| 触发时机 | Service 事务内、`insert` 之前；修改场景需排除自身主键 |
| 可测试性说明 | 用相同 `student_no` 连续提交两次，第二次必须返回 409 且 `select count(*) from p01_student where student_no=?` 仍为 1 |
| 关联需求编号 | `STU` |

落点与实现约束：

- 数据库层兜底：`uk_p01_student_student_no` 唯一约束，防止并发下的检查-写入竞态；
- 应用层给友好错误：`StudentNoDuplicatedException extends BusinessException`，禁止在 Controller 里 `try/catch`；
- 两个都保留：唯一约束保证正确性，异常类保证可读性。

### 引用完整性（逻辑外键）

本项目不使用物理外键，引用完整性由应用层承担，这类规则要在卡里写清四点。

| 要求 | 规则卡里的写法 | 对应检查 |
|---|---|---|
| 写入前校验 | 当提交 `class_id` 时，系统必须先在 `p01_class` 中查到该 id，查不到则拒绝并返回 `CLASS_NOT_FOUND` | `STU-04` |
| 删除父记录前校验 | 当删除班级时，系统必须先统计 `p01_student` 中引用该班级的行数，大于 0 则返回业务错误码，不做级联删除 | `STU` |
| 关联字段必须有索引 | `class_id` 必须建普通索引 `idx_p01_student_class_id`，保证校验与关联查询不扫全表 | S1 设计评审 + `EXPLAIN` |
| 孤儿数据可排查 | 每个项目在 `test/` 提供孤儿数据 SQL，例：`select s.id from p01_student s left join p01_class c on s.class_id = c.id where c.id is null` | S3 验收前执行一次，结果必须为空 |

`fk_` 前缀在本仓库不使用：约束只用 `uk_` / `idx_` / `ck_`，跨项目 JOIN 同样禁止。

## 4. 示例二：状态流转规则

### STU 学生学籍状态只允许按状态机迁移

| 字段 | 内容 |
|---|---|
| 规则编号 | `STU` |
| 规则名称 | 学籍状态按状态机迁移 |
| 规则分类 | 状态流转 |
| 规则陈述 | 当请求变更学生学籍状态时，系统必须按状态机判断 `当前状态 --事件--> 目标状态` 是否为合法边，非法边一律拒绝并返回 `STUDENT_STATE_INVALID` |
| 反面情况 | 返回 HTTP 409，`Result.code=STUDENT_STATE_INVALID`；`p01_student.status` 保持原值；不写入状态变更日志 |
| 触发时机 | Service 事务内、`update` 之前；状态值来自 `enumeration/StudentStatus` |
| 可测试性说明 | 对一条 `status=SUSPENDED` 的记录提交「毕业」事件，必须返回 409，且再次查询状态仍为 `SUSPENDED` |
| 关联需求编号 | `STU` |

状态机表示：

```text
ENROLLED  --SUSPEND-->  SUSPENDED   // 在读 -> 休学
SUSPENDED --RESUME-->   ENROLLED    // 休学 -> 在读
ENROLLED  --GRADUATE--> GRADUATED   // 在读 -> 毕业
ENROLLED  --WITHDRAW--> WITHDRAWN   // 在读 -> 退学
SUSPENDED --WITHDRAW--> WITHDRAWN   // 休学 -> 退学
GRADUATED --*-->        (终态，任何事件都拒绝)
WITHDRAWN --*-->        (终态，任何事件都拒绝)
```

状态枚举与迁移表必须成对出现，`enumeration/StudentStatus` 里用 `Set` 声明每个状态的合法后继，
Service 只查表不写 `if-else` 链：

```java
// 合法迁移表：key 为当前状态，value 为该状态可到达的目标状态集合
private static final Map<StudentStatus, Set<StudentStatus>> ALLOWED = Map.of(
        StudentStatus.ENROLLED, Set.of(StudentStatus.SUSPENDED, StudentStatus.GRADUATED, StudentStatus.WITHDRAWN),
        StudentStatus.SUSPENDED, Set.of(StudentStatus.ENROLLED, StudentStatus.WITHDRAWN),
        StudentStatus.GRADUATED, Set.of(),
        StudentStatus.WITHDRAWN, Set.of()
);
```

## 5. 示例三：并发规则

### STU 同一学生档案的并发修改必须拒绝后到者

| 字段 | 内容 |
|---|---|
| 规则编号 | `STU` |
| 规则名称 | 学生档案乐观锁更新 |
| 规则分类 | 并发 |
| 规则陈述 | 当两个请求基于同一版本号修改同一条学生记录时，系统必须只允许先提交者成功，后提交者因版本不匹配被拒绝并返回 `STUDENT_CONCURRENT_MODIFIED` |
| 反面情况 | 返回 HTTP 409，`Result.code=STUDENT_CONCURRENT_MODIFIED`；不覆盖先提交者的字段；前端提示「数据已被他人修改，请刷新后重试」 |
| 触发时机 | Service 事务内执行 `update ... where id=? and version=?`，随后判断影响行数 |
| 可测试性说明 | 用 `version=1` 提交两次修改，第二次必须返回 409，且表内 `name` 仍为第一次的值、`version` 为 2 |
| 关联需求编号 | `STU` |

实现约束：

```xml
<!-- StudentMapper.xml：乐观锁更新，返回影响行数 -->
<update id="updateWithVersion">
    update p01_student
       set name = #{name}, phone = #{phone}, version = version + 1, updated_at = now()
     where id = #{id} and version = #{version}
</update>
```

- 表结构需要 `version int not null default 0` 与 `updated_at datetime`；
- Service 必须判断 `int rows == 0` 再抛异常，**不得**用「先查后改」代替版本判定；
- 与 `STU` 叠加时，先判状态再判版本，两个错误码互不覆盖。

## 6. 规则的可测试性自检

写完每条规则，用下面 4 问过一遍，任何一问答不上就重写：

| 自检问题 | 通过标准 |
|---|---|
| 一句话能测出违反吗 | 能写出「操作 X，必须得到结果 Y」 |
| 反面情况有确定结果吗 | 有明确 HTTP 状态码与错误码，不是「提示错误」 |
| 规则是原子动作吗 | 一条规则只约束一件事；两件事拆成两条编号 |
| 规则有落点吗 | 能指出哪个类、哪个约束、哪个枚举承载它 |

## 填写检查清单

- [ ] 编号符合 `BR-<模块>-<序号>`，模块前缀与需求卡一致
- [ ] 规则分类是六类之一，没有「其他」
- [ ] 规则陈述是完整的「当…且…时，系统必须…」句式，主语是系统
- [ ] 反面情况写明 HTTP 状态码与业务错误码，以及数据是否发生变化
- [ ] 触发时机指向具体阶段（入参校验 / 事务内 / 提交前）
- [ ] 可测试性说明里的操作与结果可以直接抄成测试用例步骤
- [ ] 状态流转类规则给出了完整状态机与终态处理
- [ ] 并发类规则写明了判定字段（`version` 或条件 `WHERE`）与影响行数处理
- [ ] 每条规则至少关联一个 `REQ-*` 编号，且该需求卡反向引用了本规则
- [ ] 数据库层有对应约束（`uk_` / `ck_`，关联字段建 `idx_` 普通索引），或明确说明为何不需要
