# nexus-config Specification

## Purpose
TBD - created by archiving change nes-patch-2026. Update Purpose after archive.
## Requirements
### Requirement: Nexus 私服发布配置

本能力 MUST 满足以下要求：Nexus 私服发布配置。

`pom.xml` 必须通过 `<distributionManagement>` 配置内网 Nexus 私服作为构件发布目标，URL 使用属性占位以便环境隔离。

#### Scenario: distributionManagement 配置
- **WHEN** 检查 `pom.xml`
- **THEN** 存在 `<distributionManagement>`，包含 `releases`（`${nexusReleaseUrl}`）与 `snapshots`（`${nexusSnapshotUrl}`）两个仓库

#### Scenario: 属性由 settings 提供
- **WHEN** 执行发布
- **THEN** `${nexusReleaseUrl}` / `${nexusSnapshotUrl}` 由 `~/.m2/settings.xml` 的 profile 属性解析
- **AND** server 凭证 id `releases` / `snapshots` 与 settings 中一致

### Requirement: 私服依赖解析

本能力 MUST 满足以下要求：私服依赖解析。

构建必须优先从内网 Nexus 私服解析依赖，隔离外网。

#### Scenario: 依赖从私服拉取
- **WHEN** 执行 `./mvnw -s settings.xml clean install`
- **THEN** 依赖通过 `192.168.131.36:8088/repository/maven-public/` 聚合仓库解析成功
- **AND** 不依赖外网直连

#### Scenario: 快照发布
- **WHEN** 执行 `./mvnw -s settings.xml deploy`
- **THEN** `2.7.18-nes.patch.1-SNAPSHOT` 制品发布到 snapshots 仓库

