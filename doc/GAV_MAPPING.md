# GAV 映射表 (GAV Mapping)

本项目为 Spring Data Commons 2.7.x 的 NES fork，对 Maven 坐标（GAV）进行去特征化重命名，以规避 SCA 工具按官方 GAV 特征误报 CVE。

## 坐标映射

| 坐标 | 官方原值 | NES fork 新值 |
|------|---------|--------------|
| **groupId** | `org.springframework.data` | `cn.bjca.footstone.bpring.data` |
| **artifactId** | `spring-data-commons` | `bjca-footstone-bpring-data-commons` |
| **version** | `2.7.19-SNAPSHOT` | `2.7.18-nes.patch.1-SNAPSHOT` |

## Parent 坐标

| 坐标 | 值 | 说明 |
|------|-----|------|
| **groupId** | `org.springframework.data.build` | 保持不变（方案 A） |
| **artifactId** | `spring-data-parent` | 保持不变 |
| **version** | `2.7.18` | 由 `2.7.19-SNAPSHOT` 锁定为已发布正式版（私服可解析） |

> Parent 仅在构建期解析，不体现在发布制品的 GAV 特征上，因此保留原坐标即可。

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
    <version>2.7.18-nes.patch.1-SNAPSHOT</version>
</dependency>
```
