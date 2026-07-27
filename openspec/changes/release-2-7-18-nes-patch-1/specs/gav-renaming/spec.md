## MODIFIED Requirements

### Requirement: 版本号规范化

本能力 MUST 满足以下要求：发布版本号遵循 `X.Y.Z-nes.patch.N` 格式，不带 `-SNAPSHOT`。

#### Scenario: RELEASE 版本格式
- **WHEN** 检查 `pom.xml` 的 `version`
- **THEN** 版本号为 `2.7.18-nes.patch.1`

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
