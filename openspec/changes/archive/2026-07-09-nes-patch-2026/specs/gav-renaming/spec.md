## ADDED Requirements

### Requirement: GroupId 去特征化

本能力 MUST 满足以下要求：GroupId 去特征化。

本项目发布坐标的 GroupId 必须从 `org.springframework.data` 重命名为 `cn.bjca.footstone.bpring.data`，以规避 SCA 工具按官方 GAV 特征误报 CVE。

#### Scenario: GroupId 重命名
- **WHEN** 构建并发布本项目
- **THEN** 发布 POM 中的 `groupId` 为 `cn.bjca.footstone.bpring.data`

#### Scenario: Java 包名保持不变
- **WHEN** 下游项目依赖本项目
- **THEN** Java 源码中的 `import org.springframework.data.*` 无需修改
- **AND** 仅 Maven 坐标发生变化

### Requirement: ArtifactId 去特征化

本能力 MUST 满足以下要求：ArtifactId 去特征化。

本项目的 ArtifactId 必须从 `spring-data-commons` 重命名为 `bjca-footstone-bpring-data-commons`。

#### Scenario: 主制品重命名
- **WHEN** 构建本项目
- **THEN** 发布的 ArtifactId 为 `bjca-footstone-bpring-data-commons`

### Requirement: 版本号规范化

本能力 MUST 满足以下要求：版本号规范化。

版本号必须遵循 `X.Y.Z-nes.patch.N-SNAPSHOT` 格式。

#### Scenario: 版本格式
- **WHEN** 检查 `pom.xml` 的 `version`
- **THEN** 版本号为 `2.7.18-nes.patch.1-SNAPSHOT`

### Requirement: Parent 引用处理

本能力 MUST 满足以下要求：Parent 引用处理。

parent（`org.springframework.data.build:spring-data-parent`）MUST 采用方案 A（保留原坐标），版本从 `2.7.19-SNAPSHOT` 锁定为已发布正式版 `2.7.18`，以保证可从内网私服解析。

#### Scenario: 父 POM 版本锁定为正式版
- **WHEN** 检查 `pom.xml` 的 `<parent>`
- **THEN** `groupId` 为 `org.springframework.data.build`，`artifactId` 为 `spring-data-parent`，`version` 为 `2.7.18`

#### Scenario: 父 POM 可解析
- **WHEN** 执行 `./mvnw -s settings.xml clean install`
- **THEN** 构建能从私服成功解析父 POM `spring-data-parent:2.7.18`
- **AND** 构建产物不残留官方 GAV 特征（本制品坐标已去特征化）

### Requirement: JPMS 模块名保持

本能力 MUST 满足以下要求：JPMS 模块名保持。

`java-module-name`（`spring.data.commons`）保持不变，避免破坏下游 JPMS `requires` 声明。

#### Scenario: 模块名不变
- **WHEN** 检查发布制品的模块描述
- **THEN** 自动模块名仍为 `spring.data.commons`
