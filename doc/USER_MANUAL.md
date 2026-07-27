# 用户手册 (User Manual)

## 1. 项目定位

本项目是 **Spring Data Commons 2.7.x 的 NES（Never-Ending Support）fork**，由 BJCA 维护分支管理。其核心目标：

1. **本体安全修复**：修复本库自身携带、官方在 2.7.x（EOL）线不再提供开源补丁的 CVE。
2. **GAV 去特征化**：重命名 Maven 坐标，规避 SCA 工具误报。
3. **私服隔离**：依赖解析与制品发布统一走内网 Nexus。

本项目与官方 2.7.18 功能对齐，仅在安全修复与坐标上有差异。

## 2. 与官方版本的差异

| 维度 | 官方 2.7.18 | 本 NES fork |
|------|------------|-------------|
| GAV | `org.springframework.data:spring-data-commons:2.7.18` | `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:2.7.18-nes.patch.1` |
| CVE-2026-41721 | 未修复（EOL） | ✅ 已修复 |
| CVE-2026-41716 | 未修复（EOL） | ✅ 已修复 |
| CVE-2026-41711 | 未修复（EOL） | ✅ 已修复 |
| Java 包名 | `org.springframework.data.*` | 不变 |
| API 兼容性 | — | 完全兼容 |

详见 [GAV 映射表](GAV_MAPPING.md) 与 [漏洞状态总览](VULNERABILITY_REPORT.md)。

## 3. 安全修复说明

本 fork 修复了三个本体 DoS 漏洞，均遵循 TDD（先失败测试，再实现），行为对正常业务无影响：

### 3.1 CVE-2026-41721 — MapDataBinder 数据绑定 DoS

`@ProjectedPayload` 端点绑定表单参数时，SpEL 集合自动增长无上限，可被 `fooBar[100000000]` 类参数触发内存耗尽。修复：为 SpEL 设置 `maximumAutoGrowSize=256` 上限。正常集合绑定（索引 < 256）不受影响。

### 3.2 CVE-2026-41716 — 无界负结果缓存 DoS

`TypeDiscoverer` 把每个查询过的属性名（含不存在的）永久缓存，可被大量伪造 property 名撑爆堆内存。修复：仅缓存真实存在的属性，不缓存 miss。

### 3.3 CVE-2026-41711 — PropertyPath 解析栈溢出 DoS

超长 camelCase 属性名触发 `PropertyPath` 递归栈溢出（绕过旧的点分深度防护）。修复：为 camel-case 递归增加深度上限 1000。正常属性路径不受影响。

## 4. RELEASE 依赖与发布

发布坐标：

```xml
<dependency>
    <groupId>cn.bjca.footstone.bpring.data</groupId>
    <artifactId>bjca-footstone-bpring-data-commons</artifactId>
    <version>2.7.18-nes.patch.1</version>
</dependency>
```

Nexus RELEASE 地址：`http://192.168.131.36:8088/repository/releases`。

内部依赖由已发布的
`cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1`
统一托管。直接引用的 core、beans、context、expression、tx、oxm、web、webflux
和 webmvc 均必须解析为同一 RELEASE 版本，不允许内部 SNAPSHOT。

本组件无发布排除项。其父 POM 仍为公开 RELEASE
`org.springframework.data.build:spring-data-parent:2.7.18`；Java 包名、自动模块名和
Java 8 字节码兼容性保持不变。

## 5. RELEASE 验证流程

既有开发证据包含全量 `3328` tests 通过，以及切换 NES Framework 依赖后的全量
回归通过。若 release-only diff 仅含版本、内部 RELEASE 依赖、文档和 OpenSpec，
且没有生产/测试源码变化，则本次发布获准不重复执行 `make build` / `make test`。

部署前仍必须由协调主会话串行执行本地安装：

```bash
make install
```

随后核对 POM、主 JAR、sources JAR 等实际 publication 资产，扫描全部 POM 的内部
SNAPSHOT 引用，并运行 RELEASE-only consumer。仅当所有目标资产在 Nexus RELEASE
中均不存在时，协调主会话才可执行一次 `make deploy`。已存在或部分存在的 RELEASE
不得覆盖或重发。

## 6. 私服配置

`pom.xml` 的 `<distributionManagement>` 使用属性占位：

```xml
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>${nexusReleaseUrl}</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>${nexusSnapshotUrl}</url>
    </snapshotRepository>
</distributionManagement>
```

`${nexusReleaseUrl}` / `${nexusSnapshotUrl}` 及凭证 `releases` / `snapshots` 由 `~/.m2/settings.xml` 提供。
