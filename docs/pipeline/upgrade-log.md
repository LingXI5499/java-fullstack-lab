# 技能与工作流升级日志

> 这个文件是流水线"越来越强"的证据链。
> 每完成一个项目，必须在这里新增一个批次；**没有新增批次 = 本轮复盘未完成，不允许开工下一个项目**。
> 每条改动的形式要求：能落到 diff 级别（改哪个文件、加哪一条规则），不允许只写"优化了流程"。

## 批次格式

```markdown
## P0N 学生信息管理（项目完成后）
- 触发：项目复盘 `projects/P01-student-info/docs/retrospective.md`
- 本批升级：
  | 提案 | 目标资产 | 版本变化 | 改动摘要 | 期望效果 |
  |---|---|---|---|---|
  | SR-001 | `engineering/skills/jfsl-crud-gen/SKILL.md` | 1.0.0 → 1.1.0 | 生成 Mapper XML 时强制带 `<where>` 包裹动态条件 | 减少动态 SQL 语法返工 |
- 提交：`chore(skills): ...`
- 度量：`docs/pipeline/metrics.md` 已更新（T(N)、返工次数）
```

---

## 批次 0：基线（仓库初始化，尚无项目）

- 触发：仓库初始化（A0）
- 本批升级：

| 提案 | 目标资产 | 版本变化 | 改动摘要 | 期望效果 |
|---|---|---|---|---|
| SR-000 | `AGENTS.md` + `docs/pipeline/delivery-pipeline.md` | — | 确立三条铁律：串行开发、每步留产出物、经验必须回写资产 | 从第一行代码起就有约束 |
| SR-000 | `docs/pipeline/delivery-pipeline.md` §2.5 | — | 确立单库 `java_fullstack_lab` + `p0N_` 前缀隔离与越界三条硬规则 | 避免十个项目互相污染 |
| SR-000 | `AGENTS.md` §2 | — | 明确推迟 OSS/域名/HTTPS，到真正需要时再引入 | 防止骨架提前变重 |

- 提交：`chore: initialize java-fullstack-lab repository`
- 度量：暂无项目数据

---

<!--
## P01 学生信息管理（项目完成后）

- 触发：`projects/P01-student-info/docs/retrospective.md`
- 本批升级：
  | 提案 | 目标资产 | 版本变化 | 改动摘要 | 期望效果 |
  |---|---|---|---|---|
- 提交：
- 度量：`docs/pipeline/metrics.md`
-->
