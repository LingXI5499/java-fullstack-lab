# 技能与工作流地图

> 全仓库**只有这一份**技能库与工作流库，所有项目共用（见 [delivery-pipeline.md](delivery-pipeline.md) 第 1 节）。
> 技能数量上限 **12 个**：新增必须先在这里立项。

---

## 1. 技能清单

技能放在 `engineering/skills/<name>/`，结构遵循 DSH 技能约定：

```text
jfsl-<name>/
├─ SKILL.md           必需：技能说明与执行指令（正文）
├─ CHANGELOG.md       必需：语义化版本与改动记录
├─ references/        可选：按需加载的详细规范
├─ templates/         可选：可直接复制到项目的文件模板
└─ scripts/           可选：本技能自带的幂等脚本
```

### 1.1 当前立项的技能

| 技能 | 版本 | 负责工序 | 一句话职责 | 产出物 |
|---|---|---|---|---|
| `jfsl-conventions` | 1.0.0 | 全部 | 唯一约定源头：版本基线、包结构、命名、错误码、注释风格、秘密管理 | 与 [AGENTS.md](../../AGENTS.md) 同步的规则集 |
| `jfsl-scaffold` | 1.0.0 | 11 | 生成可编译的 backend + frontend + deploy 骨架 | `tools/templates/` 落地后的项目目录 |
| `jfsl-data-design` | 1.0.0 | 06, 12 | ER → DDL → 迁移 → 种子；单库前缀隔离与索引规则 | `database/schema.sql` `seed.sql` |
| `jfsl-api-design` | 1.0.0 | 07 | 契约先行：资源、DTO/VO、状态码、错误码、分页与排序白名单 | 项目 `docs/api.md` |
| `jfsl-crud-gen` | 1.0.0 | 13, 14, 16 | 由模块规格生成八件套（Entity/DTO/VO/Query/Mapper+XML/Service+Impl/Controller/测试骨架） | `backend/` 下对应文件 |
| `jfsl-frontend-page` | 1.0.0 | 18 | 列表页 / 表单弹窗 / 分页搜索联动 / ECharts 骨架 | `frontend/src/` 下页面与 `api.ts` |
| `jfsl-test-layered` | 1.0.0 | 19, 20 | 由验收条件生成分层用例与冒烟脚本 | `src/test/` `tools/smoke/P0N.ps1` |
| `jfsl-release` | 1.0.0 | 21, 22, 23 | 打包、Nginx/systemd/compose 配置、Runbook、tag | `deploy/` + 发布记录 |
| `jfsl-review-gate` | 1.0.0 | G1–G4 | 四道闸门与提交前七问检查 | 门禁报告 |
| `jfsl-retro-skill-upgrade` | 1.0.0 | 24, 25, 26 | 把复盘结论转成资产的 diff 级改动 | 升级提案 + 已改动的资产 |

### 1.2 明确不立项的方向

| 不立项 | 理由 |
|---|---|
| Redis / MQ / 分布式技能 | 属于流水线下一版，见 AGENTS.md 非目标 |
| 各项目专属技能（如 `pharmacy-inventory`） | 业务知识属于项目文档，不属于可复用技能 |
| "代码风格美化""更优雅的重构" | 无法判定，也无法验证是否减少返工 |
| 单个框架的 API 手册 | 属于 `references/` 内容，不单独成技能 |

新增技能的判据（四条全满足才可以立项）：

1. 在**至少两个项目**里重复出现同类动作；
2. 能写成**可判定**的规则或可执行的脚本；
3. 有明确的**触发时机**（对应具体工序号）；
4. 能**减少具体动作**（少写文件、少改一处、少一次返工），而不是"更规范"。

---

## 2. 工作流清单

工作流放在 `engineering/workflows/`，是**阶段编排**：说明某个阶段该按什么顺序调用哪些技能、每步的人工确认点在哪。

| 工作流 | 覆盖工序 | 用途 | 人工确认点 |
|---|---|---|---|
| `stage-spec.md` | 01–05 | 需求 → 需求卡 + 业务规则 + 验收条件 + NFR | 用户确认 G1 规格闸门 |
| `stage-design.md` | 06–10 | 规格包 → ER + API 契约 + 包结构 + ADR | 用户确认 G2 设计闸门 |
| `stage-implement.md` | 11–18 | 骨架 → 分层实现（含 `TODO(user)` 位） | 用户实现业务规则后复核 |
| `stage-accept.md` | 19–23 | 测试 → 冒烟 → 打包 → 部署 → tag | 用户确认 G4 发布闸门 |
| `project-kickoff.md` | 01–26 | **串起一个项目的完整生命周期**，并强制"一次一个项目" | 每个闸门 |
| `pipeline-v2.md` | 01–26 | 盲测：用一个全新需求验证流水线是否真的"一定能做出来" | 全程记录，不改流水线 |

工作流的硬约束：

- `project-kickoff.md` 在工序 01 之前必须先执行一次"上一个项目复盘已完成"的检查；未完成则终止。
- 任何工作流都不得跨项目并行。
- 工作流只**编排**技能，不复制技能内容；技能改动只发生在技能库里。

---

## 3. 技能版本与升级

版本规则：

| 变更 | 版本位 | 触发条件 |
|---|---|---|
| MAJOR | `x.0.0` | 改了会让**已完成项目不合规**的规则 |
| MINOR | `.x.0` | 新增能力、新增模板、新增检查项 |
| PATCH | `.0.x` | 修正错误、措辞澄清、补示例 |

升级流程：

```text
项目复盘（工序 24）
  ↓
写技能升级提案（工序 25，用 docs/templates/skill-upgrade-proposal.md）
  ↓
按提案实际修改 engineering/skills/**（含 CHANGELOG.md 版本 +1）
  ↓
更新 docs/pipeline/upgrade-log.md（触发项目 / 改动 / 期望效果）
  ↓
更新 docs/pipeline/metrics.md（工序 11–23 耗时与返工次数）
  ↓
同一个提交（chore(skills): ...），然后才允许开工下一个项目
```

**没有实际修改资产的复盘不算完成**——这是铁律三的落地方式。

---

## 4. 技能与工作流的自检

每次升级后运行：

```powershell
.\tools\lab.ps1 skills --check     # 结构完整性：SKILL.md / CHANGELOG.md / 版本号 / 与 AGENTS.md 一致性
.\tools\lab.ps1 skills --sync      # 同步到用户技能目录，使其在会话中可调用
```

`--check` 至少校验：

- 每个技能目录都有 `SKILL.md` 与 `CHANGELOG.md`
- `SKILL.md` 头部声明了版本号与负责工序号
- `CHANGELOG.md` 最新版本与 `SKILL.md` 声明一致
- `jfsl-conventions` 中的规则与 `AGENTS.md` 无冲突
- 技能数量不超过 12
