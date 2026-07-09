# Design — nes-patch-2026

> 本文档记录 spring-data-commons-2.7 NES fork 改造的技术设计与关键决策。
> 遵循 TDD：所有本体 CVE 修复先写触发用例，再改业务代码。

## 1. 背景与约束

| 项 | 值 |
|----|-----|
| 基线版本 | `2.7.19-SNAPSHOT`（parent 原引用 `spring-data-parent:2.7.19-SNAPSHOT`；改造后锁定已发布正式版 `2.7.18` 为基线与 parent 版本） |
| 目标版本 | `2.7.18-nes.patch.1-SNAPSHOT` |
| 构建工具 | Maven（`mvnw` 3.9.5，本项目自带 wrapper） |
| 私服 | `http://192.168.131.36:8088`（release `/repository/releases/`，snapshot `/repository/snapshots/`，聚合 `/repository/maven-public/`） |
| Git 分支 | `2.7.x-bjca-patch` |
| Java | **硬约束：产物必须兼容 Java 8**。parent 2.7.18 `source.level=1.8`，项目本为 Java 8 基线。修复/测试代码只用 Java 8 API 与语法（禁 var、List.of、Map.of 等 Java 9+）。本地开发 JDK 17（Corretto），编译目标 1.8 |

**核心约束**：官方对 CVE-2026-41711 / 41716 / 41721 仅在 3.5.12 / 4.0.6 修复，**2.7.x 属 EOL，无开源补丁**。本 fork 即承担 NES 角色，将官方补丁 backport 到 2.7.x。

## 2. 本体 CVE 修复方案（核心）

### 2.1 攻击面总览

```
   HTTP 请求（不可信输入）
        │
        ├── ?sort=a.b.c.d... ──────────► SortHandlerMethodArgumentResolver
        │                                  └─► PropertyPath.from() ──► 递归无深度限制
        │                                                              【CVE-2026-41711 StackOverflow】
        │
        ├── @ProjectedPayload 表单参数 ──► ProxyingHandlerMethodArgumentResolver
        │       ?list[99999999]=x          └─► MapDataBinder ─► SpEL autoGrow 无上限
        │                                                        【CVE-2026-41721 堆耗尽】
        │
        └── 大量不同 property 名 ─────────► PropertyPath.from / getProperty
                ?p1=&p2=&p3=...              └─► TypeDiscoverer.fieldTypes 无界缓存
                                                 【CVE-2026-41716 堆耗尽 OOM】
```

三个 CVE 共享同一条数据流：**HTTP 参数 → PropertyPath → TypeDiscoverer**。这是本库 web 支持的固有攻击面，`@ProjectedPayload` / `@QuerydslPredicate` / `Sort` 端点是三个入口。

### 2.2 逐个修复设计

#### CVE-2026-41721 — MapDataBinder SpEL 无界自动增长

- **根因**：`MapDataBinder.java:113-114`
  ```java
  private static final SpelExpressionParser PARSER = new SpelExpressionParser(
          new SpelParserConfiguration(false, true)); // autoGrowCollections=true, 无 maximumAutoGrowSize
  ```
- **官方修复方向**：`SpelParserConfiguration` 有四参构造器 `(SpelCompilerMode, ClassLoader, boolean autoGrowNullReferences, boolean autoGrowCollections, int maximumAutoGrowSize)`。为 `maximumAutoGrowSize` 设一个合理上限（官方 3.5.12 采用固定上限，如 256）。
- **TDD**：在 `MapDataBinderUnitTests` 增加用例 —— 绑定 `list[100000]` 形式的属性名，**期望抛出可控异常而非分配海量内存**（可用超时或断言异常类型）。
- **兼容性**：正常 `@ProjectedPayload` 绑定的集合索引远小于上限，不受影响。现有测试须全绿。

#### CVE-2026-41711 — PropertyPath 递归无深度限制

- **根因**：`PropertyPath.java:344 from(String, TypeInformation)` 及其递归 `create(...)`，对超长点分路径 `a.b.c.d...` 无深度上限 → StackOverflow。
- **官方修复方向**：在解析入口对属性路径的**段数 / 总长度**设上限（官方引入常量上限，如路径段数 ≤ 1000 或长度阈值），超限抛 `PropertyReferenceException` 或专用异常。
- **TDD**：`PropertyPathUnitTests`（若无则新建）增加用例 —— 构造 N 层深的点分路径，**期望抛受控异常而非 StackOverflowError**。
- **风险**：`PropertyPath.from` 被全库广泛使用，上限须足够大以不影响正常领域模型（真实实体嵌套极少 > 10 层），仅拦截攻击级输入。

#### CVE-2026-41716 — TypeDiscoverer 无界缓存

- **根因**：`TypeDiscoverer.java:60` `fieldTypes = new ConcurrentHashMap<>()` + `:189 computeIfAbsent`，把每个查询过的（含不存在的）property 名永久驻留。
- **官方修复方向**：将无界缓存替换为**有界缓存**（官方采用容量受限结构，如基于大小上限的 LRU / 或仅缓存"存在"的 property、不缓存 miss）。需核对 3.5.12 实际实现后采用等价方案。
- **TDD**：`TypeDiscovererUnitTests`（若无则新建）—— 查询大量不同的不存在 property 名，**断言缓存大小不无限增长**（反射/暴露测试钩子读取 `fieldTypes.size()`）。
- **风险**：缓存策略变更可能影响性能。需保证正常 property 解析仍命中缓存。**本期完整修复（决策 3），不走缓解降级。**

