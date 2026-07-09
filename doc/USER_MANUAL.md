# 用户手册 (User Manual)

## 1. 项目定位

本项目是 **Spring Data Commons 3.5.x 的 NES（Never-Ending Support）fork**，由 BJCA 维护分支管理。其核心目标：

1. **安全基线对齐**：本库自身携带的三个本体 CVE，官方已在 3.5.12 修复，本基线 3.5.13 已包含；本 fork 验证修复生效并文档留痕（**不改动业务代码**）。
2. **GAV 去特征化**：重命名 Maven 坐标，规避 SCA 工具误报。
3. **私服隔离**：依赖解析与制品发布统一走内网 Nexus。

本项目与官方 3.5.13 功能对齐，仅在坐标与发布渠道上有差异；**源码不含任何本 fork 私有改动**。

## 2. 与官方版本的差异

| 维度 | 官方 3.5.13 | 本 NES fork |
|------|------------|-------------|
| GAV | `org.springframework.data:spring-data-commons:3.5.13` | `cn.bjca.footstone.bpring.data:bjca-footstone-bpring-data-commons:3.5.13-nes.patch.1-SNAPSHOT` |
| 上游 spring 依赖 | `org.springframework:spring-*` | `cn.bjca.footstone.bpring:bjca-footstone-bpring-*`（6.2.19-nes.patch.1，经 fork BOM） |
| CVE-2026-41721 | ✅ 已修复（3.5.12） | ✅ 继承官方修复 |
| CVE-2026-41716 | ✅ 已修复（3.5.12） | ✅ 继承官方修复 |
| CVE-2026-41711 | ✅ 已修复（3.5.12） | ✅ 继承官方修复 |
| Java 包名 | `org.springframework.data.*` | 不变 |
| 业务源码 | — | 完全一致（零私有改动） |

详见 [GAV 映射表](GAV_MAPPING.md) 与 [漏洞状态总览](VULNERABILITY_REPORT.md)。

> **与 spring-data-commons-2.7 fork 的区别**：2.7.x 属 EOL、官方无补丁，2.7 fork 需自行 backport 三个 CVE；而 3.5.x 是活跃线、官方已在 3.5.12 修复，本 fork 直接继承官方修复，不做 backport。

## 3. 安全修复说明

三个本体 DoS 漏洞均由**官方 upstream 在 3.5.12 修复**，本基线 3.5.13 已包含。本 fork 运行回归测试确认防护生效，对正常业务无影响：

### 3.1 CVE-2026-41721 — MapDataBinder 数据绑定 DoS

`@ProjectedPayload` 端点绑定表单参数时，SpEL 集合自动增长无上限，可被 `fooBar[100000000]` 类参数触发内存耗尽。官方修复：为 SpEL 设置 `maximumAutoGrowSize=1024` 上限（`MapDataBinder.DEFAULT_COLLECTION_LIMIT`）。正常集合绑定（索引 < 1024）不受影响。

### 3.2 CVE-2026-41716 — 无界负结果缓存 DoS

`TypeDiscoverer` 把每个查询过的属性名（含不存在的）永久缓存，可被大量伪造 property 名撑爆堆内存。官方修复：改为一次性初始化类型属性，消除无界 miss 缓存。

### 3.3 CVE-2026-41711 — PropertyPath 解析栈溢出 DoS

超长 camelCase 属性名触发 `PropertyPath` 递归栈溢出（绕过旧的点分深度防护）。官方修复：以 `remainingDepth` 计数器统一约束点分与 camel-case 两条递归，上限 1000。正常属性路径不受影响。

## 4. 构建与发布

参见 [快速入门](QUICK_START.md)。常用命令：

```bash
make build      # 编译打包
make test       # 运行测试
make install    # 安装到本地仓库
make deploy     # 发布到 Nexus 私服
```

## 5. 私服配置

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

`${nexusReleaseUrl}` / `${nexusSnapshotUrl}` 及凭证 `releases` / `snapshots` 由 `~/.m2/settings.xml` 的 `bjca` profile（默认激活）提供。
