## Why

`nes-patch-2026` 的 GAV 去特征化只覆盖了**本项目自身**的坐标（groupId / artifactId / version / parent），其 `tasks.md` 阶段 2 明确「不纳入传递依赖」。但发布 POM 的 `<dependencies>` 中仍保留 `org.springframework:spring-*` 官方坐标——SCA 工具扫描发布物料时，这些官方依赖坐标同样会按 GAV 特征命中 CVE 误报。

因此需要补一步：将本项目直接依赖的 Spring 组件坐标一并切换到 NES fork 坐标，并由 fork 的 framework BOM 统一托管版本。此改动已在 `pom.xml` 落地（工作区改动），但**不在原 `nes-patch-2026` 计划内**，且交付文档（`GAV_MAPPING.md`、`REQUIREMENTS.md`）尚未反映——本变更将其立为正式的、可追溯的独立变更，并同步文档。

## What Changes

- **引入 framework fork BOM**：`pom.xml` 的 `<dependencyManagement>` 增加 import 作用域的
  `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1-SNAPSHOT`，
  替代原先由 parent 传递的官方 `spring-framework-bom:5.3.31`，统一托管去特征化后的 Spring 组件版本。
- **依赖坐标去特征化**：本项目直接依赖的 8 个 Spring 组件由 `org.springframework:spring-*`
  重命名为 `cn.bjca.footstone.bpring:bjca-footstone-bpring-*`：
  `core`、`beans`、`context`、`expression`、`tx`、`oxm`、`web`、`webflux`、`webmvc`。
- **验证前提变化**：`nes-patch-2026` 的「3328 测试全绿」是在官方 `5.3.31` 依赖下取得的；
  切换到 fork `5.3.39-nes.patch.1` 后需重新回归（已重跑，全量通过，Failures 0 / Errors 0）。
- **文档同步**：`doc/GAV_MAPPING.md` 增加「依赖坐标去特征化」映射节；
  `doc/REQUIREMENTS.md` §4 依赖概览更新为 fork BOM 版本。

## Capabilities

### Modified Capabilities

- `gav-renaming`：在既有「本项目 GAV 去特征化」之上，新增「直接依赖坐标 + framework BOM 去特征化」要求。

## Impact

- **构建配置**：`pom.xml`（新增 `<dependencyManagement>` import fork BOM；9 处 `<dependency>` 坐标切换）。
- **依赖解析前提**：内网 Nexus 必须可解析 `bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1-SNAPSHOT`
  及其托管的 `bjca-footstone-bpring-*` 组件（已确认存在）。
- **文档**：`doc/GAV_MAPPING.md`、`doc/REQUIREMENTS.md`。
- **下游影响**：无。依赖为本项目内部消费，下游只依赖本项目主制品坐标；Java 包名 `org.springframework.*` 不变。
- **风险点**：
  - fork BOM `5.3.39` 相对官方 `5.3.31` 为同 minor 内 patch 递进，理论兼容；已通过全量回归验证。
  - 发布链 `make deploy` 带 `-DskipTests`，故回归须在 `make test` / CI 阶段独立保证，不能依赖 deploy 拦截。
