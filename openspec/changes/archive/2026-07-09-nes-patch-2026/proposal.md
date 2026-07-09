## Why

`spring-data-commons` 3.5.x 是官方仍在维护的活跃版本线。BJCA 维护分支 `3.5.x-bjca-patch` 需要与已完成的 `spring-data-commons-2.7`、`spring-kafka-2.9`、`spring-framework-5.3`、`spring-boot-2.7` 四个 NES（Never-Ending Support）fork 对齐，建立统一的**安全维护 + 私服隔离 + 文档化**流程。

与 2.7 fork（EOL、需自己 backport）**本质不同**：本项目的三个本体 CVE（CVE-2026-41721 / 41711 / 41716）**官方已在 3.5.12 / 3.5.13 基线内修复**，源码中防护已存在。因此本次**不做任何业务代码修改**（避免画蛇添足），只需：验证官方修复生效、去特征化 GAV、配置私服、补齐文档。具体解决三个问题：

1. **GAV 去特征化**：重命名 Maven 坐标，规避 SCA 工具按官方 GAV 特征误报 CVE。（**必做**）
2. **私服隔离**：统一依赖下载与构件发布渠道，通过内网 Nexus 私服隔离外网。
3. **文档化**：建立完整的漏洞报告体系与交付文档，如实记录三个本体 CVE 由官方 upstream 修复的事实与验证证据。

## What Changes

- **本体 CVE 验证（零源码改动）**：三个 CVE 官方已在基线修复，本次仅补充/运行回归测试确认防护生效，状态记 `✅已修复`，修复方标注为**官方 upstream**（非本 fork backport）。
  - CVE-2026-41721：`MapDataBinder` 已用 `SpelParserConfiguration(false, true, 1024)` 限制 SpEL 集合自动增长上限（官方提交 `e33b68517`，`MapDataBinder.java:124`）。
  - CVE-2026-41711：`PropertyPath.create` 已用 `remainingDepth` 计数器统一约束点分与 camel-case 两条递归路径（官方提交 `c3b2abf29`，`PropertyPath.java:449-493`，上限 `MAX_PARSE_DEPTH=1000`）。
  - CVE-2026-41716：`TypeDiscoverer` 已一次性初始化属性、消除无界 miss 缓存（官方提交 `e71100a8b` / `accc38344`）。
- **GAV 去特征化**：
  - GroupId: `org.springframework.data` → `cn.bjca.footstone.bpring.data`
  - ArtifactId: `spring-data-commons` → `bjca-footstone-bpring-data-commons`
  - Version: `3.5.14-SNAPSHOT` → `3.5.13-nes.patch.1-SNAPSHOT`
  - Parent: 方案 A 保留 `spring-data-parent`，版本 `3.5.14-SNAPSHOT` → `3.5.13`（已发布正式版，本地 `.m2` 已确认存在）
  - `java-module-name`（`spring.data.commons`）保持不变。
- **上游 spring-framework 依赖替换**（与 spring-data-commons-2.7 一致）：
  - 导入 fork BOM `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:6.2.19-nes.patch.1-SNAPSHOT`。
  - 10 个 spring 依赖 `org.springframework:spring-*` → `cn.bjca.footstone.bpring:bjca-footstone-bpring-*`（core/beans/context/expression/tx/oxm/web/webflux/webmvc/core-test）。
  - `spring-hateoas` 无对应 fork，保留官方坐标。
- **私服配置**：`pom.xml` 增加 `<distributionManagement>`（`${nexusReleaseUrl}` / `${nexusSnapshotUrl}`），凭证/属性由 `~/.m2/settings.xml` 的 `bjca` profile 提供。
- **CVE 文档体系**（参考 `spring-boot-2.7/doc/`）：
  - `doc/VULNERABILITY_REPORT.md`：漏洞状态总览，6 态归一化。
  - `doc/CVE/CVE-xxxx.md`：每个 CVE 一个独立文档。
- **交付文档**：`doc/USER_MANUAL.md`、`doc/QUICK_START.md`、`doc/REQUIREMENTS.md`、`doc/GAV_MAPPING.md`。
- **构建快捷方式**：`Makefile`（build / test / install / deploy）。

## Capabilities

### New Capabilities

- `cve-documentation`：漏洞报告与 CVE 独立文档体系，含"验证官方修复生效"的回归测试场景。
- `gav-renaming`：GAV 去特征化重命名逻辑。
- `nexus-config`：Nexus 私服配置与发布管理。
- `build-documentation`：构建、发布与交付文档。

### Modified Capabilities

- （无——本项目此前无 openspec 受管能力）

## Impact

- **源码**：**无业务代码改动**（三个本体 CVE 官方基线已修复）。仅可能新增/补充测试用例以固化验证。
- **测试**：`MapDataBinderUnitTests`、`PropertyPathUnitTests`、`TypeDiscovererUnitTests`（确认既有防护测试存在并通过；如缺失则补充边界用例）。
- **构建配置**：`pom.xml`（groupId / artifactId / version / parent / distributionManagement / dependencyManagement 导入 fork BOM / 10 个 spring 依赖坐标替换）。
- **新增文件**：`Makefile`、`doc/VULNERABILITY_REPORT.md`、`doc/CVE/*.md`、`doc/USER_MANUAL.md`、`doc/QUICK_START.md`、`doc/REQUIREMENTS.md`、`doc/GAV_MAPPING.md`。
- **下游影响**：依赖方需更新 GAV 坐标（Java `import` 无需改动，包名不变）。
- **风险点**：
  - Parent 采用方案 A（保留 `spring-data-parent`，锁定已发布正式版 `3.5.13`）；本地 `.m2` 已确认存在 `spring-data-parent:3.5.13.pom`，阶段 0 再干跑验证私服解析。
  - 本次零源码改动，风险主要在构建坐标与发布链路；通过 `clean install` / `clean deploy` 干跑验证。
