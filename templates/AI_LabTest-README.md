# AI_LabTest — Claude Code 自动化测试框架

一套可移植到任意项目的 AI 驱动自动化测试规则集。把 Claude Code 当成 QA 自动化工程师，按 3 步工作流交付从「测试用例设计」到「测试报告」的完整链路。

> **V0.4 起**：剔除「代码库分析 / 需求分析 / 需求评审」三步。测试用例设计直接以用户提供的需求文档或现成用例为输入，6 步 → 3 步。
> **V0.3 起**：取消「测试脚本生成」步——不再生成 `.spec.ts`。测试用例直接交给 AI，通过 **Playwright MCP** 浏览器工具实时驱动浏览器逐条执行、自判四态。

---

## 解决什么问题

传统自动化测试痛点：

- 测试用例与需求脱钩 → 无法追溯
- 测试失败时 AI 乱改源代码 → 引入新 bug
- 测试结果难以汇总 → 决策依据不足

AI_LabTest 用 3 步强制工作流 + 用户门控解决以上问题。

---

## 3 步工作流

```
┌──────────────────┐    用户确认用例
│ 1. 测试用例设计   │ ─────────────────→ testCase/{模块}-testcase.md
│   输入二选一：    │
│   · 用户需求文档  │
│   · 用户现成用例  │
└──────────────────┘
         ↓
┌──────────────────────────────┐
│ 2. 测试执行                   │  AI 经 Playwright MCP 实时驱动
│   （AI 实时驱动 Playwright）   │ ─────────→ report/{模块}-{ENV}.run.json（四态）+ 证据截图
└──────────────────────────────┘
         ↓
┌──────────────────┐
│ 3. 测试报告       │ ─────────────────→ report/{模块}-report-{ENV}-{日期}.md
└──────────────────┘
```

> **输入来源（用户驱动）**：测试用例设计只接受用户提供的需求文档（PRD / 设计稿 / 接口契约等）**或**用户直接给的测试用例。不再做代码库分析、三级回退需求发现、4 维度需求评审；用户两者都没有时，提示其先提供其一，不自行从代码反推。

---

## 目录结构

```
/AI_LabTest
├── README.md               # 本文档
├── environments.json       # 多环境注册表（local / SIT）
├── rules/                  # 规则定义（核心）
│   ├── test-workflow.md           # 总流程（3 步）
│   ├── test-case-completeness.md  # 步骤 1
│   ├── test-execution.md          # 步骤 2（AI 驱动 Playwright + 环境预检 + 四态）
│   ├── test-report.md             # 步骤 3（文件名含 ENV）
│   └── system_knowledge_map.md    # 附加能力：系统知识地图（独立于 3 步）
├── testCase/               # 步骤 1 输出
└── report/                 # 步骤 2 执行结果（run.json + 证据）+ 步骤 3 报告（文件名含 ENV）
```

> 项目根 `.mcp.json` 注册了 Playwright MCP（`@playwright/mcp`），步骤 2 执行靠它提供的 `browser_*` 工具。

---

## 移植到新项目（3 分钟）

### 1. 复制框架

```bash
# 在目标项目根目录
cp -r /path/to/this/AI_LabTest .                  # 含 AI_LabTest/CLAUDE.md（框架文件）
cp -r /path/to/this/.claude/commands ./.claude/   # 全部 slash 命令
# 在项目根 CLAUDE.md 追加一行（已存在则保留原内容）
grep -qxF '@AI_LabTest/CLAUDE.md' CLAUDE.md 2>/dev/null \
  || printf '\n@AI_LabTest/CLAUDE.md\n' >> CLAUDE.md
```

需要复制的文件：
- 整个 `/AI_LabTest/rules/` 目录（3 步流程的规则定义 + 知识地图）
- `/AI_LabTest/CLAUDE.md`（框架 QA 指南，独立文件）
- `/AI_LabTest/environments.json`（多环境注册表模板）
- 整个 `/.claude/commands/ai-lab-test*.md`（全部 slash 命令）
- `/AI_LabTest/README.md`（即本文档）
- 在项目根 `CLAUDE.md` 追加一行 `@AI_LabTest/CLAUDE.md`（不覆盖你已有的内容）

不需要复制的文件（项目特有）：
- `testCase/`、`tests/`、`report/`（这些是上一项目的产物）
- `.last-run-env.json`（运行时生成）
- `.claude/settings.local.json`（个人设置）

### 1.5 配置环境注册表

打开 `/AI_LabTest/environments.json`，根据实际项目调整。**仅支持 local 和 SIT 两个环境**，UAT 由业务方负责验收，不在本框架范围内。

