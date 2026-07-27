# 快速入门 (Quick Start)

## 环境要求

| 项 | 要求 |
|----|------|
| JDK | Java 8+（构建产物字节码为 Java 8，本地可用 JDK 17） |
| Maven | 3.6+（项目自带 `mvnw` 3.9.5） |
| 私服 | 内网 Nexus `http://192.168.131.36:8088`（依赖解析与发布） |
| settings | `~/.m2/settings.xml` 提供 `nexus` mirror、`bjca` profile（含 `nexusReleaseUrl` / `nexusSnapshotUrl` 属性，默认激活）及 `snapshots` 发布凭证。命令无需 `-s`，Maven 默认读取 |

## 引入 RELEASE 制品

Nexus RELEASE 仓库地址为 `http://192.168.131.36:8088/repository/releases`。

```xml
<dependency>
    <groupId>cn.bjca.footstone.bpring.data</groupId>
    <artifactId>bjca-footstone-bpring-data-commons</artifactId>
    <version>2.7.18-nes.patch.1</version>
</dependency>
```

> Java 包名保持 `org.springframework.data.*` 不变，源代码 `import` 无需修改。

该 RELEASE 依赖已经发布并验证的 Framework BOM：
`cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1`。
本组件没有发布排除项，Java 8 字节码和 `spring.data.commons` 自动模块名保持不变。

## 发布验证

开发阶段已有两轮可复用验证：安全补丁变更的全量 `3328` tests 通过，以及切换
NES Framework 依赖后的全量回归通过。RELEASE 变更仅允许修改版本、内部 RELEASE
依赖和发布文档；经 release-only diff 审计确认无源码/测试变化后，获准不重复执行
`make build` 和 `make test`。

正式部署前仍须由协调主会话串行执行：

```bash
make install
```

该命令展开为 `./mvnw -DskipTests clean install`。完成后必须核对本地 publication
完整集，扫描全部生成 POM 中的内部 SNAPSHOT 引用，并运行只使用 RELEASE 仓库的
consumer 验证。确认 Nexus 中目标版本完全不存在后，协调主会话才可执行一次
`make deploy`；不得由组件准备会话部署或重发同一 RELEASE。

## 日常开发命令

```bash
make build      # 编译打包（跳过测试）
make test       # 运行测试
make install    # 安装到本地仓库
```

## 更多文档

- [用户手册](USER_MANUAL.md)
- [GAV 映射表](GAV_MAPPING.md)
- [漏洞状态总览](VULNERABILITY_REPORT.md)
- [需求与版本清单](REQUIREMENTS.md)
