---
name: build-cron
description: 创建执行完成后能回复消息到指定渠道的 OpenClaw 定时任务（cron job）。当用户提到"创建定时任务"、"新建 cron"、"设置定时执行"、"定时运行"、"scheduled task"、"每隔"、"每分钟"、"每小时"、"每周"、"每月"、"点到点每隔"时使用此 skill。通过对话收集任务名称、任务描述、执行时间（支持定点时间、间隔循环、周期时间、定时范围四种模式），自动获取 session key（不需要用户配合），然后构造并执行 openclaw cron add 命令。
---

# Build Cron Skill

将用户需求转换为 `openclaw cron add` 命令并执行。

## 安装方法

```bash
npx skills add https://github.com/Rinsonlaw/law-skills.git
```

## 工作流程

### 第一步：收集信息

向用户询问以下信息（不要跳跃，等用户回复后再继续）：

1. **任务名称** — 用户给这个定时任务起的名字，如 `daily-report`
2. **任务描述** — 用户说的原话，直接填入 `--message`（不要总结，不要改写）
3. **执行时间** — 用户期望的执行时间，支持四种模式：
   - **定点时间**：询问小时（0-23）和分钟（0-59），如"20 30"表示晚上8点30分
   - **间隔循环**：询问间隔数值和单位，如"每5分钟"、"每2小时"
   - **周期时间**：指定每周几或每月几号，如"每周一9点"、"每月1号0点"
   - **小时范围**：指定在哪些小时执行，如"10点到12点每隔1小时"

> **Session Key 自动获取**：自动从当前会话的元数据中获取，不需要用户手动提供。

### 第二步：解析 sessionKey（自动获取）

自动从当前会话元数据中获取 sessionKey，不需要用户配合。然后用冒号 `:` 分割，提取以下字段：

```
session_key = "agent:research:feishu:direct:ou_xxx"
agent_id    = session_key_parts[1]   # → "research"
channel_name = session_key_parts[2]   # → "feishu"
target_id   = session_key_parts[4]   # → "ou_xxx"
```

### 第三步：解析执行时间

cron 表达式格式说明（从左到右）：
| 位置 | 代表含义 | 取值范围 |
|------|---------|---------|
| 第1个 | 分钟 | 0-59，`*/N` 表示每 N 分钟 |
| 第2个 | 小时 | 0-23，`*/N` 表示每 N 小时 |
| 第3个 | 日期 | 1-31 |
| 第4个 | 月份 | 1-12 |
| 第5个 | 星期 | 0-6（0=周日）|

**范围语法：**
- `*` — 每任意值
- `A` — 指定值
- `A-B` — 范围，适用于任意位置，如 `10-12`（小时）、`5-31`（日期）、`2-5`（星期）
- `*/N` — 间隔，如 `*/5` 表示每5个单位
- `A-B/N` — 范围间隔，如 `10-12/1` 表示从10到12每间隔1

- 示例：每 5 分钟 → `"*/5 * * * *"`（第1个位置为 `*/5`）
- 示例：每 2 小时 → `"0 */2 * * *"`（第2个位置为 `*/2`）
- 示例：每小时 → `"0 * * * *"`
- 示例：每周一早上9点 → `"0 9 * * 1"`
- 示例：每月1号凌晨0点 → `"0 0 1 * *"`
- 示例：10点到12点每1小时 → `"0 10-12/1 * * *"`
- 示例：工作日9点到18点每2小时 → `"0 9-18/2 * * 1-5"`

执行时间支持四种模式：

**模式 A：定点时间**
- 用户说"每天早上8点"、"晚上8点30分"等
- 需要解析出小时（0-23）和分钟（0-59）
- 生成 cron 表达式：`{minute} {hour} * * *`
- 示例：小时=8, 分钟=0 → `"0 8 * * *"`

**模式 B：间隔循环**
- 用户说"每隔5分钟"、"每30分钟"、"每小时"等
- 需要解析出间隔数值和单位（分钟/小时）
- 间隔循环的 cron 表达式格式：`*/{interval} * * * *`（每 N 分钟）或 `0 */{interval} * * *`（每 N 小时）


**模式 C：周期时间**
- 用户说"每周一9点"、"每月1号凌晨0点"等
- 需要解析出：周期类型（周/月）、具体日期（周一/1号）、小时和分钟
- 生成 cron 表达式：`{minute} {hour} {day_of_month} {month} {day_of_week}`
- 星期：0=周日，1=周一，...，6=周六
- 示例：每周一早上9点 → `"0 9 * * 1"`
- 示例：每月1号凌晨0点 → `"0 0 1 * *"`

**模式 D：定时范围**
- 用户说"10点到12点每隔1小时"、"工作日早上9点到下午6点每隔2小时"等
- 需要解析出：起始时间、结束时间、执行间隔
- cron 表达式格式：`{minute} {start_hour}-{end_hour}/{interval} * *`
- 示例：10点到12点每隔1小时，0分执行 → `"0 10-12/1 * * *"`
- 示例：9点到18点每隔2小时 → `"0 9-18/2 * * *"`
- 注意：范围通常指小时范围，间隔单位为小时

判断规则：
- 如果用户提到"每隔"、"每.*分钟"、"每.*小时"，则视为**间隔循环**模式
- 如果用户提到"每周"、"每月"、"周几"，则视为**周期时间**模式
- 如果用户提到"点到点每隔"，则视为**定时范围**模式
- 如果用户只说时间点（如"8点"、"20:30"），则视为**定点时间**模式

### 第四步：获取时区并构造 CLI 命令

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

**定点时间示例：**

```
用户：我想创建一个定时任务，每天晚上8点执行
助手：请问这个任务叫什么名字？
用户：nightly-digest
助手：请问这个任务具体要做什么？（直接填入 message，原话即可）
用户：每晚整理并总结当天的竞品动态
助手：请问执行时间是几点几分？（比如 20 30 表示晚上8点30分）
用户：20 30
助手：（自动获取当前 session key）
     识别为定点时间模式，构造如下命令，请确认是否创建：
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

**间隔循环示例：**

```
用户：我想每5分钟检查一下服务器状态
助手：请问这个任务叫什么名字？
用户：server-health-check
助手：请问这个任务具体要做什么？（直接填入 message，原话即可）
用户：检查服务器状态是否正常
助手：请问执行时间是怎样的？（比如 "每5分钟" 或 "每2小时"）
用户：每5分钟
助手：（自动获取当前 session key）
     识别为间隔循环模式（每5分钟），构造如下命令，请确认是否创建：
TZ=$(readlink /etc/localtime | sed 's|.*zoneinfo/||')
openclaw cron add \
  --name "server-health-check" \
  --agent "research" \
  --session-key "agent:research:feishu:direct:ou_xxx" \
  --cron "*/5 * * * *" \
  --tz "$TZ" \
  --session "isolated" \
  --wake "now" \
  --message "检查服务器状态是否正常" \
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
