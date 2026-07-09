# 需求与版本清单 (Requirements)

## 1. 项目版本

| 项 | 值 |
|----|-----|
| 基线版本 | Spring Data Commons 2.7.18（官方正式版） |
| 制品版本 | 2.7.18-nes.patch.1-SNAPSHOT |
| Parent | `org.springframework.data.build:spring-data-parent:2.7.18` |
| 构建工具 | Maven（`mvnw` 3.9.5） |
| 编译目标 | Java 8（字节码 major version 52） |
| Git 分支 | `2.7.x-bjca-patch` |

## 2. 改造需求清单

| 编号 | 需求 | 状态 |
|:---:|------|:---:|
| R1 | 修复本体 CVE-2026-41721（MapDataBinder DoS） | ✅ 完成 |
| R2 | 修复本体 CVE-2026-41711（PropertyPath 栈溢出 DoS） | ✅ 完成 |
| R3 | 修复本体 CVE-2026-41716（无界负结果缓存 DoS） | ✅ 完成 |
| R4 | GAV 去特征化（groupId/artifactId/version） | ✅ 完成 |
| R5 | Parent 版本锁定 2.7.18（私服可解析） | ✅ 完成 |
| R6 | 私服发布配置（distributionManagement） | ✅ 完成 |
| R7 | CVE 文档体系（VULNERABILITY_REPORT + 每 CVE 独立文档） | ✅ 完成 |
| R8 | 交付文档（用户手册、快速入门、GAV 映射、需求清单） | ✅ 完成 |
| R9 | Makefile 构建快捷命令 | ✅ 完成 |

## 3. 约束

| 约束 | 说明 |
|------|------|
| Java 8 兼容 | 所有修复与测试代码仅用 Java 8 API 与语法，产物字节码 Java 8 |
| 包名不变 | `org.springframework.data.*` 保持，下游 `import` 无需修改 |
| JPMS 模块名不变 | `spring.data.commons` |
| API 兼容 | 对外公开 API 签名不变 |
| TDD | 本体修复先写失败测试，再实现 |
| 私服隔离 | 依赖解析与发布走内网 Nexus，不直连外网 |

## 4. 依赖概览

本项目直接依赖的 Spring 组件已去特征化（坐标切换为 NES fork），版本由 fork 的 framework BOM 统一托管——`cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom:5.3.39-nes.patch.1-SNAPSHOT`（以 `import` 作用域置于本项目 `<dependencyManagement>`，覆盖 parent 传递的官方 `spring-framework-bom:5.3.31`）。编译期关键依赖：

| 依赖 | 坐标 | 版本 | 说明 |
|------|------|------|------|
| framework BOM | `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom` | 5.3.39-nes.patch.1-SNAPSHOT | 统一托管以下组件版本（import scope） |
| spring-core（fork） | `cn.bjca.footstone.bpring:bjca-footstone-bpring-core` | 由 BOM 托管 | 核心工具（compile） |
| spring-expression（fork） | `cn.bjca.footstone.bpring:bjca-footstone-bpring-expression` | 由 BOM 托管 | SpEL，CVE-2026-41721 修复所用 `SpelParserConfiguration` 三参构造器所在 |

> 直接依赖去特征化由独立变更 `nes-dep-defeature` 管理。传递依赖 CVE 盘点不在本期范围，如需可另起 openspec change。

## 5. 测试覆盖

| 修复 | 测试 | 结果 |
|------|------|:---:|
| CVE-2026-41721 | MapDataBinderUnitTests（7 个） | ✅ |
| CVE-2026-41711 | PropertyPathUnitTests（44 个） | ✅ |
| CVE-2026-41716 | TypeDiscovererUnitTests（19 个） | ✅ |
| 回归 | web + mapping + util 包（2056 个） | ✅ |
