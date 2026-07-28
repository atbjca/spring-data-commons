## MODIFIED Requirements

### Requirement: 私服依赖解析

构建 MUST 优先从内网 Nexus 私服解析依赖，隔离外网。目标 RELEASE 只能在完整本地验证及 Nexus 不存在性检查通过后发布一次。

#### Scenario: 依赖从私服拉取
- **WHEN** 协调主会话串行执行已记录的本地 install 命令
- **THEN** 依赖通过 `192.168.131.36:8088/repository/maven-public/` 聚合仓库解析成功
- **AND** 不依赖外网直连

#### Scenario: RELEASE 发布
- **WHEN** 协调主会话确认完整 publication 集在 Nexus 中均不存在并执行一次 `make deploy`
- **THEN** `2.7.18-nes.patch.1` 制品发布到 `${nexusReleaseUrl}` 对应的 releases 仓库
- **AND** 不覆盖、删除或重发已存在的 RELEASE 资产
