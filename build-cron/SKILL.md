---
name: build-cron
description: 创建 OpenClaw 定时任务（cron job）。当用户提到"创建定时任务"、"新建 cron"、"设置定时执行"、"定时运行"、"scheduled task"时使用此 skill。通过对话收集任务名称、任务描述、执行时间（小时和分钟），自动获取 session key（不需要用户配合），然后构造并执行 openclaw cron add 命令。
---

# Build Cron Skill

将用户需求转换为 `openclaw cron add` 命令并执行。

## 工作流程

### 第一步：收集信息

向用户询问以下信息（不要跳跃，等用户回复后再继续）：

1. **任务名称** — 用户给这个定时任务起的名字，如 `daily-report`
2. **任务描述** — 用户说的原话，直接填入 `--message`（不要总结，不要改写）
3. **执行时间** — 用户期望的执行时间，询问**小时**（0-23）和**分钟**（0-59）

> **Session Key 自动获取**：自动从当前会话的元数据中获取，不需要用户手动提供。

### 第二步：解析 sessionKey（自动获取）

自动从当前会话元数据中获取 sessionKey，不需要用户配合。然后用冒号 `:` 分割，提取以下字段：

```
session_key = "agent:research:feishu:direct:ou_xxx"
agent_id    = session_key_parts[1]   # → "research"
channel_name = session_key_parts[2]   # → "feishu"
target_id   = session_key_parts[4]   # → "ou_xxx"
```

### 第三步：生成 cron 表达式和获取时区

1. 将用户输入的小时和分钟转换为 cron 表达式：
```
cron_expr = "{minute} {hour} * * *"
```
   示例：小时=13, 分钟=30 → `"30 13 * * *"`

2. 自动获取 IANA 时区：
```bash
TZ=$(readlink /etc/localtime | sed 's|.*zoneinfo/||')
# 例如 /var/db/timezone/zoneinfo/Asia/Shanghai → "Asia/Shanghai"
```

### 第四步：构造 CLI 命令

使用以下参数构造 `openclaw cron add` 命令：

```bash
TZ=$(readlink /etc/localtime | sed 's|.*zoneinfo/||')
openclaw cron add \
  --name "{task_name}" \
  --agent "{agent_id}" \
  --session-key "{session_key}" \
  --cron "{cron_expr}" \
  --tz "$TZ" \
  --session "isolated" \
  --wake "now" \
  --message "{task_message}" \
  --timeout-seconds 600 \
  --announce \
  --channel "{channel_name}" \
  --to "{target_id}"
```

**参数说明：**

| CLI 参数 | 说明 | 默认值 |
|---------|------|--------|
| `--name` | 任务名称 | 必填 |
| `--agent` | 从 sessionKey 解析 | 必填 |
| `--session-key` | 完整 session key | 必填 |
| `--cron` | cron 表达式 | 必填 |
| `--tz` | 时区 | 自动检测 `readlink /etc/localtime` |
| `--session` | session 模式 | `"isolated"` |
| `--wake` | 唤醒模式 | `"now"` |
| `--message` | 任务描述（用户的原话，不做任何修改） | 必填 |
| `--timeout-seconds` | 超时秒数 | `600` |
| `--announce` | 启用结果播报到 channel | 默认关闭 |
| `--channel` | 播报 channel | 从 sessionKey 解析 |
| `--to` | 播报目标 | 从 sessionKey 解析 |

> **注意**：时区固定为 `"Asia/Shanghai"`，若用户需要其他时区可替换。

### 第五步：执行创建命令

将构造好的命令展示给用户确认，用户输入 `yes` 后执行。

## 完整对话示例

```
用户：我想创建一个定时任务，每天晚上8点执行
助手：请问这个任务叫什么名字？
用户：nightly-digest
助手：请问这个任务具体要做什么？（直接填入 message，原话即可）
用户：每晚整理并总结当天的竞品动态
助手：请问执行时间是几点几分？（比如 20 30 表示晚上8点30分）
用户：20 30
助手：（自动获取当前 session key）
     构造如下命令，请确认是否创建：
TZ=$(readlink /etc/localtime | sed 's|.*zoneinfo/||')
openclaw cron add \
  --name "nightly-digest" \
  --agent "research" \
  --session-key "agent:research:feishu:direct:ou_xxx" \
  --cron "30 20 * * *" \
  --tz "$TZ" \
  --session "isolated" \
  --wake "now" \
  --message "每晚整理并总结当天的竞品动态" \
  --timeout-seconds 600 \
  --announce \
  --channel "feishu" \
  --to "ou_xxx"
是否确认创建？（输入 yes 确认）
用户：yes
助手：执行命令...
```

## 注意事项

- **必须等待用户回复**再继续下一步，不要假设用户提供的信息
- 如果自动获取 session key 失败，记录错误并告知用户
- 所有固定值参数已在命令模板中写死，不需要用户输入
- 命令生成后**必须得到用户确认**才能执行
