---
description: Step 2 — 需求分析（三级回退：用户提供 → 项目扫描 → 代码反推）
argument-hint: <模块名，必填>
---

# Step 2：需求分析

## 前置检查

1. **必须**存在 `/AI_LabTest/analysis/codebase-overview.md`；不存在则提示用户先运行 `/ai-lab-test:analyze`
2. **必须**有模块名参数（`$ARGUMENTS`）；不填则提示

## 预估耗时（先告知用户）

开始前先给用户一个区间预估，并说明取决于命中的回退路径：

| 命中路径 | 预估区间 |
|---|---|
| L1 用户直接提供文档 | ~1 分钟 |
| L2 项目内扫描 | 1–3 分钟 |
| L3 代码反推 | 3–8 分钟 |

话术示例：「预计 1–5 分钟，取决于是否需要从代码反推需求（L3 最慢）。」确定路径后可更新为更准确的预估。

## 规则
严格遵循 `${CLAUDE_PLUGIN_ROOT}/rules/requirement-discovery.md`（**先用 Read 工具读取该文件再执行**）的三级回退策略。

## 核心原则：统一归一

无论需求来自哪一级，**唯一标准产物始终是 `{模块}-requirement.md`**：有文档 → 结构化写入并标 `PROJECT_DOC`；无文档 → 反推生成并标 `REVERSE_ENGINEERED`。下游 Step 3 永远只读这一个文件。

## 执行步骤

### L1 — 询问用户是否有需求文档

使用 `AskUserQuestion`：
- 选项 1：路径形式（请提供 .md/.txt/.pdf/.docx 路径）
- 选项 2：粘贴形式（请粘贴需求文本）
- 选项 3：无 → 进入 L2

### L2 — 项目内扫描（L1 选无时）

约定：用户的需求文档**通常放在 `/docs`**，但仍扫描 `docs/`、`doc/`、`requirements/`、`PRD/`、`specs/`、`.planning/`、根目录 `*.md` 全部位置（不收敛到单一目录）。

对**当前模块**给候选文档算置信度（高=文件名精确命中 / 中=文件名或标题含模块名 / 低=正文关键词命中），按置信度决定是否打扰用户：

- **唯一高置信命中** → 直接采用，仅提示「已采用 `{file}`，如需更换请告知」（不弹问）
- **多候选 / 仅中低置信 / 零命中** → 用 `AskUserQuestion` 让用户单选，选项为：各候选文档（附来源路径 + 置信度）+ **`手动指定其他文件`** + **`该模块无文档 → 走代码反推`**
- 「手动指定」与「无文档→反推」两个兜底选项**始终带上**

选定文档后结构化写入 `{模块}-requirement.md`。

### L3 — 代码反推（L1+L2 均无产出，或用户选「无文档」时）

**先问再反推**：动手前先确认「{模块} 确实没有现成需求文档吗？」，用户确认后才反推。

按 requirement-discovery.md 第 1.3 节策略反推：
- 路由 → 功能模块
- 表单 + validators → 输入项 / 校验规则
- 错误处理 → 异常路径
- DB schema → 数据约束

**每条需求必须标注来源**：`USER` / `PROJECT_DOC: path#line` / `REVERSE_ENGINEERED: file:line`

## 用户门控（强制）

呈现完整需求清单，使用 `AskUserQuestion`：
- 「以下 N 条需求是否准确？」（全部/部分/调整/补充）
- 反推需求逐条确认（如有歧义）

**用户未对每个模块明确确认前，不得进入 Step 3。**

## 输出

`/AI_LabTest/requirements/{模块}-requirement.md`

## 完成后的提示

> Step 2 完成。{模块} 已生成 N 条需求（来源分布：USER X / PROJECT_DOC Y / REVERSE_ENGINEERED Z）。下一步：运行 `/ai-lab-test:review {模块}` 进入需求评审。

## 参数

$ARGUMENTS
