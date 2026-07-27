# 快速入门 (Quick Start)

## 环境要求

| 项 | 要求 |
|----|------|
| JDK | Java 17（3.5.x 基线要求；本地 sdkman 可用 `17.0.17-amzn`） |
| Maven | 3.9+（项目自带 `mvnw` 3.9.16） |
| 私服 | 内网 Nexus `http://192.168.131.36:8088`（依赖解析与发布） |
| settings | `~/.m2/settings.xml` 提供 `nexus` mirror、`bjca` profile（含 `nexusReleaseUrl` / `nexusSnapshotUrl` 属性，默认激活）及 `releases` / `snapshots` 发布凭证。命令无需 `-s`，Maven 默认读取 |

## RELEASE 坐标

```xml
<dependency>
    <groupId>cn.bjca.footstone.bpring.data</groupId>
    <artifactId>bjca-footstone-bpring-data-commons</artifactId>
    <version>3.5.13-nes.patch.1</version>
</dependency>
```

必需的内部上游是
`cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:6.2.19-nes.patch.1`，
已由中央发布流程完成 Nexus RELEASE 验证。本组件没有发布排除项。

目标仓库：`http://192.168.131.36:8088/repository/releases`。

## 日常开发命令

### 1. 编译

```bash
./mvnw -DskipTests clean package
```

### 2. 运行测试

```bash
./mvnw test
```

### 3. 安装到本地仓库

```bash
./mvnw -DskipTests install
```

## 使用 Makefile（推荐）

```bash
make build      # 编译打包（跳过测试）
make test       # 运行测试
make install    # 安装到本地仓库
make deploy     # 发布到 Nexus 私服
```

## RELEASE 验证约束

本次 RELEASE 经操作人批准，不重复执行 `make build` 和 `make test`：
发布差异只允许包含版本元数据、文档和 OpenSpec 证据，不包含生产或测试源码。
正式部署前，主会话仍必须在全局构建锁下串行执行增量本地安装：

```bash
JAVA_HOME=/Users/anan/.sdkman/candidates/java/17.0.17-amzn \
JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8 \
MAVEN_OPTS='-Xmx4g -Dfile.encoding=UTF-8' \
./mvnw -DskipTests install
```

随后必须枚举本地实际制品、扫描所有生成 POM 中的内部 SNAPSHOT，
并完成代表性 consumer 验证。`make deploy` 只允许主会话在 Nexus 目标版本
不存在性检查通过后执行一次。

> Java 包名保持 `org.springframework.data.*` 不变，源代码 `import` 无需修改。

## 更多文档

- [用户手册](USER_MANUAL.md)
- [GAV 映射表](GAV_MAPPING.md)
- [漏洞状态总览](VULNERABILITY_REPORT.md)
- [需求与版本清单](REQUIREMENTS.md)
