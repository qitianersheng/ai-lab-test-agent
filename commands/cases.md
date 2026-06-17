---
description: Step 4 — 设计测试用例并对照完整性矩阵核验
argument-hint: <模块名，必填>
---

# Step 4：测试用例设计

## 前置检查

1. **必须**存在 `/AI_LabTest/requirements/{模块}-requirement.md`（含评审补充）
2. **必须**存在 `/AI_LabTest/requirements/{模块}-coverage-matrix.md`
3. **必须**有模块名参数

## 规则
严格遵循 `${CLAUDE_PLUGIN_ROOT}/rules/test-case-completeness.md`（**先用 Read 工具读取该文件再执行**）。

## 执行步骤

1. 读取需求文档 + 覆盖矩阵
2. 为每个「需求 × 维度」组合生成测试用例（含判定表每条规则、状态迁移每个格 —— 见 test-case-completeness.md §2.1）
3. 每个用例必须包含 6 必填元素：
   - 目的 / 前置条件 / 测试操作 / 预期结果 / 预期结果来源（oracle）/ 清理
4. 每个用例必须标注：
   - ID：`TC-{模块}-{编号}`
   - 需求引用：`REQ-{模块}-{编号}` + 维度
   - 断言级别：L1/L2/L3/L4
   - oracle 来源 + 验证性质（功能验证 / 回归基线）—— 见 test-case-completeness.md §1.6
5. 对照完整性规则核验：结构 / 覆盖 / 断言 / 隔离
6. 在文档末尾输出「完整性检查」章节，列出每项核验结果

## 用户门控（强制）

使用 `AskUserQuestion`：
- 「以下 N 个测试用例是否可纯入步骤 5？」
- 选项：全部采纳 / 调整后生成 / 跳过某些 / 补充某些
- **若本批用例全部为「回归基线」（oracle = 现状锁定）**：额外提示「这些用例只锁定现状、不验证正确性，是否补充权威需求来源（PRD / 产品确认）？」（见 test-case-completeness.md §1.6）

**用户未确认前，不得进入 Step 5。**

## 输出

`/AI_LabTest/testCase/{模块}-testcase.md`

## 完成后的提示

> Step 4 完成。{模块} 已生成 N 个测试用例，覆盖维度 X 项。下一步：运行 `/ai-lab-test:execute {模块}` 进入测试执行（AI 实时驱动 Playwright，无门控，自动执行）。

## 参数

$ARGUMENTS
