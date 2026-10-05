# 页面与状态

> **用途**：把每个页面「长什么样、有哪些状态、什么条件下切到哪个状态、调用哪个接口」写清楚，让前后端在 G2 就能对齐，而不是联调时才发现少接口或多接口。
> **填写时机**：S1 与 `api-contract.md` 并行互校；G2 前必须**闭合**（第 6 节的对应表两侧数量相等、字段名逐字一致）。
> **产出位置**：`projects/P0N-<slug>/docs/page-state.md`；前端代码落到 `frontend/src/views/`、`frontend/src/stores/`、`frontend/src/api/`。

## 1. 页面清单

| 页面名 | 路由 | 角色可见性 | 依赖接口 | 主要状态 |
|---|---|---|---|---|
| `<学生列表>` | `/students` | `ANONYMOUS` | `GET /api/v1/students`、`DELETE /api/v1/students/{id}` | `loading / loaded / empty / error / submitting` |
| `<学生新增弹窗>` | 同页弹窗 | `ANONYMOUS` | `POST /api/v1/students`、`GET /api/v1/students/options` | `submitting` |

填写规则：

- 路由全小写、kebab-case（`/students`、`/borrow-records`）；详情页带参数（`/students/:id`）。
- 角色可见性取值与 `api-contract.md` 权限列一致（`ANONYMOUS` / `ROLE_ADMIN` / `OWNER`）；**隐藏菜单或按钮不算权限**，写操作仍由后端校验。
- 依赖接口写全「方法 + 路径」，且必须能在 `api-contract.md#3` 端点表找到同名行。
- 弹窗、抽屉、Tab 若不共享数据，可作为一个页面块登记；共享同一个列表数据时不要重复登记。

## 2. 列表页状态机

| 状态 | 触发 | 界面表现 | 退出条件 |
|---|---|---|---|
| `loading` | 首屏进入 / 翻页 / 改条件 / 点重试 | 表格骨架屏或 loading 遮罩；禁用分页器与搜索按钮 | 响应返回（成功或失败） |
| `loaded` | 响应成功且 `list.length > 0` | 渲染表格 + 分页器（`total` 用后端值）；顶部显示"共 N 条" | 改条件 / 翻页 / 删除 / 重新查询 |
| `empty` | 响应成功且 `list.length === 0` | 表格空态（`el-empty`）；**区分两种空**：库里没数据 vs 条件无匹配 | 改条件 / 清空条件 / 新增成功 |
| `error` | 请求失败（网络 / 5xx / 业务错误码） | 表格区域显示错误提示 + 「重试」按钮；保留顶部搜索条件 | 点重试 → `loading`；改条件 → `loading` |
| `submitting` | 行内删除确认后 / 弹窗提交中 | 该行按钮 loading、整表禁用；弹窗按钮 loading 且禁止关闭 | 成功 → `loading` 并重新查询；失败 → 回 `loaded` 并提示 |

```text
loading ──成功且非空──> loaded ──删除本页最后一条──> loading（页码回退）
loading ──成功且为空──> empty
loading ──失败──> error ──重试──> loading
loaded ──提交──> submitting ──成功──> loading
submitting ──失败──> loaded（保留原数据，只提示错误）
```

用三个变量派生状态，不要手写五个 `ref` 互相同步：

```ts
type ListState = 'loading' | 'loaded' | 'empty' | 'error'

const loading = ref(false)
const error = ref<string | null>(null)
const rows = ref<StudentVO[]>([])

const state = computed<ListState>(() => {
  if (loading.value) return 'loading'
  if (error.value) return 'error'
  return rows.value.length > 0 ? 'loaded' : 'empty'
})
```

规则：`empty` 与 `error` 必须区分（网络失败不能显示"暂无数据"）；`loading` 期间禁止重复触发同一请求；错误文案优先用后端 `message`（见 `api-contract.md#5`），前端不另造文案。

## 3. 表单页（新增 / 编辑）

| 字段 | 控件 | 校验规则 | 校验时机 | 提示文案 |
|---|---|---|---|---|
| `studentNo` | `el-input` | 必填，8 位数字，与后端 `@Pattern` 一致 | blur | 学号必须是 8 位数字 |
| `name` | `el-input` | 必填，≤30 | blur | 姓名不能为空 |
| `classId` | `el-select` | 必填 | change | 请选择班级 |
| `enrollYear` | `el-input-number` | 必填，2000–2100 | change | 入学年份不合法 |

