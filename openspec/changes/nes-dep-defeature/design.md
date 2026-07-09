# Design — nes-dep-defeature

## 1. 背景与定位

本变更是 `nes-patch-2026` 的补充。原变更完成了「本项目自身 GAV 去特征化」，但有意将传递/直接依赖排除在外（`tasks.md` 第 12 行：「不纳入传递依赖 CVE」）。实践中发现：发布 POM 的 `<dependencies>` 仍带官方 `org.springframework` 坐标，SCA 工具扫描发布物料时仍会命中官方特征。本变更闭合这一缺口。

## 2. 关键决策

### 2.1 依赖版本托管方式：import fork BOM（而非逐个钉版本）

**决策**：在本项目 `<dependencyManagement>` 中 import `bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1-SNAPSHOT`，各 `<dependency>` 不写 `<version>`。

**理由**：
- 与官方 Spring 的 BOM 用法一致，版本集中托管，避免逐依赖钉版本导致的版本漂移。
- 本项目 `<dependencyManagement>` 的优先级高于 parent 继承，fork BOM 覆盖 parent 传递的官方 `spring-framework-bom:5.3.31`，无需改动 parent。

### 2.2 版本落点：5.3.31 → 5.3.39

**决策**：采用 fork BOM 已发布的 `5.3.39-nes.patch.1-SNAPSHOT`，对应 framework fork 的 5.3.39 基线。

**理由**：同 minor（5.3.x）内 patch 递进，API 与行为兼容；fork BOM 在内网 Nexus 已存在、可解析。风险以全量回归兜底。

### 2.3 parent 保持不变（方案 A 延续）

parent 仍为 `spring-data-parent:2.7.18`。fork BOM 通过本项目 `<dependencyManagement>` 就近覆盖，不触碰 parent，最小化改动面。

## 3. 验证

- **解析验证**：`mvn dependency:tree` / `help:effective-pom` 确认 fork BOM 生效、5.3.39 覆盖 5.3.31、无版本冲突。
- **回归验证**：切换依赖后重跑 `./mvnw clean test`，全量通过（Failures 0 / Errors 0）。原 `nes-patch-2026` 的 3328 绿灯基于官方 5.3.31，不能直接沿用为本变更的背书。

## 4. 非目标

- 不盘点传递依赖（jackson-databind、guava、querydsl 等）的 CVE——仍归入后续独立变更。
- 不改动 parent 坐标或版本。
- 不改动任何 Java 源码或对外 API。