```json
{
  "allowedEnvs": ["local", "sit"],
  "environments": {
    "local": {
      "frontendBaseUrl": "http://localhost:5173",   // ← 改成你项目的本地 URL
      "apiBaseUrl": "http://127.0.0.1:3001",
      "healthEndpoint": "/api/health",              // ← 改成你项目的 health 路径
      "testAccount": { "username": "...", "password": "..." }
    },
    "sit": { "frontendBaseUrl": "TBD", ... }        // ← TBD 留空，首次执行时会引导填
  }
}
```

`playwright.config.ts` 会在 `TEST_ENV=uat` 或任何非白名单值时主动抛错中断 — 这是一道安全护栏。

### 2. 安装测试依赖

V0.3 起执行靠 Playwright MCP（AI 实时驱动浏览器）：

```bash
npm install -D @playwright/mcp @playwright/test
npx playwright install chromium
```

并在项目根 `.mcp.json` 注册 playwright 服务（`scripts/install.sh` 会自动写/合并）：

```json
{ "mcpServers": { "playwright": { "command": "npx", "args": ["-y", "@playwright/mcp@latest", "--browser", "chromium"] } } }
```

> 用 `scripts/install.sh` 安装时，以上依赖与 `.mcp.json` 都会自动就绪。

### 3.（可选）playwright.config.ts

> V0.3 执行不再通过 spec 文件，本配置仅作「环境白名单」的防御性护栏 + 供手动 playwright 使用，非必需。最小配置：

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './AI_LabTest/tests',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:5173',
    headless: true,
    launchOptions: { slowMo: 0 },
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
  },
  reporter: [['html', { outputFolder: 'playwright-report' }], ['list']],
  projects: [
    {
      name: 'headed',
      use: { headless: false, launchOptions: { slowMo: 1000 } },
    },
    {
      name: 'headless',
      use: { headless: true },
    },
  ],
});
```

### 4. 启动

在新项目中打开 Claude Code，有 3 种触发方式：

**方式 1：主入口命令（推荐首次使用）**
```
/ai-lab-test
```
Claude 会从 Step 1（测试用例设计）开始走完 3 步。

**方式 2：自然语言**
```
我想为这个项目生成自动化测试
```
Claude 会经由根 CLAUDE.md 的 `@AI_LabTest/CLAUDE.md` 导入框架指南后启动同样流程（会先向你索取需求文档或现成用例）。

**方式 3：按步骤独立调用（推荐增量补充测试）**
```
/ai-lab-test-cases login          # 据需求文档/现成用例设计某模块用例
/ai-lab-test-execute              # 仅跑测试，会问选哪些
/ai-lab-test-status               # 查看当前流水线状态
```

完整命令清单见 `AI_LabTest/CLAUDE.md`「Slash 命令入口」节。

---

## 适配非 Playwright 项目

如果项目使用 Cypress / Selenium / Vitest E2E，需要做以下调整：

1. 修改 `AI_LabTest/CLAUDE.md` 中的「测试执行方式」节，更新所用工具
2. 在 `rules/test-execution.md` 中把「Playwright MCP 浏览器工具」适配为新工具的等价能力（导航/点击/输入/断言/截图）
3. `rules/test-case-completeness.md` **无需修改**（与测试框架无关）

---

## 常见问答

**Q：能跳过步骤吗？比如直接执行测试。**
A：禁止。每一步的产物是下一步的输入，跳过会导致质量崩塌。进入 Step 2 执行前必须已有 Step 1 测试用例。如确实要快速验证，使用 Playwright 原生工具，不要走 AI_LabTest 流程。

**Q：项目没有需求文档怎么办？**
A：V0.4 起测试用例设计只接受用户提供的输入——需求文档**或**现成测试用例。两者都没有时，框架会提示你先提供其一，**不**自行从代码反推需求（避免「AI 自说自话」）。如果你只有现成用例，直接把用例给到 Step 1 即可。

**Q：测试失败了 AI 会不会乱改我代码？**
A：不会。`AI_LabTest/CLAUDE.md` 明确禁止修改生产代码。失败只会被分类为「测试 bug / 应用 bug / 环境问题」并写入报告。

**Q：可以中途修改规则吗？**
A：可以，规则在 `/AI_LabTest/rules/`，按需求改。但建议先理解原规则的意图（用户门控、可追溯性、零源码修改）。

---

## 后续演进

| 阶段 | 形态 |
|------|------|
| 当前 | 项目内模板（复制粘贴）/ Claude Code 插件 |
| 远期 | 多语言、多框架、CI/CD 集成 |
