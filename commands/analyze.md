---
description: Step 1 — 代码库分析，产出模块矩阵并确认测试范围
argument-hint: [可选：模块名提示]
---

# Step 1：代码库分析

## 规则
严格遵循 `${CLAUDE_PLUGIN_ROOT}/rules/codebase-analysis.md`（**先用 Read 工具读取该文件再执行**）。

## 执行步骤

1. 扫描代码库收集：技术栈、前端路由、后端 API、数据模型、鉴权机制、已有测试
2. 按「路由分组 / 页面文件组织 / API 路由分组 / 领域目录」识别模块
3. 给每个模块标 P0/P1/P2 优先级（依据：鉴权、支付、核心业务流程 = P0；CRUD = P1；展示 = P2）
4. 写入 `/AI_LabTest/analysis/codebase-overview.md`（模板见 codebase-analysis.md 第 2 节）

## 用户门控（强制）

使用 `AskUserQuestion` 询问用户：
- 「本轮要测试哪些模块？」（提供识别到的模块清单）
- 「是否有需要排除的模块？」
- 「执行环境（baseURL）？本地 / 测试环境」
- 「测试账号 / 预置数据是否就绪？」

**用户未确认范围之前，不得进入 Step 2。**

## 输出

`/AI_LabTest/analysis/codebase-overview.md`

## 完成后的提示

向用户陈述：
> Step 1 完成。已分析 N 个模块，本轮范围 = {用户确认的模块}。下一步：运行 `/ai-lab-test:requirements {模块名}` 进入需求发现。

## 参数

$ARGUMENTS
