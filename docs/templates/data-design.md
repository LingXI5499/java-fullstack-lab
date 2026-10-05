# 数据设计（概念 — 逻辑 — 物理）

> **用途**：把需求里的名词变成可建表、可迁移、可验证的库表设计，并说清楚每张表为什么长这样。
> **填写时机**：S1 设计阶段的第二步（`adr.md` 之后、`api-contract.md` 之前）——契约的字段名要对得上表与实体。
> **产出位置**：`projects/P0N-<slug>/docs/data-design.md`；DDL 落到 `database/schema.sql` 与 `database/migrations/`，排查 SQL 落到 `test/`。

## 1. 三层与各自的产出

| 层 | 回答什么 | 产出物 | 评审看什么 |
|---|---|---|---|
| 概念层 | 业务里有哪几个「东西」，它们什么关系 | 实体表 + 关系文字描述（第 3 节） | 有没有漏实体、有没有把关系当字段 |
| 逻辑层 | 每个实体落成哪张表、有哪些列、什么约束与索引 | 字段表（第 4 节）、索引清单（第 7 节） | 约束是否够、关联字段是否都有索引 |
| 物理层 | 用哪条 DDL 建、怎么迁移、种子怎么灌、孤儿怎么查 | `schema.sql` / `migrations/*.sql` / `seed.sql` / `test/*.sql` | 幂等、可回滚、无数据库层外键 |

三层必须能互相指认：概念层每个实体在逻辑层有表；逻辑层每张表在物理层有 DDL；物理层的每条索引在逻辑层有理由。

## 2. 全局基线（固定值，不得逐项目自定）

| 项 | 固定值 | 说明 |
|---|---|---|
| 库 | `java_fullstack_lab` | 十个项目**共用一个库**，不各自建库 |
| 表前缀 | `p0N_` | P01 → `p01_student`，P02 → `p02_book`；本项目 SQL **只允许出现自己的前缀**，禁止跨项目 JOIN（门禁自动扫描） |
| 表名 | 业务单数 | `p01_student` 而非 `p01_students` / `t_student` |
| 字段名 | `snake_case` | 布尔用 `is_` 前缀（`is_deleted`），时间用 `_at` 后缀（`created_at`） |
| 关联字段 | `<被引用表单数>_id` | 如 `p01_student.class_id`、`p02_borrow.book_id`；**必须建普通索引** |
| 约束前缀 | `uk_` / `idx_` / `ck_` | 唯一约束 / 普通索引 / 检查约束；本仓库**没有**数据库层外键，见第 5 节 |
| 字符集与排序规则 | `utf8mb4` / `utf8mb4_general_ci` | 建表显式声明，避免继承服务器默认值 |
| 引擎 | `InnoDB` | 事务与行锁的前提 |
| 时间类型 | `datetime` | 不用 `timestamp`（避免 2038 与隐式时区转换） |
| 迁移脚本 | `database/migrations/V<版本>__<说明>.sql` | 如 `V20260105__add_book_isbn.sql` |

命名规则：`uk_<表>_<业务语义>`、`idx_<表>_<列1>_<列2>`、`ck_<表>_<列>_<规则>`；全小写、下划线分隔、长度不超过 64 字符。

```sql
-- 三种前缀一次看全（P02 示例；关联用普通索引，不用数据库层外键，见第 5 节）
ALTER TABLE p02_borrow
  ADD CONSTRAINT uk_p02_borrow_no     UNIQUE (borrow_no),
  ADD CONSTRAINT ck_p02_borrow_status CHECK (status IN ('BORROWED', 'RETURNED', 'OVERDUE')),
  ADD INDEX        idx_p02_borrow_reader_status (reader_id, status),
  ADD INDEX        idx_p02_borrow_book_id (book_id);
```

## 3. 概念层：实体与关系

实体表模板：

| 实体 | 说明 | 关键属性 | 来源需求 |
|---|---|---|---|
| `<实体名>` | <一句话业务含义> | <业务键 / 状态 / 归属> | `REQ-<模块>-<序号>` |

