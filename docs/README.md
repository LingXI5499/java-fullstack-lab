# 文档地图

`docs/` 分四层，各自的读者和用途不同。

| 目录 | 放什么 | 谁看 |
|---|---|---|
| [pipeline/](pipeline/) | 流水线本体：阶段规范、技能注册表、度量、升级日志 | 每次开发前 |
| [templates/](templates/) | 可直接复制的空白模板：需求卡、任务卡、ADR、设计完成卡、用例、发布记录、故障记录、变更影响卡、启动清单 | 写文档时 |
| [projects/](projects/) | 十个项目各自的规格与设计文档（每个项目一份目录） | 做某个项目时 |
| [reference/](reference/) | 需求来源与外部规范（Java 全栈母版等），只读 | 需要回溯源头时 |

## 阅读路线

- **第一次接触本仓库**：根 [README.md](../README.md) → [pipeline/delivery-pipeline.md](pipeline/delivery-pipeline.md) → [dev-env.md](dev-env.md)
- **准备开始一个新项目**：`projects/P0N/requirements.md` → [templates/project-charter.md](templates/project-charter.md) → [pipeline/delivery-pipeline.md](pipeline/delivery-pipeline.md)
- **卡住了**：`projects/P0N/progress.md` 看卡在哪一工序 → [pipeline/delivery-pipeline.md](pipeline/delivery-pipeline.md) 对应工序的产出物清单
- **做完一个项目**：`projects/P0N/retrospective.md` → [pipeline/upgrade-log.md](pipeline/upgrade-log.md) → [pipeline/metrics.md](pipeline/metrics.md)
