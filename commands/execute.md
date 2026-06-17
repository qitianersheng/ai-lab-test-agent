---
description: Step 2 — 测试执行（AI 实时驱动 Playwright）。选环境与模式，逐条用例实时执行并判四态
argument-hint: [可选：模块名或 all]
---

# Step 2：测试执行（AI 实时驱动 Playwright）

> **V0.3 变更**：不再有 `.spec.ts`、不再 `npx playwright test`。直接读测试用例，用 Playwright MCP 浏览器工具实时驱动浏览器逐条执行、自判四态。

## 前置检查

1. `/AI_LabTest/testCase/{模块}-testcase.md` 存在且非空（缺失 → 提示先完成 Step 1）
2. `AI_LabTest/environments.json` 存在（缺失 → 提示先运行 `/ai-lab-test:init`）
3. Playwright MCP 浏览器工具（`browser_*`）可用（本插件已随附 `playwright` MCP 服务，插件启用即自动注册；若不可用，提示用户确认插件已启用 / 重载插件）

## 规则
严格遵循 `${CLAUDE_PLUGIN_ROOT}/rules/test-execution.md`（**先用 Read 工具读取该文件再执行**），**特别是第 0 节「环境预检」与第 2 节「四态判定」**。

---

## 执行步骤

### 1. 环境选择（强制 — 必须在范围/模式之前）

读 `environments.json` 的 `allowedEnvs`（默认 `["local", "sit"]`），用 `AskUserQuestion`：

**问题 1：本次测试目标环境？**
- 本地（local）— frontendBaseUrl: {local.frontendBaseUrl}
- SIT — {sit.frontendBaseUrl 或 「未配置，将引导填写」}

> ⚠️ **范围边界**：不支持 UAT 与生产。UAT 由业务方手工验收；生产永不可被自动化触达；用户主动要求测 UAT 也必须拒绝。

若选 SIT 且为 TBD：依次问前端 URL / API URL / 账号 username / password / role，写回 `environments.json`。

### 2. 被测地址探活（强制）

GET `frontendBaseUrl`（有响应即视为站点在跑）：

| 结果 | 行为 |
|---|---|
| 可达 | ✅ 进入执行 |
| 连接失败/超时 | ❌ **中断**：local 请启动前端服务 / SIT 请检查 VPN 与地址。不可达不要白跑一整轮 |

### 3. 范围与模式选择

**问题 2：要执行哪些模块？** 根据 `testCase/` 实际用例文档动态列出（全部 / 单模块 / 自定义）。

**问题 3：执行模式？**
- 后台（无头，默认，更快）
- 可见（有头浏览器，操作肉眼可见）

### 4. 逐条执行（用 Playwright MCP 浏览器工具）

对所选模块的每条用例：

1. 读「前置条件 / 测试操作 / 预期结果 / 断言级别」。
2. `browser_navigate` 打开被测地址；需要登录的用例先用账号登录建立会话。
3. 按用例步骤操作（`browser_click` / `browser_type` / `browser_fill_form` / `browser_select_option` / `browser_wait_for`），用 `browser_snapshot` 观察真实结构。
4. 对照「预期结果」判四态（pass / fail / partial / blocked，见 test-execution.md §2）。
5. `browser_take_screenshot` 截图留证。

### 5. 落盘结果

把逐条结果写入 `/AI_LabTest/report/{模块}-{ENV}.results.json`（schema 见 test-execution.md §4）；
归约出的 `/AI_LabTest/report/{模块}-{ENV}.run.json` 是 Step 3 报告的唯一权威事实来源。

**严禁修改源代码**——执行只在被测系统页面内操作；fail 是事实通报，不放水、不给修复方案。

### 6. flaky 处理

flaky 可单次重跑 1 次；仍不稳定按事实记录。

---

## 用户门控

环境 + 范围 + 模式共 3 次问询，之后自动执行直到结束。

## 输出

- `/AI_LabTest/report/{模块}-{ENV}.results.json`（AI 落盘逐条结果）
- `/AI_LabTest/report/{模块}-{ENV}.run.json`（归约四态，供 Step 3）
- 证据截图

## 完成后的提示（陈述式，不问询）

> Step 2 完成于 **{ENV}** 环境（{frontendBaseUrl}）。{N} 条用例，{X} pass / {Y} fail / {P} partial / {B} blocked。
> 下一步：运行 `/ai-lab-test:report` 生成 `{module}-report-{ENV}-{YYYYMMDD}-{HHMMSS}.md`。

## 参数

$ARGUMENTS