关系用文字写清楚，每行含基数、含义与删除策略：

```text
班级 1 ── N 学生        含义：学生归属唯一班级；删班级前必须检查学生（CLS）
图书 1 ── N 借阅记录    含义：一条借阅指向唯一图书；删图书前必须检查未归还记录（BOOK）
读者 N ── N 图书        经中间表 p02_borrow；附加属性：borrow_at / due_at / status
```

规则：

- 每个业务名词要么是**实体**（有表）、要么是**属性**（有列）、要么是**枚举值**（有 `ck_` 或 `enumeration` 类）；不允许出现「综合信息表」。
- 关系在物理层一律落成**关联字段**（`<被引用表单数>_id`），不落成数据库层外键；跨项目只存值（如 `reader_no`），不 JOIN 别的项目表。
- 关系上的「数量」若会变化（如借阅次数），放到子表统计，不塞进父表 JSON 字段。

## 4. 逻辑层：字段表模板

复制这张表，一张表填一份：

| 字段名 | 类型 | 允许空 | 默认值 | 说明 | 是否索引 | 关联 |
|---|---|---|---|---|---|---|
| `id` | `bigint unsigned` | 否 | `AUTO_INCREMENT` | 主键 | 主键 | — |
| `class_id` | `bigint unsigned` | 否 | — | 所属班级 | `idx_p01_student_class_id` | `p01_class.id`（逻辑外键） |
| `<字段>` | `<类型>` | 是 / 否 | `<默认值>` | <业务含义与取值来源> | `uk_...` / `idx_...` / 否 | `<表>.<列>` |

填写规则：

- 类型只从这批里选：`bigint unsigned` / `int unsigned` / `tinyint(1)` / `varchar(n)` / `char(n)` / `decimal(10,2)` / `date` / `datetime` / `text`。
- 业务键（学号、ISBN、订单号）必须 `NOT NULL` + `uk_`；金额必须 `decimal`，禁用 `float`。
- 关联字段必须 `NOT NULL` + `idx_`，并在「关联」列写明被引用表与列、在第 5 节登记存在性校验；每张表都带 `is_deleted tinyint(1) NOT NULL DEFAULT 0`、`created_at`、`updated_at`（见第 9 节）。
- 「是否索引」列不允许空着：不建索引要写「否」，并在第 7 节说明为什么不该建。

## 5. 逻辑外键 vs 物理外键（本仓库铁律）

物理外键 = 在 DDL 里声明的数据库层外键约束（含级联删除语法）；逻辑外键 = 只存 `<被引用表单数>_id`，引用完整性由应用层保证。**本流水线一律使用逻辑外键**，`schema.sql` 中不得出现数据库层外键约束与级联删除语法（门禁扫描禁用词）。

| 关注点 | 物理外键（数据库层约束） | 逻辑外键（本仓库采用） |
|---|---|---|
| 引用完整性 | 数据库强制：插入不存在的父 id 直接失败 | 应用层保证：写入前查父记录，不存在返回 `CLASS_NOT_FOUND`；漏写校验就会产生孤儿 |
| 删除父记录 | 交给数据库按 RESTRICT / CASCADE 处理，级联删除隐式且影响面难评估 | 删除前在 Service 显式检查子记录，有引用即返回 `CLASS_HAS_STUDENTS`；删除动作在代码里可见可测 |
| 迁移与批量导入 | 顺序被约束绑死：先导子表即报错，`schema.sql` 必须按依赖排序 | 导入顺序自由；代价是导入后必须跑一次孤儿排查（第 6 节） |
| 表前缀隔离与未来拆分 | 跨项目外键无法建立；将来按项目拆库还要先删约束 | 每个项目只出现自己的 `p0N_` 前缀，十个项目同库共存，垂直拆分零改表结构 |
| 插入与更新性能 | 每次写子表都要回查父表并加锁，高并发写放大 | 无数据库层回查锁；代价是校验与事务范围要自己控制 |
| 孤儿数据风险 | 结构上不可能出现（数据库兜底） | 可能人为产生：漏写校验、直连 SQL 改数据、手工删父行；靠排查 SQL + 用例兜底 |

