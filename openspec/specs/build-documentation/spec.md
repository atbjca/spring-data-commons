# build-documentation Specification

## Purpose
TBD - created by archiving change nes-patch-2026. Update Purpose after archive.
## Requirements
### Requirement: 交付文档完整性

本能力 MUST 满足以下要求：交付文档完整性。

必须提供面向使用者与运维的交付文档，至少包含用户手册与快速入门。

#### Scenario: 快速入门
- **WHEN** 查看 `doc/QUICK_START.md`
- **THEN** 包含最小化的依赖引入、构建、测试步骤，使用新 GAV 坐标

#### Scenario: 用户手册
- **WHEN** 查看 `doc/USER_MANUAL.md`
- **THEN** 说明本 fork 的定位、GAV 变化、安全修复内容、与官方版本差异

#### Scenario: 需求与版本清单
- **WHEN** 查看 `doc/REQUIREMENTS.md`
- **THEN** 列出目标版本、依赖版本清单、CVE 修复范围

#### Scenario: GAV 映射表
- **WHEN** 查看 `doc/GAV_MAPPING.md`
- **THEN** 提供原始 GAV 与 NES GAV 的对照表

### Requirement: 构建快捷命令

本能力 MUST 满足以下要求：构建快捷命令。

必须提供 `Makefile` 封装常用 Maven 构建命令。

#### Scenario: Makefile 目标
- **WHEN** 查看 `Makefile`
- **THEN** 至少包含 `build`、`test`、`install`、`deploy` 目标
- **AND** 各目标使用项目 `settings.xml` 与 `mvnw`

