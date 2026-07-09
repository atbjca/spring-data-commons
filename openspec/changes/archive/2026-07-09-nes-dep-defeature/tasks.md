# Tasks — nes-dep-defeature

> 图例：`[ ]` 未开始 `[~]` 进行中 `[x]` 完成

## 阶段 0：前置确认

- [x] 0.1 确认内网 Nexus 可解析 `bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1-SNAPSHOT`

## 阶段 1：依赖去特征化（pom.xml）

- [x] 1.1 `<dependencyManagement>` 增加 import 作用域的 fork framework BOM（`5.3.39-nes.patch.1-SNAPSHOT`）
- [x] 1.2 9 个 Spring 直接依赖坐标 `org.springframework:spring-*` → `cn.bjca.footstone.bpring:bjca-footstone-bpring-*`
- [x] 1.3 各 `<dependency>` 移除显式 `<version>`，改由 fork BOM 托管

## 阶段 2：解析与回归验证

- [x] 2.1 `mvn dependency:tree` 确认 fork BOM 生效、5.3.39 覆盖官方 5.3.31、无版本冲突
- [x] 2.2 `./mvnw clean test` 在 fork 依赖下全量回归通过（Failures 0 / Errors 0）

## 阶段 3：文档同步

- [x] 3.1 `doc/GAV_MAPPING.md` 增加「依赖坐标去特征化」映射节（含 framework BOM）
- [x] 3.2 `doc/REQUIREMENTS.md` §4 依赖概览更新为 fork BOM `5.3.39-nes.patch.1`
- [x] 3.3 `openspec/specs/gav-renaming/spec.md` 合入本变更新增 Requirement

## 阶段 4：收尾

- [x] 4.1 `openspec validate nes-dep-defeature` 通过
- [x] 4.2 归档 change 到 `openspec/changes/archive/`