规则：前端校验只为体验，**规则与后端 DTO 注解一字不差**；两者不一致时以 `api-contract.md#6` 为准（后端是唯一权威）。

表现约定：

- 提交中：提交按钮 `loading` + 表单整体 `disabled`，防重复提交（后端再由 `uk_` 唯一索引兜底）。
- 失败（字段级）：把 `Result.data` 的 `List<FieldErrorVO>` 回填到对应字段（`formRef` 的字段错误），不弹全局错误。
- 失败（业务级，如 `STUDENT_NO_DUPLICATED`）：保留用户输入，用 `ElMessage.error(message)` 展示后端文案，前端不改字。
- 成功（新增）：关闭弹窗 → 重置表单 → 列表回 `page = 1` 重新查询（新增记录按默认排序可能在首或尾，回第 1 页最不容易困惑）。
- 成功（编辑）：关闭弹窗 → **保持当前页**重新查询（不跳页，避免用户丢失位置）。
- 关闭弹窗或切换路由前有未保存修改：提示"放弃未保存的修改？"。

## 4. 表格 — 分页 — 搜索联动

| 触发动作 | `page` 如何处理 | 其它 |
|---|---|---|
| 修改任一搜索条件 | 重置为 1 | 立即查询；清空旧数据，避免条件与新结果不匹配 |
| 修改排序（`sortBy` / `sortOrder`） | 重置为 1 | 只能传 `api-contract.md#7` 白名单内的字段 |
| 翻页 | 目标页 | 保留全部搜索条件与排序 |
| 修改 `pageSize` | 重置为 1 | 上限 100 |
| 删除当前页最后一条且 `page > 1` | `page - 1` | 重新查询，避免停在空页 |
| 新增成功 | 重置为 1 | 见第 3 节 |
| 离开页面再返回 | 按页面清单里写明的策略：保留条件与页码，或整体重置 | 二选一写清楚，不要"看情况" |

请求竞态：快速改条件会连续发出多个请求，必须只采纳最后一次响应。

```ts
let seq = 0
async function query() {
  const current = ++seq
  const res = await listStudents(buildQuery())
  if (current !== seq) return   // 过期响应直接丢弃，不写入 rows
  rows.value = res.list
}
```

## 5. Pinia store 划分原则

| 放进 store | 留在组件本地状态 |
|---|---|
| 跨页面 / 跨路由共享数据：当前用户、字典与枚举、权限标记 | 弹窗开关、`el-dialog` 可见性 |
| 需要在返回列表时保留的筛选条件（策略在第 1 节写明） | 表单草稿（未提交的输入） |
| 列表数据 + 分页参数（多组件共用同一份时） | 单页一次性数据（除非多处复用） |
| 需要被多个页面触发的刷新动作（如"新增后刷新列表"） | 单页 loading、分页器当前页 |

规则：

- store 命名 `useXxxStore`（`stores/student.ts` → `useStudentStore`），异步动作只调 `api/` 下的函数，**不在 store 里写 axios 配置**。
- **UI 提示不写进 store**：store 只负责"拿到数据 / 抛错"，`ElMessage`、路由跳转留在页面组件，便于复用与测试。
- store 里不放 DOM 引用、组件实例、`File` 对象；不放只在单个组件内使用的临时状态。

## 6. 与 API 契约的对应表

| 页面动作 | 端点 | 请求字段 | 响应用途 | 失败时的界面 |
|---|---|---|---|---|
| 进入列表 / 改条件 / 翻页 | `GET /api/v1/students` | `page,pageSize,keyword,classId,sortBy,sortOrder` | 渲染表格 + 分页器 | 表格区错误态 + 重试按钮 |
| 打开详情 | `GET /api/v1/students/{id}` | 路径参数 | 详情抽屉字段 | 顶部提示 `STUDENT_NOT_FOUND` 文案 |
| 点「新增」提交 | `POST /api/v1/students` | `StudentCreateDTO` | 关闭弹窗 + 回第 1 页 | 字段级回填 / 顶部提示 |
| 点「编辑」提交 | `PUT /api/v1/students/{id}` | `StudentUpdateDTO` | 关闭弹窗 + 当前页刷新 | 字段级回填 / 顶部提示 |
| 点「删除」确认 | `DELETE /api/v1/students/{id}` | 路径参数 | 当前页刷新（必要时回退页码） | 顶部提示 |
| 班级选择器输入 | `GET /api/v1/students/options` | `keyword,limit` | 下拉候选列表 | 下拉内联"加载失败，点击重试" |