**三条强制要求**（缺一条，G2 判据 3 不通过）：

1. **所有逻辑外键字段必须建普通索引**：如 `idx_p01_student_class_id (class_id)`；删除前的引用检查、JOIN 查询、孤儿排查都靠它。与状态列常一起用时，可另建联合索引，但单列索引不可省。
2. **删除父记录前必须在 Service 层显式检查引用方并返回业务错误码**：班级下有学生时删除班级必须返回 `CLASS_HAS_STUDENTS`（异常子类 `ClassHasStudentsException`）。留给用户的实现位写成
   `// TODO(user): 删除前用 idx_p01_student_class_id 检查是否存在未删除学生，有则抛 ClassHasStudentsException - TC-CLS-DELETE-02`。
3. **「引用不存在」路径必须有测试用例**：用不存在的 `class_id` 新增学生必须返回 `CLASS_NOT_FOUND`，而不是插入成功（`TC-STU-CREATE-04`）；修改时把 `class_id` 改成不存在的值同样要失败（`TC-STU-UPDATE-03`）。

## 6. 孤儿数据排查

逻辑外键没有数据库兜底，所以每个项目都要有一条**孤儿排查 SQL**，放在 `projects/P0N-<slug>/test/orphan-check.sql`，在 G3 门禁与每次数据修复后执行，**期望结果为 0 行**。

```sql
-- test/orphan-check.sql：学生指向了不存在的（或已删除的）班级 —— 逻辑外键被破坏
SELECT s.id, s.student_no, s.class_id
FROM p01_student s
LEFT JOIN p01_class c
       ON c.id = s.class_id
      AND c.is_deleted = 0
WHERE s.is_deleted = 0
  AND c.id IS NULL;
-- 期望：0 行。>0 行说明写入前未校验引用，或父记录被绕过应用层删除（见 CLS）
```

写这条 SQL 的两个要点：

- 父表的过滤条件必须放在 `ON` 里。写成 `WHERE c.is_deleted = 0` 会把 `LEFT JOIN` 退化成内连接，孤儿记录反而查不出来。
- 反向也要查一次：父表被逻辑删除、子记录仍在用的场景（上例已覆盖）；若项目有「子记录必须在父存在时可写」的批量导入，导入脚本跑完必须立刻执行本 SQL。执行方式 `Get-Content test\orphan-check.sql | mysql -uroot -p java_fullstack_lab`，结果非 0 行即失败。

## 7. 索引设计

**该建索引的字段**

| 场景 | 建法 | 例子 |
|---|---|---|
| 关联字段（逻辑外键） | **必须**建普通索引 | `idx_p02_borrow_book_id (book_id)` |
| 业务唯一键 | `uk_` 唯一约束 | `uk_p01_student_student_no (student_no)` |
| 关联 + 状态过滤 | 联合索引，等值列在前 | `idx_p02_borrow_reader_status (reader_id, status)` |
| 高频等值 + 排序 | 联合索引，等值列在前、排序列在后 | `idx_p01_student_class_name (class_id, name)` |
| 列表页只取少数列 | 覆盖索引（把查询列放进索引，避免回表） | `idx_p01_student_class_no_name (class_id, student_no, name)` |

**不该建索引的字段**

- 低基数列单独建：`is_deleted`、只有两三个值的 `status`（单独建几乎不会被优化器选中，要与其它列联合）。
- 极少出现在 `WHERE` / `ORDER BY` 的长文本（`text`、`varchar(500)`）；确需检索用前缀索引并说明选择性。
- 已被联合索引最左前缀覆盖的单列（如已有 `(reader_id, status)`，再建 `(reader_id)` 是冗余）；写多读少的流水表上的「顺手索引」同样不值得建。