### 2.3 状态映射（初判，实现后回填）

| CVE | 攻击面存在 | 计划状态 |
|-----|:---:|:---:|
| CVE-2026-41721 | ✅ MapDataBinder | 🔧修复中 → ✅已修复 |
| CVE-2026-41711 | ✅ PropertyPath | 🔧修复中 → ✅已修复 |
| CVE-2026-41716 | ✅ TypeDiscoverer | 🔧修复中 → ✅已修复（本期完整修复，决策 3） |

> **41717 / 41719**（同批披露）：本期**不纳入**（决策 4，仅聚焦本体三 CVE）。若后续核实涉及本库，另起 change 处理。

## 3. GAV 去特征化设计

| 坐标 | 原值 | 新值 |
|------|------|------|
| groupId | `org.springframework.data` | `cn.bjca.footstone.bpring.data` |
| artifactId | `spring-data-commons` | `bjca-footstone-bpring-data-commons` |
| version | `2.7.19-SNAPSHOT` | `2.7.18-nes.patch.1-SNAPSHOT` |
| java-module-name | `spring.data.commons` | 保持不变（JPMS 模块名，改动影响下游 `requires`） |

**Parent 处理 —— 已定：方案 A（保留原 parent 坐标，版本锁定 2.7.18 正式版）**。

```
✅ 方案 A：保留原 parent 坐标      ── 最小改动，parent 仅构建期依赖、不进制品 GAV 特征
   方案 B：同步 fork parent 去特征  ──（未采用）工作量翻倍
   方案 C：扁平化去 parent          ──（未采用）
```

**关键修正（用户澄清）**：原 `pom.xml` 的 parent 版本是 `2.7.19-SNAPSHOT`（未发布快照，私服未必有），必须改为 **`2.7.18`（已发布正式版，私服可解析）**：

```xml
<parent>
  <groupId>org.springframework.data.build</groupId>
  <artifactId>spring-data-parent</artifactId>
  <version>2.7.18</version>   <!-- 原为 2.7.19-SNAPSHOT，改为已发布正式版 -->
</parent>
```

理由：
- parent 只在构建期解析，不体现在发布制品的 GAV 特征上，SCA 工具扫描的是制品自身坐标（已去特征化）。
- 引用已发布的 `2.7.18` 正式版，彻底消除 SNAPSHOT 父 POM 在私服解析不到的风险。
- 与制品版本 `2.7.18-nes.patch.1-SNAPSHOT` 的基线（2.7.18）一致。

**包名（package）不改**：`org.springframework.data.*` 保持，故下游 `import` 无需改动——与 kafka fork 一致。

## 4. 私服配置设计

`pom.xml` 增加（用户已指定形态）：
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
`${nexusReleaseUrl}` / `${nexusSnapshotUrl}` 由 `~/.m2/settings.xml` 的 profile 提供（已存在），`settings.xml`（项目内）对齐 server 凭证 id = `releases` / `snapshots`。

## 5. 变更影响分析（Impact Analysis）

| 受影响对象 | 变更 | 风险 | 缓解 |
|-----------|------|------|------|
| `MapDataBinder` | SpEL 加上限 | 破坏正常集合绑定 | 上限足够大 + 现有测试全绿 |
| `PropertyPath` | 解析加深度限 | 影响全库属性解析 | 上限远超真实模型深度 |
| `TypeDiscoverer` | 缓存改有界 | 性能/命中率 | 保留正常命中，仅界定上限 |
| `pom.xml` GAV | 坐标重命名 | 下游解析失败 | GAV_MAPPING 文档 + 保留包名 |
| parent 引用 | 版本 2.7.19-SNAPSHOT → 2.7.18 | 私服解析父 POM | 方案 A 已定，锁正式版 2.7.18（私服可解析） |
| 下游依赖方 | 需换 GAV | 构建中断 | 文档说明 + 版本对照表 |

## 6. 测试策略（TDD + 覆盖率 ≥60%）

- **红 → 绿**：每个 CVE 先提交失败测试（触发 DoS 的边界输入），再提交修复使其变绿。
- **回归**：`MapDataBinderUnitTests` / `SortHandlerMethodArgumentResolverUnitTests` / `ProxyingHandlerMethodArgumentResolverUnitTests` 全量通过。
- **覆盖率**：核心修复类（3 个）行覆盖 ≥60%，用 `mvn test` + jacoco（若 parent 已配）核验。
- **构建验证**：`./mvnw -s settings.xml clean install` 从私服解析依赖成功。

## 7. 决策记录（已拍板，2026-07-08）

1. **groupId/artifactId 命名**：✅ 采用 `cn.bjca.footstone.bpring.data` / `bjca-footstone-bpring-data-commons`。
2. **Parent 处理**：✅ 方案 A（保留原坐标 `spring-data-parent`），**版本从 `2.7.19-SNAPSHOT` 改为已发布正式版 `2.7.18`**（私服可解析，用户已确认私服有 2.7.18）。
3. **CVE-2026-41716**：✅ 本期**完整修复**（TypeDiscoverer 改有界缓存），不走缓解降级。
4. **传递依赖 CVE**：✅ **不纳入**本期。VULNERABILITY_REPORT 仅聚焦本体 CVE（41721/41711/41716）。若后续需要可另起 change。
