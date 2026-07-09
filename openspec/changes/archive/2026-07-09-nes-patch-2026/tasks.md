# Tasks — nes-patch-2026 (spring-data-commons 3.5)

> 遵循"变更前确认"原则。**核心特征：三个本体 CVE 官方已修复，本次零业务代码改动。**
> 图例：`[ ]` 未开始 `[~]` 进行中 `[x]` 完成

## 阶段 0：准备与确认

- [x] 0.1 确认 git 分支状态干净、已备份（当前分支 `3.5.x-bjca-patch`，工作区仅 `.claude/`、`openspec/` 未跟踪）
- [x] 0.2 用 sdkman 选定并验证 JDK 17（`17.0.17-amzn`），`./mvnw -v` 可运行（Maven 3.9.16）
- [x] 0.3 **前置验证**：本地 `.m2` 已确认存在 `spring-data-parent-3.5.13.pom`，parent 可解析

> 决策已定（见 design 第 7 节）：本体 CVE 官方已修复、零源码改动；GAV 采用 `cn.bjca.footstone.bpring.data` / `bjca-footstone-bpring-data-commons`；version `3.5.13-nes.patch.1-SNAPSHOT`；parent 方案 A（锁 `3.5.13`）；仅盘本体三 CVE。

## 阶段 1：本体 CVE 验证（零源码改动）

### 1A CVE-2026-41721 — MapDataBinder
- [x] 1A.1 核对 `MapDataBinder.java:124` 已用 `SpelParserConfiguration(false, true, DEFAULT_COLLECTION_LIMIT=1024)` 限制自动增长
- [x] 1A.2 `MapDataBinderUnitTests` 含 `rejectsValuesBeyondConfiguredMaxCollectionSize` 边界用例并通过（8/8）

### 1B CVE-2026-41711 — PropertyPath
- [x] 1B.1 核对 `PropertyPath.java:449-493` `remainingDepth` 计数器统一约束两条递归路径（上限 `MAX_PARSE_DEPTH=1000`）
- [x] 1B.2 `PropertyPathUnitTests` 含路径段数 > 1000 抛 `IllegalArgumentException` 用例并通过（48/48）

### 1C CVE-2026-41716 — TypeDiscoverer
- [x] 1C.1 核对 `TypeDiscoverer` 一次性初始化、无 miss 无界缓存（官方 `e71100a8b`(3.5.12)/`accc38344`(3.5.13)）
- [x] 1C.2 `TypeDiscovererUnitTests`（含 `ofCached` 缓存路径）通过（33/33）

### 1D 整体验证
- [x] 1D.1 三 CVE 测试类合计 89 tests 全绿；全量 `clean install` 3630 tests（Failures 0/Errors 0/Skipped 7）无回归

## 阶段 2：GAV 去特征化

- [x] 2.1 `pom.xml`：groupId → `cn.bjca.footstone.bpring.data`
- [x] 2.2 `pom.xml`：artifactId → `bjca-footstone-bpring-data-commons`
- [x] 2.3 `pom.xml`：version → `3.5.13-nes.patch.1-SNAPSHOT`
- [x] 2.4 parent 方案 A：`spring-data-parent` 版本由 `3.5.14-SNAPSHOT` 改为 `3.5.13`（已发布正式版）
- [x] 2.5 确认 `java-module-name` 仍为 `spring.data.commons`（不变）
- [x] 2.6 **上游依赖替换**：`<dependencyManagement>` 导入 `bjca-footstone-bpring-framework-bom:6.2.19-nes.patch.1-SNAPSHOT`
- [x] 2.7 **上游依赖替换**：10 个 spring 依赖（core/beans/context/expression/tx/oxm/web/webflux/webmvc/core-test）→ `bjca-footstone-bpring-*`；`spring-hateoas` 保留官方
- [x] 2.8 验证 `./mvnw clean test` 全量通过（3630 tests，Failures 0 / Errors 0 / Skipped 7），包名 `org.springframework.data.*` 不变

## 阶段 3：私服配置

- [x] 3.1 `pom.xml` 增加 `<distributionManagement>`（releases/snapshots + `${nexusReleaseUrl/nexusSnapshotUrl}`）
- [x] 3.2 确认私服属性/凭证/mirror 由 `~/.m2/settings.xml` 的 `bjca` profile（默认激活）提供；Makefile 不带 `-s settings.xml`
- [x] 3.3 `clean deploy` 真实发布 —— **用户已手动执行 `make deploy` 完成，制品发布至 snapshots 私服**

## 阶段 4：CVE 文档体系

- [x] 4.1 `doc/CVE/CVE-2026-41721.md`（本体，✅已修复，修复方=官方 upstream `e33b68517`@3.5.12）
- [x] 4.2 `doc/CVE/CVE-2026-41711.md`（本体，✅已修复，修复方=官方 upstream `c3b2abf29`@3.5.12）
- [x] 4.3 `doc/CVE/CVE-2026-41716.md`（本体，✅已修复，修复方=官方 upstream `e71100a8b`@3.5.12）
- [x] 4.4 `doc/VULNERABILITY_REPORT.md`：6 态归一化总览 + 统计 + 索引；明确"官方基线已修复，本 fork 仅验证"
- [x] 4.5 校验 CVE 文档链接与状态一致（3/3 OK）；措辞未照抄 2.7 的 backport 表述

## 阶段 5：交付文档

- [x] 5.1 `doc/GAV_MAPPING.md`：GAV 映射表
- [x] 5.2 `doc/QUICK_START.md`：快速入门（新 GAV 坐标、默认 settings、Java 17）
- [x] 5.3 `doc/USER_MANUAL.md`：用户手册（fork 定位、GAV 变化、与官方差异）
- [x] 5.4 `doc/REQUIREMENTS.md`：需求与版本清单
- [x] 5.5 `Makefile`：build / test / install / deploy 快捷命令

## 阶段 6：收尾

- [x] 6.1 `openspec validate nes-patch-2026 --strict` 通过
- [x] 6.2 构建 + 测试最终验证（`make clean test` 3630 全绿；`make deploy` 用户已执行发布成功）
- [x] 6.3 更新 VULNERABILITY_REPORT 最终状态（三本体 CVE 均 ✅已修复 / 官方 upstream）
- [x] 6.4 归档 change 到 `openspec/changes/archive/`
