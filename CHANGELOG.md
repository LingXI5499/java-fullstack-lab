# Changelog

本仓库的每个里程碑都记录 `Added / Changed / Fixed / Learning Focus` 四段。
版本号形如 `p0N-vX.Y.Z`（例如 `p01-v1.0.0`）对应一个带注释的 Git tag。

## Unreleased

### Added

- 仓库骨架：`README.md`、`.gitignore`、`.gitattributes`、`.editorconfig`、`LICENSE`、`AGENTS.md`、`CLAUDE.md`。
- `docs/`、`engineering/`、`tools/`、`projects/` 四层目录约定。
- 母版文档入库到 `docs/reference/`，作为流水线的需求来源。
- 交付流水线规范 `docs/pipeline/delivery-pipeline.md`：7 阶段 / 23 工序 / 4 闸门 + 复盘升级三步。
- 技能与工作流地图 `docs/pipeline/skills-map.md`：10 个技能立项、6 个工作流、版本与升级规则。
- 度量与升级日志骨架 `docs/pipeline/metrics.md`、`docs/pipeline/upgrade-log.md`。
- P01 规格包 `docs/projects/P01/`：项目章程 + 需求/业务规则/验收条件/非功能需求。
- 20 份可复制模板 `docs/templates/`（含索引 README），覆盖 S0–S3 与复盘升级的全部工序。
- **标识符约定**：`REQ/BR/AC/TC/NFR/TASK-<模块>-<两位序号>`，模块段与场景段全大写、序号两位补零；
  写入 `AGENTS.md` §3.7 与交付流水线规范第 2 节。
- **错误码清单**：P01 第 9.1 节给出本项目封闭错误码集合与对应异常子类。

### Changed

- 明确三条铁律：**串行开发**（一次只做一个项目）、每步留产出物、经验必须回写资产（文档不算）。
- 确立数据库铁律：单库 `java_fullstack_lab` + `p0N_` 前缀隔离、禁止跨项目 JOIN、
  **禁止物理外键只用逻辑外键**。
- 推迟 OSS / 域名 / HTTPS，到真正需要时再引入。

### Learning Focus

- 仓库在"第一行业务代码之前"应该长什么样：约定、目录、门禁先到位。
- 为什么十个项目要共用一个技能库、一个数据库，而不是各建一套。
- 逻辑外键把引用完整性从数据库搬到应用层之后，业务规则与用例要怎么跟着补。

<!--
## p01-v1.0.0

### Added
### Changed
### Fixed
### Learning Focus
-->
