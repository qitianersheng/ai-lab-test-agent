---
description: Step 3 — 需求评审（4 维度评分 + 覆盖矩阵 + 缺口交互）
argument-hint: <模块名，必填>
---

# Step 3：需求评审

## 入口硬校验（强制 — 先于一切操作）

需求评审消费的是 Step 2 的产物。无产物则无可评审对象，必须硬拦截，**不提供「知情后跳过」选项**：

1. `/AI_LabTest/requirements/{模块}-requirement.md` **存在**
2. 且**非空**（含至少 1 条带来源标注的需求，而非仅模板骨架）
3. 任一不满足 → 停止，告知用户「Step 2 尚无有效输出，需先返回 Step 2 生成需求清单」，并引导运行 `/ai-lab-test:requirements {模块}`，**不得进入评审**
4. **必须**有模块名参数（`$ARGUMENTS`）

> 注：UI 若把 Step 2 标为「可跳过」，指其用户门控确认可跳过，而非产物可缺失。Step 2 产物始终是 Step 3 的硬前置。

## 规则
严格遵循（**先用 Read 工具读取**）：
- `${CLAUDE_PLUGIN_ROOT}/rules/requirement-review.md`（4 维度评分）
- `${CLAUDE_PLUGIN_ROOT}/rules/requirement-completeness.md`（覆盖矩阵）

## 执行步骤

1. 读取 `{模块}-requirement.md`
2. 按 4 维度评分：清晰度 / 完整度 / 可测试性 / 原子性（每项 0-25，总分 0-100）
3. 输出 verdict：
   - `pass`（≥ 80 且无关键问题）
   - `pass_with_notes`（60-79）
   - `reject`（< 60 或致命缺失）
4. 为每个需求填充覆盖矩阵（正常 / 空 / 无效 / 边界 / 错误反馈 / 状态恢复，必要时加鉴权 / 数据生命周期维度；命中时补判定表 / 状态迁移 —— 见 `requirement-review.md` §7.1）

## 必须主动陈述的不足（强制 — 评分后第一时间）

按 `requirement-review.md` 第 7.3 节，**评分计算完成后立即向用户呈现**：

1. 每个维度的扣分明细表（扣分点 / 原文证据 / 扣分理由 / 可修复性）
2. 修复后理论最高分
3. 然后才进入用户门控问询

**禁止只报总分让用户主动问「为什么扣了 X 分」**。这是 v4 改进 1 的核心约束。

## 用户门控（强制 — 缺口必交互）

使用 `AskUserQuestion` 对以下情形必须发起交互：

| 触发 | 询问 |
|---|---|
| 任一维度 `MISSING` | 「当 {场景} 时应该发生什么？」 |
| 任一维度评分 `< 10` | 「{维度名} 维度具体问题：{quotes}。是否补充？」 |
| `verdict = reject` | 「这份需求被评为 reject，原因 X。退回修订 / 跳过 / 继续？」 |
| `escalation_triggers` 非空 | 「以下原因建议人工评审：{triggers}。是否继续？」 |
| **总有扣分（即便 verdict=pass）** | 「{N} 个扣分项已陈述。接受当前评分进 Step 4 / 修全部可修复项重评 / 部分修复？」 |

每次只问一个清晰问题，回答后追加到 `{模块}-requirement.md` 的「## 评审补充」章节。

**所有缺口闭合前，不得进入 Step 4。**

## 输出

- `/AI_LabTest/requirements/{模块}-review.json`（评分 JSON）
- `/AI_LabTest/requirements/{模块}-coverage-matrix.md`（覆盖矩阵）
- 更新后的 `/AI_LabTest/requirements/{模块}-requirement.md`（带评审补充）

## 完成后的提示

> Step 3 完成。{模块} 评分 X/100，verdict = {verdict}。缺口已闭合。下一步：运行 `/ai-lab-test:cases {模块}` 进入测试用例设计。

## 参数

$ARGUMENTS
