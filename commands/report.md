---
description: Step 3 — 生成结构化测试报告（终态产物，文件名含环境标签）
argument-hint: [可选：模块名或 all]
---

# Step 3：测试报告生成

## 前置检查

1. **必须**存在最近一次 Step 2 执行结果 `/AI_LabTest/report/{模块}-{ENV}.run.json`（由测试执行归约写入）
   - 若不存在 → 中断，提示用户先运行 `/ai-lab-test:execute`
2. 报告以 `run.json`（四态计数 + 逐条状态 + 证据路径）为唯一权威事实来源

## 规则
严格遵循 `${CLAUDE_PLUGIN_ROOT}/rules/test-report.md`（**先用 Read 工具读取该文件再执行**），**特别是第 8 节：终态原则**。

## 执行步骤

### 1. 读取环境上下文

从 `.last-run-env.json` 提取：
- `tag`（local / SIT）→ 用作文件名后缀
- `env`（小写 local/sit）→ 用作 `latest-{env}.md` 后缀
- `frontendBaseUrl` / `apiBaseUrl` / `testAccount` → 写入报告概要节

> ⚠️ 如执行环境为 `uat` 或其他白名单外值 → 中断报告生成（防御性检查；正常流程下此情况不会出现，因为 Step 2 不会对 uat 执行）

### 2. 收集执行数据

- 通过 / 失败 / 跳过数与耗时
- 每条失败的错误信息、分类、可追溯链路

### 3. 生成报告（含 7 个章节）

按 `test-report.md` 第 2 节模板：
1. 概要（**首行必须明示 ENV + URL**）
2. 模块结果
3. 失败明细（含分类、根因、处理路径**陈述**）
4. 需求覆盖矩阵
5. 覆盖缺口
6. 应用 Bug 清单
7. 建议

### 4. 文件命名（必须含 ENV + 时分秒）

| 范围 | 文件名 |
|---|---|
| 单模块 | `/AI_LabTest/report/{模块}-report-{ENV}-{YYYYMMDD}-{HHMMSS}.md` |
| 多模块 | `/AI_LabTest/report/multi-report-{ENV}-{YYYYMMDD}-{HHMMSS}.md` |
| 本环境最新（覆盖） | `/AI_LabTest/report/latest-{env}.md` |

**示例**（含改进 3 时分秒后缀）：
- `login-report-local-20260525-171042.md`
- `login-report-SIT-20260525-171825.md`
- `latest-local.md` / `latest-sit.md`

**关键**：时分秒来自 `.last-run-env.json.executedAt`，不要用 `new Date()` 重新取（防止报告时间 ≠ 执行时间）：

```js
const iso = lastRunEnv.executedAt;
const date = iso.slice(0,10).replace(/-/g,'');
const time = iso.slice(11,19).replace(/:/g,'');
// filename: {module}-report-{tag}-{date}-{time}.md
```

## 用户门控（铁律 — 终态原则）

**禁止任何用户问询**。Step 3（测试报告）是终态：

| 禁止 | 含义 |
|---|---|
| ❌ AskUserQuestion 索取裁定 | 用例已在 Step 1 确认 |
| ❌「请你选择 A/B/C」 | 报告是产物不是问卷 |
| ❌ 暂停等待回复 | 一次性产出 |
| ❌「待用户确认」占位 | 失败分类必须明确 |
| ❌ 环境字段「未知」/ 留空 | 必须从 .last-run-env.json 读取，读不到则中断 |

**正确做法（陈述式）**：直接在报告中列出处理路径建议。

## 口头汇报（强制，4 件事）

报告写完后，**必须口头**向用户陈述：
1. **环境**：本次报告基于 {ENV} 环境（{frontendBaseUrl}）
2. **结果总览**：通过 / 失败 / 跳过数与百分比
3. **风险点**：应用 bug 数 + 高严重度 bug ID
4. **下一步建议**（陈述句）：修 bug / 补测试 / 跨环境对比 / 推进发布

**禁止以问句结尾**。

## 输出

- `/AI_LabTest/report/{模块}-report-{ENV}-{YYYYMMDD}.md`
- `/AI_LabTest/report/latest-{env}.md`

## 参数

$ARGUMENTS
