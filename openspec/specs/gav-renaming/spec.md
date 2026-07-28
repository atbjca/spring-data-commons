# gav-renaming Specification

## Purpose
TBD - created by archiving change nes-patch-2026. Update Purpose after archive.
## Requirements
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

本能力 MUST 满足以下要求：发布版本号遵循 `X.Y.Z-nes.patch.N` 格式，不带 `-SNAPSHOT`。

#### Scenario: RELEASE 版本格式
- **WHEN** 检查 `pom.xml` 的 `version`
- **THEN** 版本号为 `2.7.18-nes.patch.1`

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

### Requirement: 直接依赖坐标去特征化

本能力 MUST 满足以下要求：直接依赖坐标去特征化。

本项目发布 POM 的 `<dependencies>` 中，直接依赖的 Spring 组件坐标 MUST 从官方 `org.springframework:spring-*` 重命名为 NES fork 坐标 `cn.bjca.footstone.bpring:bjca-footstone-bpring-*`，以消除发布物料中残留的官方 GAV 特征、避免 SCA 工具按官方坐标误报 CVE。

#### Scenario: 依赖坐标重命名

- **WHEN** 检查发布 POM 的 `<dependencies>`
- **THEN** 直接依赖的 Spring 组件（core / beans / context / expression / tx / oxm / web / webflux / webmvc）groupId 为 `cn.bjca.footstone.bpring`、artifactId 前缀为 `bjca-footstone-bpring-`
- **AND** 不残留 `org.springframework:spring-*` 官方依赖坐标

#### Scenario: Java 包名与下游无影响

- **WHEN** 下游项目依赖本项目
- **THEN** 依赖去特征化仅改变本项目内部依赖坐标，下游只声明本项目主制品坐标即可
- **AND** Java 源码中的 `import org.springframework.*` 无需修改

### Requirement: framework fork BOM 版本托管

去特征化后的 Spring 组件版本 MUST 由已发布并经 Nexus 验证的 NES Framework BOM 统一托管，替代原先由 parent 传递的官方 `spring-framework-bom`。

#### Scenario: import RELEASE fork BOM

- **WHEN** 检查 `pom.xml` 的 `<dependencyManagement>`
- **THEN** 存在 import 作用域的 `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom`，版本为 `5.3.39-nes.patch.1`
- **AND** 该 BOM 覆盖 parent 传递的官方 `spring-framework-bom:5.3.31`
- **AND** 发布 POM 不包含内部 `cn.bjca.footstone` SNAPSHOT 版本

#### Scenario: RELEASE fork BOM 可解析且回归证据有效

- **WHEN** 既有全量回归证据覆盖当前源码且 release-only diff 不含生产或测试源码变化
- **THEN** 可按获准例外复用该证据，不重复执行 `make build` 或 `make test`
- **AND** 协调主会话仍须串行执行已记录的本地 install 命令并完成 RELEASE-only consumer 验证