**索引的代价**：每次 `INSERT` / `UPDATE` / `DELETE` 都要维护索引页（写放大）；索引占 buffer pool，挤占数据缓存；候选索引变多，统计信息不准时优化器可能选错；大表加索引是 DDL 成本，必须走第 8 节的迁移三步。

**用 `EXPLAIN` 验证**（每个高频查询都要留一次结论）：

```sql
EXPLAIN
SELECT id, student_no, name, class_id
FROM p01_student
WHERE is_deleted = 0 AND class_id = 12
ORDER BY student_no
LIMIT 0, 10;
```

判据：`type` 至少是 `ref` 或 `range`（**不出现 `ALL`**）；`key` 命中预期索引；`rows` 与预期结果量级一致；列表页 `Extra` 不出现 `Using filesort` / `Using temporary`。反例：`type=ALL` 且 `rows` 接近全表 → 补联合索引，或检查是否写了前置通配（`LIKE '%张'` 无法走索引）。

**深分页取舍**：`LIMIT 100000, 10` 会扫描并丢弃前 10 万行。本仓库约定：`pageSize` 上限 100，超过 200 页的列表改用游标式 `WHERE id > #{lastId} ORDER BY id LIMIT #{pageSize}`。

## 8. 物理层：DDL / 迁移 / 种子

| 文件 | 用途 | 谁执行 | 幂等要求 |
|---|---|---|---|
| `database/schema.sql` | 全量结构，本地与 CI 一键重建 | 开发 / CI | 必须可重复执行（`DROP TABLE IF EXISTS` + `CREATE TABLE`），且**不含数据库层外键约束** |
| `database/seed.sql` | 种子数据，让页面一打开就有内容 | 开发 / CI | 可重复执行（`INSERT ... ON DUPLICATE KEY UPDATE`）；**禁止在生产执行** |
| `database/migrations/V<版本>__<说明>.sql` | 增量变更，含回滚 SQL（同文件末尾注释块或 `_down.sql`） | 开发 / 部署 | 一次变更一个文件，只出现本项目前缀 |
| `test/orphan-check.sql` | 孤儿数据排查（第 6 节） | G3 门禁 / 数据修复后 | 只读，期望 0 行 |

**变更三步：扩展 — 迁移 — 收缩**（禁止一步改到位）：

```sql
-- 第 1 步 扩展：只加可空列，不影响在跑的业务
ALTER TABLE p02_book ADD COLUMN isbn varchar(20) NULL COMMENT 'ISBN' AFTER title;
-- 第 2 步 迁移：回填历史行（分批执行，避免长事务）
UPDATE p02_book SET isbn = CONCAT('UNKNOWN-', id) WHERE isbn IS NULL;
-- 第 3 步 收缩：数据齐了再收紧约束
ALTER TABLE p02_book MODIFY COLUMN isbn varchar(20) NOT NULL COMMENT 'ISBN';
ALTER TABLE p02_book ADD CONSTRAINT uk_p02_book_isbn UNIQUE (isbn);
-- 回滚（与上面一一对应，按逆序执行）
ALTER TABLE p02_book DROP INDEX uk_p02_book_isbn;
ALTER TABLE p02_book DROP COLUMN isbn;
```

## 9. 逻辑删除与时间字段的取舍

| 方案 | 优点 | 代价 | 本仓库结论 |
|---|---|---|---|
| 物理删除 | 表不膨胀，查询无需额外条件 | 数据不可恢复；引用它的子记录立刻变成孤儿；审计断链 | 仅用于种子 / 测试数据清理 |
| 逻辑删除 `is_deleted tinyint(1)` | 可恢复、可追溯；子记录仍指向有效父行；误删可救 | 每个查询都必须带 `is_deleted = 0`，漏写就出脏数据；唯一索引与删除冲突；表持续增长 | **采用**：所有业务表统一带该列，Mapper XML 用 `<sql>` 片段统一带条件 |

