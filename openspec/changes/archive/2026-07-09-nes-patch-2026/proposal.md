## Why

`spring-data-commons` 2.7.x 已进入开源生命周期终点（EOL），官方不再为该版本线发布安全补丁（CVE-2026-41721 等仅在 3.5.12 / 4.0.6 修复）。BJCA 维护分支需要建立一套标准的 **NES（Never-Ending Support）安全维护流程**，与 `spring-kafka-2.9`、`spring-framework-5.3`、`spring-boot-2.7` 三个已完成的 NES fork 对齐，具体解决四个问题：

1. **本体安全修复**：本项目自身即 CVE-2026-41721 / 41711 / 41716 的宿主，攻击面真实存在于 `org.springframework.data.web` 包，必须 backport 官方补丁（而非像下游项目那样标"免疫"）。
2. **私服隔离**：统一依赖下载与构件发布渠道，通过内网 Nexus 私服隔离外网。
3. **GAV 去特征化**：重命名 Maven 坐标，规避 SCA 工具按 GAV 特征误报 CVE。
4. **文档化**：建立完整的漏洞报告体系与交付文档。

## What Changes

- **本体 CVE 修复（TDD，均本期完整修复）**：
  - CVE-2026-41721：`MapDataBinder` 的 `SpelParserConfiguration(false, true)` 开启集合自动增长但无上限，给其设置 `maximumAutoGrowSize` 上限（根因 `MapDataBinder.java:114`）。
  - CVE-2026-41711：`PropertyPath.from` 递归解析属性路径无深度限制导致 `StackOverflowError`，增加路径段数/长度上限（根因 `PropertyPath.java:344`，入口为 Sort 参数解析）。
  - CVE-2026-41716：`TypeDiscoverer.fieldTypes` 无界 `ConcurrentHashMap` 缓存永久驻留不存在的 property 名导致堆耗尽，改为有界缓存（根因 `TypeDiscoverer.java:60`）。
- **GAV 去特征化**：
  - GroupId: `org.springframework.data` → `cn.bjca.footstone.bpring.data`
  - ArtifactId: `spring-data-commons` → `bjca-footstone-bpring-data-commons`
  - Version: `2.7.19-SNAPSHOT` → `2.7.18-nes.patch.1-SNAPSHOT`
  - Parent: 方案 A 保留 `spring-data-parent`，版本 `2.7.19-SNAPSHOT` → `2.7.18`（已发布正式版）
- **私服配置**：`pom.xml` 增加 `<distributionManagement>`（`${nexusReleaseUrl}` / `${nexusSnapshotUrl}`），`settings.xml` 对齐内网 Nexus（`192.168.131.36:8088`）。
- **CVE 文档体系**（参考 `spring-boot-2.7/doc/`）：
  - `doc/VULNERABILITY_REPORT.md`：漏洞状态总览，6 态归一化。
  - `doc/CVE/CVE-xxxx.md`：每个 CVE 一个独立文档。
- **交付文档**：`doc/USER_MANUAL.md`、`doc/QUICK_START.md`、`doc/REQUIREMENTS.md`、`doc/GAV_MAPPING.md`。
- **构建快捷方式**：`Makefile`（build / test / install / deploy）。

## Capabilities

### New Capabilities

- `cve-remediation`：本体 CVE 的 backport 修复（TDD 驱动，三者本期均完整修复）——**本项目区别于其他 NES fork 的核心价值**。
- `gav-renaming`：GAV 去特征化重命名逻辑。
- `nexus-config`：Nexus 私服配置与发布管理。
- `cve-documentation`：漏洞报告与 CVE 独立文档体系。
- `build-documentation`：构建、发布与交付文档。

### Modified Capabilities

- （无——本项目此前无 openspec 受管能力）

## Impact

- **源码（本体修复）**：`src/main/java/org/springframework/data/web/MapDataBinder.java`、`SortHandlerMethodArgumentResolverSupport.java`、`ProxyingHandlerMethodArgumentResolver.java`。
- **测试**：`MapDataBinderUnitTests`、`SortHandlerMethodArgumentResolverUnitTests`、`ProxyingHandlerMethodArgumentResolverUnitTests`（TDD 先加触发用例）。
- **构建配置**：`pom.xml`（groupId / artifactId / version / parent / distributionManagement / java-module-name）、`settings.xml`。
- **新增文件**：`Makefile`、`doc/VULNERABILITY_REPORT.md`、`doc/CVE/*.md`、`doc/USER_MANUAL.md`、`doc/QUICK_START.md`、`doc/REQUIREMENTS.md`、`doc/GAV_MAPPING.md`。
- **下游影响**：依赖方需更新 GAV 坐标（Java `import` 无需改动，包名不变）。
- **风险点**：
  - Parent 采用方案 A（保留 `spring-data-parent`，锁定已发布正式版 `2.7.18`）；私服已确认有 `2.7.18`，阶段 0 干跑验证解析。
  - 本体修复改动 SpEL / PropertyPath / 缓存行为，需确保不破坏 `@ProjectedPayload`、Sort、property 解析正常语义（现有测试全绿 + 新增边界用例）。
