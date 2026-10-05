# 可复制模板

> 这些模板是"可执行文档"的最小单元：每份都规定**什么时候填、填什么、怎么判定填完**。
> 它们不是形式主义——`docs/pipeline/delivery-pipeline.md` 的每道闸门都以这些模板为判定依据。

## 使用方式

1. 在 S0 开始前，把需要的模板复制到 `docs/projects/P0N/` 下并改名（例如 `requirement-card.md` → `requirements.md`）。
2. 占位符统一写成 `<...>`，填完即删掉尖括号。
3. 每份模板末尾都有「填写检查清单」，**清单没有全部勾选就不算这一步完成**。

```powershell
# 工序 01：起一个项目的文档骨架
$p = "docs\projects\P01"
Copy-Item docs\templates\project-charter.md "$p\charter.md"
Copy-Item docs\templates\progress-ledger.md "$p\progress.md"
```

---

## 按阶段索引

### S0 立项与规格（G1 闸门）

| 模板 | 用途 | 什么时候必须填完 |
|---|---|---|
| [project-charter.md](project-charter.md) | 项目章程：定位、技术新增点、不做范围、成功判据、风险 | 工序 01，在写任何需求之前 |
| [requirement-card.md](requirement-card.md) | 一张卡一个需求，含角色、场景、期望行为、优先级 | 工序 02 |
| [business-rule.md](business-rule.md) | 一张卡一条业务规则，写成"当…且…时，必须…" | 工序 03 |
| [acceptance-criteria.md](acceptance-criteria.md) | Given/When/Then 验收条件，覆盖正常/边界/异常/权限 | 工序 04 |
| [nfr.md](nfr.md) | 非功能需求：可测量的指标 + 测量方式 | 工序 05 |
| [traceability-matrix.md](traceability-matrix.md) | 需求 ↔ 规则 ↔ 接口 ↔ 表 ↔ 用例的追踪矩阵 | 与设计同步维护 |

### S1 设计（G2 闸门）

| 模板 | 用途 | 什么时候必须填完 |
|---|---|---|
| [data-design.md](data-design.md) | 概念-逻辑-物理三层数据设计；单库前缀隔离、逻辑外键、索引 | 工序 06 |
| [api-contract.md](api-contract.md) | 端点、DTO/VO、状态码、错误码、分页与排序白名单 | 工序 07 |
| [class-responsibility.md](class-responsibility.md) | 类职责、调用链、事务边界、分层越界反例 | 工序 08 |
| [page-state.md](page-state.md) | 页面清单、状态机、表格/表单/分页联动 | 工序 09 |
| [adr.md](adr.md) | 有取舍的地方记录决策、代价与复审触发条件 | 工序 10 |
| [design-completion.md](design-completion.md) | **G2 闸门检查表**：9 项判据逐条勾选 | G2 判定时 |

### S2 实现（G3 闸门）

| 模板 | 用途 | 什么时候必须填完 |
|---|---|---|
| [dev-task-card.md](dev-task-card.md) | 垂直切片任务卡：从 Vue 到 MySQL 的文件级边界 | 工序 11 之前 |
| [progress-ledger.md](progress-ledger.md) | 阶段账本、闸门记录、返工与度量（喂给 metrics.md） | 从工序 01 起持续更新 |
| [discovery-record.md](discovery-record.md) | 开发中发现的、与设计不符或未覆盖的事实 | 一旦发现就记，不等项目结束 |

### S3 验证与交付（G4 闸门）

| 模板 | 用途 | 什么时候必须填完 |
|---|---|---|
| [test-plan.md](test-plan.md) | 分层测试计划、工具矩阵、环境与数据策略 | 工序 19 |
| [test-case.md](test-case.md) | 用例：前置、数据、步骤、预期、**不应发生的变化** | 工序 19 |
| [api-verification.md](api-verification.md) | 接口验证/冒烟记录，含权限与错误路径矩阵 | 工序 20 |
| [release-record.md](release-record.md) | 发布记录：产物、迁移、配置、验证、回滚入口 | 工序 23 |
| [runbook.md](runbook.md) | 部署与回滚手册，含 Nginx / systemd / 可选 compose | 工序 22 |

### 复盘升级（下一个项目开工之前）

| 模板 | 用途 | 什么时候必须填完 |
|---|---|---|
| [retrospective.md](retrospective.md) | 四问复盘 + 度量 + 资产回写清单 | 工序 24 |
| [skill-upgrade-proposal.md](skill-upgrade-proposal.md) | 技能升级提案：落到 diff 级别的资产改动 | 工序 25 |
| [change-impact.md](change-impact.md) | 变更影响卡：需求中途变了才用 | 变更提出时 |
| [incident-record.md](incident-record.md) | 故障记录：现象、分层定位、根因、回归用例 | 出故障时 |

---

## 模板的四条共同约定

1. **可判定**：每份模板的"通过标准"都能回答是/否，不出现"合理""良好""基本完成"。
2. **可追踪**：编号贯穿需求 → 规则 → 接口 → 用例，任何一条都能双向查到来源与去向。
3. **不重复**：模板之间互相引用（如设计完成卡引用 data-design 的章节号），不复制内容。
4. **不写教程**：模板是填空处，不是知识讲解；讲解放在项目文档与技能库的 `references/` 里。
