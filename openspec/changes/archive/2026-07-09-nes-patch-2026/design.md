# Design — nes-patch-2026 (spring-data-commons 3.5)

> 本文档记录 spring-data-commons-3.5 NES fork 改造的技术设计与关键决策。
> **核心特征：三个本体 CVE 官方已在 3.5.12 / 3.5.13 基线修复，本次零源码改动，只做验证 + GAV + 私服 + 文档。**

## 1. 背景与约束

| 项 | 值 |
|----|-----|
| 基线版本 | `3.5.13`（官方已发布正式版；当前分支 pom 为 `3.5.14-SNAPSHOT`，改造后锁定 `3.5.13` 为基线与 parent 版本） |
| 目标版本 | `3.5.13-nes.patch.1-SNAPSHOT` |
| 构建工具 | Maven（`mvnw`，本项目自带 wrapper；**非 Gradle**，`~/dev` 下 gradle 版本本项目用不到） |
| 私服 | `http://192.168.131.36:8088`（release `/repository/releases/`，snapshot `/repository/snapshots/`，聚合 `/repository/maven-public/`） |
| Git 分支 | `3.5.x-bjca-patch` |
| Java | 3.5.x 基线为 **Java 17**（与 2.7 的 Java 8 不同）。本地 sdkman 可选 `17.0.17-amzn`。修复/测试代码遵循项目现有 Java 17 语法约定 |

**核心约束（与 2.7 的根本差异）**：官方对 CVE-2026-41711 / 41716 / 41721 的修复**正是在 3.5.12 / 4.0.6 发布**。本项目基线 3.5.13 已包含这些修复，因此**本 fork 不承担 backport 角色，只做验证与文档留痕**。这直接体现"避免刻舟求剑"——不照抄 2.7 的"本体修复"动作。

## 2. 本体 CVE 验证方案（核心，零源码改动）

### 2.1 攻击面总览（与 2.7 相同，但防护已就位）

```
   HTTP 请求（不可信输入）
        │
        ├── ?sort=a.b.c.d... / 超长 camelCase ─► SortHandlerMethodArgumentResolver
        │                                          └─► PropertyPath.from()
        │                                              【CVE-2026-41711】✅ remainingDepth 计数器已防护
        │
        ├── @ProjectedPayload 表单参数 ──────────► ProxyingHandlerMethodArgumentResolver
        │       ?list[99999999]=x                    └─► MapDataBinder
        │                                                【CVE-2026-41721】✅ autoGrow 上限 1024 已防护
        │
        └── 大量不同 property 名 ─────────────────► PropertyPath.from / TypeDiscoverer
                ?p1=&p2=&p3=...                       └─► fieldTypes 缓存
                                                         【CVE-2026-41716】✅ 一次性初始化、无 miss 缓存
```

### 2.2 逐个 CVE 的现状核对（源码已含官方修复）

#### CVE-2026-41721 — MapDataBinder SpEL 自动增长上限

- **官方修复现状**：`MapDataBinder.java:124`
  ```java
  new SpelParserConfiguration(false, true, MapDataBinder.DEFAULT_COLLECTION_LIMIT)); // DEFAULT_COLLECTION_LIMIT=1024
  ```
  官方提交 `e33b68517 "Init MapDataBinder with custom collection limit"`。上限为 **1024**（注意：与 2.7 backport 采用的 256 不同，本次以官方 3.5 实现为准，不改动）。
- **验证方式**：确认 `MapDataBinderUnitTests` 存在对应边界用例并通过；若缺失，补充"绑定超大集合索引应被上限拒绝"用例（红→绿仅用于固化，不改业务代码）。

#### CVE-2026-41711 — PropertyPath 递归深度上限

