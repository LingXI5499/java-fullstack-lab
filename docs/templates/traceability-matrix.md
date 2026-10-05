# 需求追踪矩阵

> **用途**：用一张表回答「每条需求落到哪个接口、哪张表、哪个页面、哪条测试」，并暴露没有来源的代码与没有测试的需求。
> **填写时机**：`S0` 建表登记需求编号，`S1` 补 API / 数据表 / 前端页面列，`S2` 补测试用例列，`G4` 复盘前必须无空列。
> **产出位置**：`projects/P01-student-info/docs/traceability.md`（每项目一份，随阶段增量维护）。

## 1. 主表骨架

一个项目一张主表，一行一条需求。复制下面表头即可开始登记。

| 需求编号 | 需求标题 | 业务规则编号 | API 端点 | 数据表 | 前端页面 | 测试用例编号 | 验收条件编号 | 状态 |
|---|---|---|---|---|---|---|---|---|
| `REQ-<模块>-<序号>` | <标题> | `BR-<模块>-<序号>` | `<METHOD> /api/p0N/<资源>` | `p0N_<表>` | `<路由 + 组件>` | `TC-<模块>-<场景>-<序号>` | `AC-<模块>-<序号>-<序号>` | DRAFT / READY / IMPLEMENTED / VERIFIED |

列填写规则：

| 列 | 规则 |
|---|---|
| 业务规则编号 | 多条用 `、` 分隔；确无规则时写「无」，不允许留空 |
| API 端点 | 与 SpringDoc 契约中的 `path` 完全一致；多条用 `<br>` 分隔 |
| 数据表 | 只写本项目前缀的表（`p01_*`），跨项目引用视为违规 |
| 前端页面 | 写路由路径 + `.vue` 文件名，例 `/student/list` → `StudentList.vue` |
| 测试用例编号 | 分层测试中真实存在的用例名，S2 结束前不得为空 |
| 验收条件编号 | 与 `acceptance-criteria.md` 中编号一致，P0 需求每类场景至少 1 条 |
| 状态 | 取该需求全部关联项中最低的状态 |

## 2. 主表示例（P01 学生信息管理系统）

| 需求编号 | 需求标题 | 业务规则编号 | API 端点 | 数据表 | 前端页面 | 测试用例编号 | 验收条件编号 | 状态 |
|---|---|---|---|---|---|---|---|---|
| STU | 新增学生档案 | STU、STU、STU | POST /api/p01/students | p01_student | /student/list → StudentList.vue | TC-STU-CREATE-01、TC-STU-CREATE-02、TC-STU-CREATE-03 | STU-01、STU-02、STU-03、STU-04 | READY |
| STU | 条件分页查询学生 | STU | GET /api/p01/students | p01_student、p01_class | /student/list → StudentList.vue | TC-STU-QUERY-01、TC-STU-QUERY-02 | STU-01、STU-02 | READY |
| STU | 修改学生档案 | STU、STU | PUT /api/p01/students/{id} | p01_student | /student/list → StudentEditDialog.vue | TC-STU-UPDATE-01、TC-STU-UPDATE-03 | STU-01 | READY |
| STU | 接口鉴权与角色控制 | STU | 全局拦截器 | — | — | TC-STU-AUTH-01、TC-STU-AUTH-02 | STU-01、STU-02 | READY |
| STU | 学籍状态变更 | STU | PATCH /api/p01/students/{id}/status | p01_student | /student/detail → StudentStatusPanel.vue | TC-STU-STATUS-01 | STU-01 | READY |
| STU | 删除学生档案 | STU | DELETE /api/p01/students/{id} | p01_student | /student/list → StudentList.vue | TC-STU-DELETE-01 | STU-01 | READY |

表中「—」只允许出现在基础设施类需求（如鉴权拦截器）确实没有独立表或页面的单元格，
其余情况下留「—」必须在同行的备注中说明原因。

## 3. 反向索引

主表按需求看，出问题时按代码看。反向索引按需生成，不必手工维护全量。

| 反查目标 | 生成方式 | 用途 |
|---|---|---|
| 按 API 端点 | 从 SpringDoc 导出 `openapi.json`，按 `path` 分组 | 发现契约里有、主表里没有的端点 |
| 按数据表 | `select table_name from information_schema.tables where table_schema='java_fullstack_lab'` | 发现建了却没被需求引用的表 |
| 按前端页面 | 扫描 `src/views/**/*.vue` 的路由注册 | 发现入口页面上没有的孤儿页面 |
| 按错误码 | 扫描 `exception/` 包中的 `code` 常量 | 发现没有需求来源的错误分支 |

