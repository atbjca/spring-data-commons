# Tasks — nes-patch-2026

> 执行顺序遵循 TDD（先测试后实现）与"变更前确认"原则。每个本体修复任务的测试子项必须先于实现子项完成（红→绿）。
> 图例：`[ ]` 未开始 `[~]` 进行中 `[x]` 完成

## 阶段 0：准备与确认

- [x] 0.1 确认 git 分支状态干净、已备份（当前分支 `2.7.x-bjca-patch`）
- [x] 0.2 用 sdkman 选定并验证 JDK 版本，`./mvnw -v` 可运行
- [x] 0.3 **前置验证**：干跑确认私服可解析父 POM `spring-data-parent:2.7.18`（curl 验证 2.7.18 HTTP 200；parent 版本已从 `2.7.19-SNAPSHOT` 改为 `2.7.18`）

> 决策已定（见 design 第 7 节）：GAV 采用 `cn.bjca.footstone.bpring.data` / `bjca-footstone-bpring-data-commons`；parent 方案 A（版本锁 `2.7.18`）；41716 完整修复；不纳入传递依赖 CVE。

## 阶段 1：本体 CVE 修复（TDD，核心）

### 1A CVE-2026-41721 — MapDataBinder
- [x] 1A.1 【测试先行】`MapDataBinderUnitTests` 增加 `rejectsCollectionIndexExceedingAutoGrowLimit`：绑定 `fooBar[1000]` 超大索引，断言抛 `NotWritablePropertyException`（红）
- [x] 1A.2 【实现】`MapDataBinder` 的 `SpelParserConfiguration` 增加 `maximumAutoGrowSize=256` 上限（绿）
- [x] 1A.3 回归：`MapDataBinderUnitTests`（7）+ web 包全绿
- [x] 1A.4 补充中文注释说明上限值与 CVE 依据

### 1B CVE-2026-41711 — PropertyPath 递归深度
- [x] 1B.1 【测试先行】`PropertyPathUnitTests.rejectsTooDeepCamelCaseRecursion`：构造超长 camelCase 单段路径，断言抛 `IllegalArgumentException` 而非 `StackOverflowError`（红）
- [x] 1B.2 【实现】`PropertyPath.create` camel-case 递归增加 `depth` 计数上限 1000（绿）
- [x] 1B.3 回归：`PropertyPathUnitTests`（44）+ mapping 包全绿
- [x] 1B.4 补充中文注释

### 1C CVE-2026-41716 — TypeDiscoverer 缓存
- [x] 1C.1 【测试先行】`TypeDiscovererUnitTests.doesNotCacheUnresolvedPropertyNames`：查询 1000 个不存在 property 名，反射断言 `fieldTypes` 为空（红）
- [x] 1C.2 【实现】`TypeDiscoverer.getProperty` 单段分支改为不缓存 miss（绿）
- [x] 1C.3 回归：`TypeDiscovererUnitTests`（19）+ util 包全绿
- [x] 1C.4 补充中文注释

### 1D 覆盖率与整体验证
- [x] 1D.1 三个修复类均有专项测试覆盖（红→绿验证通过）
- [x] 1D.2 `./mvnw clean test` 全绿（3328 tests，Failures 0 / Errors 0）

## 阶段 2：GAV 去特征化

- [x] 2.1 `pom.xml`：groupId → `cn.bjca.footstone.bpring.data`
- [x] 2.2 `pom.xml`：artifactId → `bjca-footstone-bpring-data-commons`
- [x] 2.3 `pom.xml`：version → `2.7.18-nes.patch.1-SNAPSHOT`
- [x] 2.4 parent 方案 A：`spring-data-parent` 版本由 `2.7.19-SNAPSHOT` 改为 `2.7.18`（已发布正式版）
- [x] 2.5 验证 `./mvnw clean install` 从私服解析成功
- [x] 2.6 验证发布 POM 中 GAV 正确、包名 `org.springframework.data.*` 不变

## 阶段 3：私服配置

- [x] 3.1 `pom.xml` 增加 `<distributionManagement>`（releases/snapshots + `${nexusReleaseUrl/nexusSnapshotUrl}`）
- [x] 3.2 **方案调整**：私服属性/凭证/mirror 统一由 `~/.m2/settings.xml` 的 `bjca` profile（默认激活）提供；Makefile 去掉 `-s settings.xml`，项目内 settings.xml 保留为 Spring 官方原版（构建不再引用）
- [x] 3.3 `./mvnw -DskipTests clean deploy` 真实发布验证：制品（pom/jar/sources.jar/metadata）成功上传至 snapshots 私服，BUILD SUCCESS

## 阶段 4：CVE 文档体系

- [x] 4.1 `doc/CVE/CVE-2026-41721.md`（本体，✅已修复）
- [x] 4.2 `doc/CVE/CVE-2026-41711.md`（本体，✅已修复）
- [x] 4.3 `doc/CVE/CVE-2026-41716.md`（本体，✅已修复）
- [x] 4.4 `doc/VULNERABILITY_REPORT.md`：6 态归一化总览 + 统计 + 索引
- [x] 4.5 校验所有 CVE 文档链接与状态一致

## 阶段 5：交付文档

- [x] 5.1 `doc/GAV_MAPPING.md`：GAV 映射表
- [x] 5.2 `doc/QUICK_START.md`：快速入门（命令已更新为默认 settings）
- [x] 5.3 `doc/USER_MANUAL.md`：用户手册
- [x] 5.4 `doc/REQUIREMENTS.md`：需求与版本清单
- [x] 5.5 `Makefile`：build / test / install / deploy 快捷命令

## 阶段 6：收尾

- [x] 6.1 `openspec validate nes-patch-2026` 通过
- [x] 6.2 全量构建 + 测试最终验证（clean test 3328 全绿；clean deploy BUILD SUCCESS）
- [x] 6.3 更新 VULNERABILITY_REPORT 最终状态（三本体 CVE 均 ✅已修复）
- [x] 6.4 归档 change 到 `openspec/changes/archive/`