- **官方修复现状**：`PropertyPath.java:449-493`，`create(...)` 携带 `remainingDepth` 参数，入口 `remainingDepth = MAX_PARSE_DEPTH - base.size()`（`:436`），递归时 `remainingDepth - 1`（`:493`），超限抛 `IllegalArgumentException(PARSE_DEPTH_EXCEEDED)`。官方提交 `c3b2abf29 "Consistently apply max traversal depth"` —— **统一约束点分与 camel-case 两条递归路径**，正是 2.7 需要 backport 的那个修复。
- **验证方式**：确认 `PropertyPathUnitTests` 含超深/超长 camelCase 路径用例并通过。

#### CVE-2026-41716 — TypeDiscoverer 缓存

- **官方修复现状**：官方提交 `e71100a8b "Initialize properties of a type in one pass in TypeDiscoverer"` 与 `accc38344 "Retain subclass fields over masked superclass fields"` —— 改为一次性初始化类型属性，消除对不存在 property 名的无界 miss 缓存。
- **验证方式**：确认 `TypeDiscovererUnitTests` 相关用例通过；必要时补充"重复查询不存在 property 名，缓存不无限增长"断言。

### 2.3 状态映射（已确定）

| CVE | 攻击面 | 状态 | 修复方 |
|-----|:---:|:---:|:---:|
| CVE-2026-41721 | MapDataBinder | ✅已修复 | 官方 upstream（3.5.12，`e33b68517`） |
| CVE-2026-41711 | PropertyPath | ✅已修复 | 官方 upstream（3.5.12，`c3b2abf29`） |
| CVE-2026-41716 | TypeDiscoverer | ✅已修复 | 官方 upstream（`e71100a8b`/`accc38344`） |

> **与 2.7 的措辞区分**：CVE 文档"本项目应对措施"段必须写明"官方基线已修复，本 fork 仅验证并文档化"，**不得**照抄 2.7 的"本 fork backport 官方补丁"表述（那与 3.5 事实不符，属知识幻觉）。

## 3. GAV 去特征化设计

| 坐标 | 原值 | 新值 |
|------|------|------|
| groupId | `org.springframework.data` | `cn.bjca.footstone.bpring.data` |
| artifactId | `spring-data-commons` | `bjca-footstone-bpring-data-commons` |
| version | `3.5.14-SNAPSHOT` | `3.5.13-nes.patch.1-SNAPSHOT` |
| java-module-name | `spring.data.commons` | 保持不变（JPMS 模块名，改动影响下游 `requires`） |

**Parent 处理 —— 已定：方案 A（保留原 parent 坐标，版本锁定 3.5.13 正式版）**：

```
✅ 方案 A：保留原 parent 坐标      ── 最小改动，parent 仅构建期依赖、不进制品 GAV 特征
   方案 B：同步 fork parent 去特征  ──（未采用）工作量翻倍
   方案 C：扁平化去 parent          ──（未采用）
```

关键修正：原 `pom.xml` 的 parent 版本是 `3.5.14-SNAPSHOT`（未发布快照），改为已发布正式版 **`3.5.13`**：

```xml
<parent>
  <groupId>org.springframework.data.build</groupId>
  <artifactId>spring-data-parent</artifactId>
  <version>3.5.13</version>   <!-- 原为 3.5.14-SNAPSHOT，改为已发布正式版 -->
</parent>
```

理由：
- 本地 `.m2` 已确认存在 `spring-data-parent/3.5.13/spring-data-parent-3.5.13.pom`，可解析。
- parent 只在构建期解析，不体现在发布制品的 GAV 特征上，SCA 扫描的是制品自身坐标（已去特征化）。
- 与制品版本 `3.5.13-nes.patch.1-SNAPSHOT` 的基线（3.5.13）一致。

**包名（package）不改**：`org.springframework.data.*` 保持，下游 `import` 无需改动。

### 3.1 上游 spring-framework 依赖替换

与 spring-data-commons-2.7 一致，本项目还将上游 spring 依赖指向去特征化 fork，使制品依赖链不残留官方 GAV 特征：

