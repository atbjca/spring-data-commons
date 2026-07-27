# 需求与版本清单 (Requirements)

## 1. 项目版本

| 项 | 值 |
|----|-----|
| 基线版本 | Spring Data Commons 3.5.13（官方正式版） |
| 制品版本 | 3.5.13-nes.patch.1 |
| Parent | `org.springframework.data.build:spring-data-parent:3.5.13` |
| 构建工具 | Maven（`mvnw` 3.9.16） |
| 编译目标 | Java 17（3.5.x 基线要求） |
| Git 分支 | `3.5.x-bjca-patch` |

## 2. 改造需求清单

| 编号 | 需求 | 状态 |
|:---:|------|:---:|
| R1 | 验证本体 CVE-2026-41721（MapDataBinder DoS）官方修复在本基线生效 | ✅ 完成 |
| R2 | 验证本体 CVE-2026-41711（PropertyPath 栈溢出 DoS）官方修复在本基线生效 | ✅ 完成 |
| R3 | 验证本体 CVE-2026-41716（无界负结果缓存 DoS）官方修复在本基线生效 | ✅ 完成 |
| R4 | GAV 去特征化（自身 groupId/artifactId/version） | ✅ 完成 |
| R5 | 上游 spring-framework 依赖替换为 NES fork（BOM + 10 个依赖） | ✅ 完成 |
| R6 | Parent 版本锁定 3.5.13（私服/本地可解析） | ✅ 完成 |
| R7 | 私服发布配置（distributionManagement） | ✅ 完成 |
| R8 | CVE 文档体系（VULNERABILITY_REPORT + 每 CVE 独立文档） | ✅ 完成 |
| R9 | 交付文档（用户手册、快速入门、GAV 映射、需求清单） | ✅ 完成 |
| R10 | Makefile 构建快捷命令 | ✅ 完成 |

## 3. 约束

| 约束 | 说明 |
|------|------|
| 零业务代码改动 | 三个本体 CVE 官方已修复，本 fork 不改任何业务源码 |
| 包名不变 | `org.springframework.data.*` 保持，下游 `import` 无需修改 |
| JPMS 模块名不变 | `spring.data.commons` |
| API 兼容 | 与官方 3.5.13 完全一致 |
| 私服隔离 | 依赖解析与发布走内网 Nexus，不直连外网 |

## 4. 依赖概览

本项目依赖由 parent `spring-data-parent:3.5.13` 统一管理；**上游 spring-framework 依赖已替换为去特征化 NES fork**，版本经 fork BOM 统一：

| 依赖 | 坐标 | 版本来源 |
|------|------|---------|
| fork BOM | `cn.bjca.footstone.bpring:bjca-footstone-bpring-framework-bom` | `6.2.19-nes.patch.1`（import） |
| spring-core 等 10 个 | `cn.bjca.footstone.bpring:bjca-footstone-bpring-*` | 由 BOM 管理 |
| spring-hateoas | `org.springframework.hateoas:spring-hateoas` | 保留官方（无对应 fork） |

> 对应官方 spring-framework 基线为 6.2.19（3.5.13 parent 指定）。传递依赖 CVE 盘点不在本项目范围（已由 spring-framework / spring-boot NES GAV 处理覆盖）。详见 [GAV 映射表](GAV_MAPPING.md)。

## 5. 测试覆盖（回归验证）

| CVE | 测试 | 结果 |
|------|------|:---:|
| CVE-2026-41721 | MapDataBinderUnitTests（8 个） | ✅ |
| CVE-2026-41711 | PropertyPathUnitTests（48 个） | ✅ |
| CVE-2026-41716 | TypeDiscovererUnitTests（33 个） | ✅ |
| 全量回归 | `clean install` 3630 个（Failures 0 / Errors 0 / Skipped 7） | ✅ |
