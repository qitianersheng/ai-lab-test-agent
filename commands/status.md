---
description: 查看 AI_LabTest 流水线状态 — 每个模块走到哪一步
---

# AI_LabTest 流水线状态

## 任务

扫描以下目录，按模块汇总每个模块当前所处的步骤：

| 目录 | 对应步骤 |
|---|---|
| `/AI_LabTest/analysis/codebase-overview.md` | Step 1 完成的标志（含模块清单） |
| `/AI_LabTest/requirements/{模块}-requirement.md` | Step 2 完成 |
| `/AI_LabTest/requirements/{模块}-review.json` | Step 3 完成 |
| `/AI_LabTest/testCase/{模块}-testcase.md` | Step 4 完成 |
| `/AI_LabTest/report/{模块}-{ENV}.run.json` | Step 5（测试执行）完成 |
| `/AI_LabTest/report/{模块}-report-*.md` | Step 6（测试报告）完成 |

## 输出格式

呈现表格：

```
模块            | S1 | S2 | S3 | S4 | S5 | S6 | 最后报告日期
--------------+----+----+----+----+----+----+-------------
login         | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 20260525
navigation-auth | ✅ |    |    |    |    |    | —
daily-practice | ✅ |    |    |    |    |    | —
```

底部附建议下一步：
> 建议下一步：运行 `/ai-lab-test:requirements navigation-auth` 继续 Step 2。

## 禁止

- 不修改任何文件，仅读取与展示
- 不发起 AskUserQuestion，仅呈现现状
