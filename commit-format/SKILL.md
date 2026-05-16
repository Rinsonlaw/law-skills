---
name: commit-format
description: Requires AI agent git commits to use a fixed Chinese template with [update|bugfix|optimize] prefix, per-file bullet changelog, and Agent/Model lines. Use when running git commit, amending, or writing commit messages for agent-made changes.
---

# Git 提交格式（Agent 必填）

## 安装方法

```bash
npx skills add https://github.com/Rinsonlaw/law-skills.git
```

## 必须遵守

凡由 **AI 编程 Agent** 执行的 `git commit`（含 `commit --amend`），提交说明 **必须** 严格按下列结构书写（标点与换行保持一致意图即可，类型三选一）。

## 模板

（首行方括号为 **三选一**：只写 `[update]`、`[bugfix]`、`[optimize]` 其中之一，勿写斜杠串。）

```text
[update/bugfix/optimize] 提交内容

- 每个文件的修改内容
- …

Agent：xxx
Model：xxx
```

### 字段说明

| 部分 | 要求 |
|------|------|
| **类型** | 与模板对应：`[update]`（功能/内容更新）、`[bugfix]`（缺陷修复）、`[optimize]`（重构/性能/体验优化等） |
| **提交内容** | 同一行紧接在 `]` 后：一句话概括本次提交 |
| **文件列表** | 每条以 `- ` 开头，**每个被改动的文件至少一行**，写该文件的具体修改说明 |
| **Agent** | 全角冒号 `：`。写实际环境，如 `Cursor Composer`、`Cursor Agent` |
| **Model** | 全角冒号 `：`。写当前模型标识；未知则 `Model：unknown` |

## 示例

```text
[bugfix] 修复空输入时崩溃

- `src/wrinkle/main.py`: 对 argv 判空并给出用法提示
- `tests/test.py`: 补充空参数用例

Agent：Cursor Composer
Model：claude-sonnet-4-20250514
```

```text
[optimize] 精简依赖与导入

- `pyproject.toml`: 移除未使用的 dev 依赖

Agent：Cursor Agent
Model：unknown
```

## 说明

- 人类手动提交 **不强制** 本模板；仅约束 Agent 代为提交。
- 若一次提交涉及多文件，**不得** 合并成一条含糊 bullet，应 **按文件拆分** 说明。