反向索引的产出放进同一个文件，用「反查目标 → 命中项 → 对应需求编号」三列表示，
命中项填不出需求编号的，就是下一节的孤儿。

## 4. 孤儿检查

三类孤儿必须在 `G4` 前清零，检查手段优先用命令，其次用脚本。

### 4.1 没有需求来源的代码

```bash
# 代码里主动标注需求编号，便于反查；命中为空的行需要解释
grep -rn "REQ-" src/main/java/

# Controller 中每个映射端点都应有主表登记
grep -rn "@GetMapping\|@PostMapping\|@PutMapping\|@PatchMapping\|@DeleteMapping" src/main/java/

# 建表脚本中的表都要被需求引用
grep -rn "create table" database/migrations/
```

判定：某端点或某张表在整个仓库里搜不到对应的 `REQ-*`，即为孤儿代码。
处理方式二选一——补一条需求卡，或删除该代码，不允许「先留着」。

### 4.2 没有测试的需求

```bash
# 需求编号应出现在测试方法名或测试注释里
grep -rn "REQ-STU-" src/test/java/

# 每个 REQ 至少要能匹配到一条 TC
for id in $(grep -o "REQ-STU-[0-9]*" projects/P01-student-info/docs/requirements.md | sort -u); do
  grep -rq "$id" src/test/java/ || echo "untested: $id"
done
```

判定：主表中「测试用例编号」为空，或编号在 `src/test/` 中搜不到实现，即为无测试需求。

### 4.3 没有验收条件的业务规则

```bash
# 每条 BR 应至少被一条验收条件或测试用例引用
grep -rn "BR-STU-" projects/P01-student-info/docs/business-rules.md
grep -rn "BR-STU-" projects/P01-student-info/docs/acceptance-criteria.md
```

判定：`business-rules.md` 中出现、但 `acceptance-criteria.md` 与 `src/test/` 中均未出现的 `BR-*`，
说明这条规则无人验证；要么补验收条件，要么删除该规则（规则不落地就是文档噪音）。

### 4.4 CI 脚本思路

把上面三步固化成 `tools/check-traceability.ps1`，接入 `lab.ps1 verify`：

1. 读取 `requirements.md`、`business-rules.md`、`acceptance-criteria.md`，用正则抽出全部 `REQ-` / `BR-` / `AC-` / `TC-` 编号；
2. 校验编号唯一性与格式（`^REQ-[A-Z]+-\d{3}$` 等），发现重号或跳格式立即失败；
3. 校验交叉引用闭合：每个 `REQ` 至少 1 条 `BR` 与 1 条 `AC`；每个 `BR` 至少 1 条 `AC` 或 `TC`；每个 `AC` 至少 1 条 `TC`（P0 需求强制）；
4. 扫描 `src/main/java` 与 `database/migrations`，把未被任何 `REQ` 引用的端点和表列为孤儿；
5. 任一项不通过则退出码非 0，并打印「缺什么 + 在哪一行」。

## 5. 状态推进与门禁挂钩

| 阶段 | 矩阵要求 | 不通过的后果 |
|---|---|---|
| G1 | 需求 / 业务规则 / 验收条件 / NFR 编号齐全，交叉引用闭合 | 不得进入 S1 设计 |
| G2 | API 端点、数据表、前端页面列填满 | 不得进入 S2 实现 |
| G3 | 测试用例列填满，且用例在 `src/test/` 中真实存在 | 不得进入 S3 验收 |
| G4 | 状态列全部为 `VERIFIED`，孤儿检查三项为零 | 不得打 tag、不得归档作品集 |

门禁执行顺序：`lab.ps1 verify`（编译 → 测试 → 前端构建 → 秘密扫描 → 追踪矩阵检查），
矩阵检查失败时输出的孤儿清单直接贴进复盘文档，作为下一轮技能回写的输入。

## 填写检查清单

- [ ] 主表每行的需求编号都能在 `requirements.md` 中找到同号卡片
- [ ] 「业务规则编号」「API 端点」「数据表」「前端页面」在 `G2` 前已填满，无空单元格
- [ ] 「测试用例编号」在 `G3` 前已填满，且编号在 `src/test/` 中可搜到
- [ ] `p01_*` 之外的表前缀没有出现在「数据表」列（跨项目引用是违规）
- [ ] 反向索引中每个 API 端点、数据表、页面都能指回一条需求
- [ ] 孤儿检查三条命令跑过一遍，输出为空或每条命中都有处理结论
- [ ] CI 的追踪矩阵检查已接入 `lab.ps1 verify`，失败会阻断门禁
- [ ] 状态列按最低状态取值，`G4` 前全部为 `VERIFIED`
- [ ] 矩阵变更与大阶段推进同步提交，历史可从 git log 回溯
