# GAV 映射表 (GAV Mapping)

本项目为 Spring Data Commons 3.5.x 的 NES fork，对 Maven 坐标（GAV）进行去特征化重命名，以规避 SCA 工具按官方 GAV 特征误报 CVE。

## 坐标映射

| 坐标 | 官方原值 | NES fork 新值 |
|------|---------|--------------|
| **groupId** | `org.springframework.data` | `cn.bjca.footstone.bpring.data` |
| **artifactId** | `spring-data-commons` | `bjca-footstone-bpring-data-commons` |
| **version** | `3.5.14-SNAPSHOT` | `3.5.13-nes.patch.1` |

## Parent 坐标

| 坐标 | 值 | 说明 |
|------|-----|------|
| **groupId** | `org.springframework.data.build` | 保持不变（方案 A） |
| **artifactId** | `spring-data-parent` | 保持不变 |
| **version** | `3.5.13` | 由 `3.5.14-SNAPSHOT` 锁定为已发布正式版（私服/本地可解析） |

> Parent 仅在构建期解析，不体现在发布制品的 GAV 特征上，因此保留原坐标即可。

## 上游 spring-framework 依赖替换

本项目不仅重命名自身坐标，还将**上游 spring-framework 依赖**统一指向去特征化的 NES fork（`spring-framework-6.2` 对应基线 6.2.19），通过导入 fork BOM 统一版本。命名规则：`spring-*` → `bjca-footstone-bpring-*`，groupId `org.springframework` → `cn.bjca.footstone.bpring`。

### dependencyManagement 导入的 BOM

| 坐标 | 值 |
|------|-----|
| groupId | `cn.bjca.footstone.bpring` |
| artifactId | `bjca-footstone-bpring-framework-bom` |
| version | `6.2.19-nes.patch.1` |
| type / scope | `pom` / `import` |

### 依赖坐标替换表（10 个）

| 官方原值 | NES fork 新值 |
|---------|--------------|
| `org.springframework:spring-core` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-core` |
| `org.springframework:spring-beans` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-beans` |
| `org.springframework:spring-context` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-context` |
| `org.springframework:spring-expression` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-expression` |
| `org.springframework:spring-tx` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-tx` |
| `org.springframework:spring-oxm` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-oxm` |
| `org.springframework:spring-web` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-web` |
| `org.springframework:spring-webflux` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-webflux` |
| `org.springframework:spring-webmvc` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-webmvc` |
| `org.springframework:spring-core-test`（test） | `cn.bjca.footstone.bpring:bjca-footstone-bpring-core-test` |

> 版本由 BOM 统一管理，依赖声明处不写 `<version>`。Java 包名仍为 `org.springframework.*`，源码 `import` 无影响。

### 保留官方坐标的依赖

| 依赖 | 说明 |
|------|------|
| `org.springframework.hateoas:spring-hateoas` | 无对应 NES fork，保留官方坐标（与 spring-data-commons-2.7 一致） |

## 保持不变的项

| 项 | 值 | 原因 |
|----|-----|------|
| **Java 包名** | `org.springframework.data.*` | 下游 `import` 语句无需修改 |
| **JPMS 自动模块名** | `spring.data.commons` | 避免破坏下游 `requires` 声明 |
| **业务源码** | 与官方 3.5.13 一致 | 三个本体 CVE 官方已修复，无私有改动 |

## RELEASE 发布边界

- 静态预期发布集合为一个 JAR GAV：`cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:3.5.13-nes.patch.1`。
- Maven 可能附加 sources 等分类器；完整资产集合必须在主会话串行执行本地安装后枚举。
- 目标仓库为 `http://192.168.131.36:8088/repository/releases`。
- 发布排除项：无。
- 所有生成 POM 必须不含 `cn.bjca.footstone` 的 SNAPSHOT 依赖、parent、BOM 或插件版本。

## 依赖方迁移说明

下游项目只需更新依赖坐标，**无需修改任何 Java 源代码**：

```xml
<!-- 修改前 -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
    <version>3.5.13</version>
</dependency>

<!-- 修改后 -->
<dependency>
    <groupId>cn.bjca.footstone.bpring.data</groupId>
    <artifactId>bjca-footstone-bpring-data-commons</artifactId>
    <version>3.5.13-nes.patch.1</version>
</dependency>
```
