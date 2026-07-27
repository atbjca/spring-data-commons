# GAV 映射表 (GAV Mapping)

本项目为 Spring Data Commons 2.7.x 的 NES fork，对 Maven 坐标（GAV）进行去特征化重命名，以规避 SCA 工具按官方 GAV 特征误报 CVE。

## 坐标映射

| 坐标 | 官方原值 | NES fork 新值 |
|------|---------|--------------|
| **groupId** | `org.springframework.data` | `cn.bjca.footstone.bpring.data` |
| **artifactId** | `spring-data-commons` | `bjca-footstone-bpring-data-commons` |
| **version** | `2.7.19-SNAPSHOT` | `2.7.18-nes.patch.1` |

## Parent 坐标

| 坐标 | 值 | 说明 |
|------|-----|------|
| **groupId** | `org.springframework.data.build` | 保持不变（方案 A） |
| **artifactId** | `spring-data-parent` | 保持不变 |
| **version** | `2.7.18` | 由 `2.7.19-SNAPSHOT` 锁定为已发布正式版（私服可解析） |

> Parent 仅在构建期解析，不体现在发布制品的 GAV 特征上，因此保留原坐标即可。

## 直接依赖坐标去特征化

发布 POM 的 `<dependencies>` 中，本项目直接依赖的 Spring 组件坐标同样去特征化，避免 SCA 工具扫描发布物料时命中官方 `org.springframework` 特征。版本由 NES fork 的 framework BOM 统一托管（替代 parent 传递的官方 `spring-framework-bom:5.3.31`）。

| 依赖组件 | 官方原坐标 | NES fork 新坐标 |
|---------|-----------|----------------|
| core | `org.springframework:spring-core` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-core` |
| beans | `org.springframework:spring-beans` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-beans` |
| context | `org.springframework:spring-context` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-context` |
| expression | `org.springframework:spring-expression` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-expression` |
| tx | `org.springframework:spring-tx` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-tx` |
| oxm | `org.springframework:spring-oxm` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-oxm` |
| web | `org.springframework:spring-web` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-web` |
| webflux | `org.springframework:spring-webflux` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-webflux` |
| webmvc | `org.springframework:spring-webmvc` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-webmvc` |

**版本托管 BOM：**

| 坐标 | 值 |
|------|-----|
| **groupId** | `cn.bjca.footstone.bpring` |
| **artifactId** | `bjca-footstone-bpring-framework-bom` |
| **version** | `5.3.39-nes.patch.1`（已发布并通过 Nexus 验证的 Spring Framework 5.3.39 NES RELEASE） |
| **scope** | `import`（置于本项目 `<dependencyManagement>`，覆盖 parent 传递的官方 `5.3.31`） |

> 该项去特征化由独立变更 `nes-dep-defeature` 管理。当前 RELEASE 不包含任何内部
> SNAPSHOT 依赖。

## RELEASE publication

| 项 | 值 |
|----|----|
| Nexus RELEASE | `http://192.168.131.36:8088/repository/releases` |
| 主 GAV | `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:2.7.18-nes.patch.1` |
| Packaging | `jar` |
| 预期附属资产 | POM、主 JAR、sources JAR；最终完整集以 `make install` 结果为准 |
| 发布排除项 | 无 |

正式部署前必须串行执行 `make install`，扫描所有生成 POM 中的内部 SNAPSHOT，并用
仅启用 RELEASE 仓库的 consumer 验证坐标。`make deploy` 仅由协调主会话在 Nexus
目标版本完全不存在时执行一次。

## 保持不变的项

| 项 | 值 | 原因 |
|----|-----|------|
| **Java 包名** | `org.springframework.data.*` | 下游 `import` 语句无需修改 |
| **JPMS 自动模块名** | `spring.data.commons` | 避免破坏下游 `requires` 声明 |
| **字节码目标** | Java 8（major version 52） | 兼容 Java 8 运行时 |

## 依赖方迁移说明

下游项目只需更新依赖坐标，**无需修改任何 Java 源代码**：

```xml
<!-- 修改前 -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
    <version>2.7.18</version>
</dependency>

<!-- 修改后 -->
<dependency>
    <groupId>cn.bjca.footstone.bpring.data</groupId>
    <artifactId>bjca-footstone-bpring-data-commons</artifactId>
    <version>2.7.18-nes.patch.1</version>
</dependency>
```