闭合规则（G2 判据 4 的判定依据）：

1. 本表动作数 == `api-contract.md#3` 端点表行数；
2. 每个端点至少被一个动作引用；
3. 字段名逐个大小写比对（`studentNo` ≠ `student_no`）；
4. 每个错误码都能在第 6 节或第 2 节找到对应的界面表现。

## 7. 示例：P01 学生列表页 + 新增弹窗

**页面**：`views/student/StudentList.vue`（路由 `/students`，权限 `ANONYMOUS`；P06 引入 JWT 后写操作收紧为 `ROLE_ADMIN`）。依赖接口 = 上文 6 个端点；返回列表时**保留**搜索条件与页码。

**列表页状态表现**（按第 2 节填满）：

- `loading`：骨架屏 10 行；搜索框与分页器禁用。
- `loaded`：表格列 = 学号 / 姓名 / 性别 / 班级 / 入学年份 / 操作（编辑、删除）；分页器显示"共 N 条"。
- `empty`（无数据）：文案"暂无学生，点击右上角新增"，按钮直达新增弹窗。
- `empty`（条件无匹配）：文案"没有符合条件的学生"，提供"清空条件"按钮。
- `error`：文案"加载失败，请重试" + 重试按钮；**不渲染空态**。
- `submitting`：删除确认后该行按钮 loading、整表禁用；成功后按第 4 节决定是否回退页码。

**新增弹窗**：`components/StudentFormDialog.vue`，`:mode="'create' | 'edit'"`。

| 字段 | 控件 | 校验规则 | 校验时机 | 提示文案 |
|---|---|---|---|---|
| `studentNo` | `el-input` | 必填，8 位数字 | blur | 学号必须是 8 位数字 |
| `name` | `el-input` | 必填，≤30 | blur | 姓名不能为空 |
| `gender` | `el-radio-group` | 必填，`MALE` / `FEMALE` | change | 请选择性别 |
| `classId` | `el-select`（选项来自班级接口） | 必填 | change | 请选择班级 |
| `enrollYear` | `el-input-number` | 必填，2000–2100 | change | 入学年份不合法 |

提交与失败表现：

| 情形 | 界面行为 |
|---|---|
| 提交中 | 提交按钮 loading，表单 disabled，遮罩禁止关闭 |
| 成功 | 关闭弹窗 → 重置表单 → `page = 1` 重新查询 → 用后端 `message` 提示"新增成功" |
| `STUDENT_NO_DUPLICATED` | 顶部 `ElMessage.error`，`studentNo` 输入框标红，其余输入保留 |
| `CLASS_NOT_FOUND` | 提示"所选班级不存在，已刷新班级列表"，并重新拉取班级选项 |
| `COMMON_PARAM_INVALID` | 按 `List<FieldErrorVO>` 回填每个字段的错误，不弹全局提示 |
| 网络超时 / 5xx | 保留输入，提示"提交失败，请重试"，允许再次点击 |

**联动**：改 `keyword` 或 `classId` → `page = 1`；翻页 → 条件与排序不变；删除本页最后一条且 `page > 1` → `page - 1`；`pageSize` 改 20 → `page = 1`；快速改条件时按第 4 节的 `seq` 机制丢弃过期响应。

## 填写检查清单

- [ ] 页面清单里每个页面的路由、角色可见性、依赖接口、主要状态都已填。
- [ ] 依赖接口每一项都能在 `api-contract.md` 端点表找到（方法 + 路径完全一致）。
- [ ] 五个列表页状态都写了界面表现，`empty` 与 `error` 明确区分，两种 `empty` 也区分。
- [ ] 状态迁移覆盖：首屏、翻页、改条件、失败重试、删除本页最后一条、提交失败。
- [ ] 表单字段的前端校验规则与后端 DTO 注解逐字段比对一致。
- [ ] 提交中、字段级失败、业务级失败、成功后的跳转与刷新行为都已写明。
- [ ] 联动规则逐条写明，特别是"改搜索条件时 `page` 重置为 1"。
- [ ] 请求竞态的处理方式已写明（按序号丢弃过期响应）。
- [ ] store 与组件本地状态的边界清楚，UI 提示没有写进 store。
- [ ] 第 6 节动作数与契约端点数相等，字段名大小写比对通过。
- [ ] 离开页面再返回时，筛选条件是保留还是重置，已明确写出。
- [ ] 第 7 节示例已按本模板填满，可直接作为同类页面的参照。