- **导入 fork BOM**（`<dependencyManagement>`）：`cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:6.2.19-nes.patch.1-SNAPSHOT`（`type=pom`、`scope=import`），统一管理 spring-* 版本，对应官方 3.5.13 基线的 spring 6.2.19。
- **10 个依赖替换**：`spring-core/beans/context/expression/tx/oxm/web/webflux/webmvc/core-test` → `bjca-footstone-bpring-*`，groupId `org.springframework` → `cn.bjca.footstone.bpring`，版本由 BOM 管理（声明处不写 version）。
- **保留官方**：`spring-hateoas`（`org.springframework.hateoas`，无对应 fork）。
- **验证**：fork 依赖的 `Automatic-Module-Name` 虽为 `bjca.footstone.bpring.*`，但 Java 包名仍是 `org.springframework.*`，源码 `import` 无影响；`./mvnw clean test` 全量 3630 通过、含 ArchUnit `DependencyTests`，无回归。

## 4. 私服配置设计

`pom.xml` 增加：
```xml
<distributionManagement>
  <repository>
    <id>releases</id>
    <name>Nexus Release Repository</name>
    <url>${nexusReleaseUrl}</url>
  </repository>
  <snapshotRepository>
    <id>snapshots</id>
    <name>Nexus Snapshot Repository</name>
    <url>${nexusSnapshotUrl}</url>
  </snapshotRepository>
</distributionManagement>
```
`${nexusReleaseUrl}` / `${nexusSnapshotUrl}` 由 `~/.m2/settings.xml` 的 `bjca` profile（默认激活）提供，server 凭证 id = `releases` / `snapshots`。与 2.7 一致：Makefile 不带 `-s settings.xml`，Maven 默认读取 `~/.m2/settings.xml`。

## 5. 变更影响分析（Impact Analysis）

| 受影响对象 | 变更 | 风险 | 缓解 |
|-----------|------|------|------|
| 业务源码 | **无改动** | 无 | 官方基线已修复三 CVE |
| 测试代码 | 可能补充固化用例 | 低 | 仅新增断言，不改主逻辑 |
| `pom.xml` GAV | 坐标重命名 | 下游解析失败 | GAV_MAPPING 文档 + 保留包名 |
| parent 引用 | 版本 3.5.14-SNAPSHOT → 3.5.13 | 私服解析父 POM | 方案 A，`.m2` 已确认有 3.5.13 |
| `pom.xml` 私服 | 新增 distributionManagement | 发布链路 | `clean deploy` 干跑验证 |
| 下游依赖方 | 需换 GAV | 构建中断 | 文档说明 + 版本对照表 |

## 6. 测试策略

- **验证优先**：三个本体 CVE 官方已修复，本次以"确认既有防护测试存在并通过"为主。
- **回归**：`./mvnw clean test` 全量通过（web + mapping + util 三个受影响包）。
- **构建验证**：`./mvnw clean install` 从私服解析（含 parent `3.5.13`）成功；`./mvnw -DskipTests clean deploy` 发布 SNAPSHOT 到私服成功。
- **覆盖率**：本次核心为文档与构建配置，无新增业务逻辑；若补充测试用例，保证针对性覆盖三个 CVE 防护点。

## 7. 决策记录（已拍板，2026-07-09）

1. **本体 CVE**：✅ 官方已在 3.5.12/3.5.13 修复，本次**不改业务代码**，仅验证 + 文档化（用户明确："官方已修复，不必画蛇添足"）。
2. **CVE 盘点范围**：✅ **仅本体三 CVE**。传递依赖（spring-core/beans、jackson、guava 等）已在 spring / spring-boot 的 GAV NES 处理，本项目不重复盘点（用户明确）。
3. **Change 粒度**：✅ 单个 `nes-patch-2026` 大 change 装 4 能力（用户："看着办" + 单个大 change）。
4. **GAV 去特征化**：✅ **必做**（用户明确）。groupId/artifactId 采用 `cn.bjca.footstone.bpring.data` / `bjca-footstone-bpring-data-commons`；version `3.5.13-nes.patch.1-SNAPSHOT`。
5. **Parent 处理**：✅ 方案 A，版本锁 `3.5.13`（本地 `.m2` 已确认可解析）。
