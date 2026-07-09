# 快速入门 (Quick Start)

## 环境要求

| 项 | 要求 |
|----|------|
| JDK | Java 8+（构建产物字节码为 Java 8，本地可用 JDK 17） |
| Maven | 3.6+（项目自带 `mvnw` 3.9.5） |
| 私服 | 内网 Nexus `http://192.168.131.36:8088`（依赖解析与发布） |
| settings | `~/.m2/settings.xml` 提供 `nexus` mirror、`bjca` profile（含 `nexusReleaseUrl` / `nexusSnapshotUrl` 属性，默认激活）及 `snapshots` 发布凭证。命令无需 `-s`，Maven 默认读取 |

## 三步上手

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
./mvnw -DskipTests clean install
```

## 使用 Makefile（推荐）

```bash
make build      # 编译打包（跳过测试）
make test       # 运行测试
make install    # 安装到本地仓库
make deploy     # 发布到 Nexus 私服
```

## 引入依赖

```xml
<dependency>
    <groupId>cn.bjca.footstone.bpring.data</groupId>
    <artifactId>bjca-footstone-bpring-data-commons</artifactId>
    <version>2.7.18-nes.patch.1-SNAPSHOT</version>
</dependency>
```

> Java 包名保持 `org.springframework.data.*` 不变，源代码 `import` 无需修改。

## 更多文档

- [用户手册](USER_MANUAL.md)
- [GAV 映射表](GAV_MAPPING.md)
- [漏洞状态总览](VULNERABILITY_REPORT.md)
- [需求与版本清单](REQUIREMENTS.md)