唯一索引与逻辑删除的冲突必须显式处理：`uk_p01_student_student_no (student_no)` 会让「删除后再用同一学号」插入失败。两种做法二选一，并在本节写明：① 删除时改写业务键（`student_no = CONCAT(student_no, '#', id)`）；② 唯一键改为 `(student_no, is_deleted)`（代价是同一学号最多删一次）。本仓库默认用 ①。

时间字段统一：`created_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP`、`updated_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`；时区由连接串 `serverTimezone=Asia/Shanghai` 固定（见 `docs/dev-env.md`）。

## 10. 示例：P02 图书管理系统表设计

**概念层**：读者 1 ── N 借阅记录；图书 1 ── N 借阅记录；`p02_borrow` 是「图书 × 读者」的带状态关联表。分类先用 `category_code` + `ck_` 约束（字典表留到需要维护层级时再建）。关联只存 id，不建数据库层外键。

**逻辑层（3 张表，字段压缩为关键列）**

`p02_book`：`id` / `isbn`(`uk_`) / `title` / `author` / `category_code` / `total_copies` / `available_copies` / `is_deleted` / `created_at` / `updated_at`
`p02_reader`：`id` / `reader_no`(`uk_`) / `name` / `status`(`ACTIVE`/`FROZEN`) / `is_deleted` / `created_at` / `updated_at`
`p02_borrow`：`id` / `borrow_no`(`uk_`) / `book_id`(`idx_`) / `reader_id`(`idx_`) / `borrow_at` / `due_at` / `returned_at`(可空) / `status` / `is_deleted` / `created_at` / `updated_at`

**物理层**

```sql
CREATE TABLE IF NOT EXISTS p02_book (
  id               bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  isbn             varchar(20)     NOT NULL COMMENT 'ISBN，业务唯一键',
  title            varchar(120)    NOT NULL COMMENT '书名',
  author           varchar(60)     NOT NULL COMMENT '作者',
  category_code    varchar(20)     NOT NULL COMMENT '分类编码',
  total_copies     int unsigned    NOT NULL DEFAULT 1 COMMENT '馆藏总数',
  available_copies int unsigned    NOT NULL DEFAULT 1 COMMENT '可借数量',
  is_deleted       tinyint(1)      NOT NULL DEFAULT 0 COMMENT '逻辑删除：0 正常 1 已删',
  created_at       datetime        NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  updated_at       datetime        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (id),
  CONSTRAINT uk_p02_book_isbn   UNIQUE (isbn),
  CONSTRAINT ck_p02_book_copies CHECK (available_copies <= total_copies),
  INDEX idx_p02_book_title_author (title, author),
  INDEX idx_p02_book_category_code (category_code)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '图书';
CREATE TABLE IF NOT EXISTS p02_reader (
  id         bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  reader_no  varchar(20)     NOT NULL COMMENT '读者证号，业务唯一键',
  name       varchar(60)     NOT NULL COMMENT '姓名',
  status     varchar(16)     NOT NULL DEFAULT 'ACTIVE' COMMENT 'ACTIVE 正常 / FROZEN 冻结',
  is_deleted tinyint(1)      NOT NULL DEFAULT 0 COMMENT '逻辑删除',
  created_at datetime        NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  updated_at datetime        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (id),
  CONSTRAINT uk_p02_reader_no     UNIQUE (reader_no),
  CONSTRAINT ck_p02_reader_status CHECK (status IN ('ACTIVE', 'FROZEN'))
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '读者';
CREATE TABLE IF NOT EXISTS p02_borrow (
  id          bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  borrow_no   varchar(24)     NOT NULL COMMENT '借阅单号，业务唯一键',
  book_id     bigint unsigned NOT NULL COMMENT '图书主键（逻辑外键 → p02_book.id）',
  reader_id   bigint unsigned NOT NULL COMMENT '读者主键（逻辑外键 → p02_reader.id）',
  borrow_at   datetime        NOT NULL COMMENT '借出时间',
  due_at      datetime        NOT NULL COMMENT '应还时间',
  returned_at datetime        NULL COMMENT '实际归还时间，未还为 NULL',
  status      varchar(16)     NOT NULL DEFAULT 'BORROWED' COMMENT 'BORROWED / RETURNED / OVERDUE',
  is_deleted  tinyint(1)      NOT NULL DEFAULT 0 COMMENT '逻辑删除',
  created_at  datetime        NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  updated_at  datetime        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (id),
  CONSTRAINT uk_p02_borrow_no     UNIQUE (borrow_no),
  CONSTRAINT ck_p02_borrow_status CHECK (status IN ('BORROWED', 'RETURNED', 'OVERDUE')),
  INDEX idx_p02_borrow_reader_status (reader_id, status),
  INDEX idx_p02_borrow_book_id (book_id)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '借阅记录';
```

