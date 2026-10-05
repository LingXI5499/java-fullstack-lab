# Changelog

本仓库的每个里程碑都记录 `Added / Changed / Fixed / Learning Focus` 四段。
版本号形如 `p0N-vX.Y.Z`（例如 `p01-v1.0.0`）对应一个带注释的 Git tag。

## Unreleased

### Changed

- **新增铁律零：需求只能来自用户**（`AGENTS.md` §5.0 与 `delivery-pipeline.md` 第 0 节）。
  起因：在用户尚未给出十个项目的需求时，Agent 依据母版 §55 自行选定了 P01，并设计了
  表结构、业务规则、错误码与种子数据——这属于编造需求，已被用户叫停。
  现在规定：需求缺口只能返回问题清单；推测出的业务内容必须显式标注并可被一键否决。
  推测产物已隔离在 `feat/p01-student-info` 分支，`main` 保持干净。
- 新增需求接收入口 [docs/requirements-intake.md](docs/requirements-intake.md)：
  六项最少信息 + 两次人工确认（确认规格 → 确认设计 → 才写代码）。
- 母版文档的角色澄清为**候选与基线来源，不构成需求**。

### Added

- 仓库骨架：`README.md`、`.gitignore`、`.gitattributes`、`.editorconfig`、`LICENSE`、`AGENTS.md`、`CLAUDE.md`。
- `docs/`、`engineering/`、`tools/`、`projects/` 四层目录约定。
- 母版文档入库到 `docs/reference/`。
- 交付流水线规范 `docs/pipeline/delivery-pipeline.md`：7 阶段 / 23 工序 / 4 闸门 + 复盘升级三步。
- 技能与工作流地图 `docs/pipeline/skills-map.md`：10 个技能立项、6 个工作流、版本与升级规则。
- 度量与升级日志骨架 `docs/pipeline/metrics.md`、`docs/pipeline/upgrade-log.md`。
- 20 份可复制模板 `docs/templates/`（含索引 README），覆盖 S0–S3 与复盘升级的全部工序。
- `engineering/skills/jfsl-conventions/SKILL.md`：唯一约定源（技术基线、包结构、命名、错误码、数据库铁律）。
- `tools/lab.ps1`：门禁工具（`doctor` / `status` / `verify` / `tag` / `skills`）。
  实测：`doctor` 7 项全通过；`verify` 能真实拦住物理外键、跨项目表前缀、被跟踪的本地配置、
  非 Java 17 字节码与缺失的交付物。

### Learning Focus

- 仓库在"第一行业务代码之前"应该长什么样：约定、目录、门禁先到位。
- **需求与推测必须分开**：把候选清单当需求，产出越多越难收拾。
- 为什么十个项目要共用一个技能库、一个数据库，而不是各建一套。
- 逻辑外键把引用完整性从数据库搬到应用层之后，业务规则与用例要怎么跟着补。
- PowerShell 三个坑：脚本必须保持 ASCII（5.1 按 GBK 解析无 BOM 文件）、
  不要把函数参数命名为 `$Args` 这类自动变量、文件读写一律走 `[IO.File]` + UTF8。

<!--
## p01-v1.0.0

### Added
### Changed
### Fixed
### Learning Focus
-->
