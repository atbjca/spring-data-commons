## ADDED Requirements

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

本能力 MUST 满足以下要求：framework fork BOM 版本托管。

去特征化后的 Spring 组件版本 MUST 由 NES fork 的 framework BOM 统一托管，替代原先由 parent 传递的官方 `spring-framework-bom`。

#### Scenario: import fork BOM

- **WHEN** 检查 `pom.xml` 的 `<dependencyManagement>`
- **THEN** 存在 import 作用域的 `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom`，版本为 `5.3.39-nes.patch.1-SNAPSHOT`
- **AND** 该 BOM 覆盖 parent 传递的官方 `spring-framework-bom:5.3.31`

#### Scenario: fork BOM 可解析且回归通过

- **WHEN** 执行 `./mvnw clean test`
- **THEN** 构建能从内网 Nexus 解析 fork framework BOM 及其托管组件
- **AND** 全量测试通过（Failures 0 / Errors 0），相对官方 5.3.31 依赖无回归