**索引用途与验证**

| 查询 / 校验 | 走哪条索引 | `EXPLAIN` 期望 |
|---|---|---|
| 某读者当前在借 | `idx_p02_borrow_reader_status` | `type=ref`、`key` 命中该索引、`rows` 个位数 |
| 某书的借阅历史 | `idx_p02_borrow_book_id` | `type=ref`，无 `Using filesort`（排序走 `borrow_at` 时需额外考量） |
| 书名前缀检索 | `idx_p02_book_title_author` 支持 `LIKE 'SQL%'` | `type=range`；若是 `%SQL%` 必然 `ALL`，需在第 7 节写明取舍 |

**应用层要补的三件事**：① 新增借阅前校验 `book_id` / `reader_id` 存在，否则 `BOOK_NOT_FOUND` / `READER_NOT_FOUND`；② 删除图书前用 `idx_p02_borrow_book_id` 检查是否存在 `status = 'BORROWED'` 的记录，有则 `BOOK_HAS_ACTIVE_BORROW`；③ `test/orphan-check.sql` 里加一条对 `p02_borrow.book_id` / `reader_id` 的孤儿排查。同一读者同一本书「未还只能有一条」：MySQL 8 没有部分索引，在 `BorrowServiceImpl` 事务里先查后插，并由 `uk_p02_borrow_no` 兜底防重复提交。

## 填写检查清单

- [ ] 概念层每个实体在逻辑层有表，逻辑层每张表在物理层有 DDL。
- [ ] 表名前缀是**本项目**的 `p0N_`，全文没有出现别的项目前缀，没有跨项目 JOIN。
- [ ] `schema.sql` 与本文档中**没有出现数据库层外键约束与级联删除语法**（门禁扫描的禁用词）。
- [ ] 每个关联字段都是 `<被引用表单数>_id`，且有 `idx_` 普通索引，并在第 5 节登记了存在性校验。
- [ ] 第 5 节的「删除前检查引用」与「引用不存在路径用例」都已落到具体错误码与 `TC-` 编号。
- [ ] `test/orphan-check.sql` 已存在，父表过滤条件写在 `ON` 里，期望结果为 0 行。
- [ ] 每张表有主键，业务键有 `uk_`，枚举列有 `ck_`，布尔用 `is_` 前缀、时间用 `_at` 后缀；字符集与排序规则显式写成 `utf8mb4` / `utf8mb4_general_ci`，引擎为 `InnoDB`。
- [ ] 字段表「是否索引」列没有空值；不建索引的字段在第 7 节说明了理由。
- [ ] 每个高频查询都有索引对应，并留下一次 `EXPLAIN` 结论（含 `type` / `key` / `rows`）。
- [ ] 变更走「扩展—迁移—收缩」三步，回滚 SQL 与正向 SQL 一一对应；`schema.sql` 可重复执行，`seed.sql` 幂等且标注「禁止在生产执行」。
- [ ] 逻辑删除与唯一索引的冲突已选定处理方式（改写业务键 or 唯一键含 `is_deleted`）。
- [ ] 金额用 `decimal`，没有 `float` / `double`；没有用 `timestamp`。
