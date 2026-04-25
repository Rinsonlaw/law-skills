# Skills

## 安装方法

```bash
npx skills add git@github.com:Rinsonlaw/law-skills.git
```

## 在线预览

本地预览 magazine（需启动 HTTP server）：

```bash
cd /Users/law/Documents/code/law-skills && python3 -m http.server 8124
```

然后访问 http://localhost:8124/magazine.html

## build-cron

创建 OpenClaw 定时任务（cron job）。对话收集任务名称、任务描述、执行时间，自动获取 session key，构造并执行 `openclaw cron add` 命令。

## commit-format

AI Agent 提交格式规范。AI 执行 `git commit` 时强制使用，格式如下：

```
[type] 提交内容

- 文件：修改说明
- ...

Agent：xxx
Model：xxx
```

- `type`: `[update]` | `[bugfix]` | `[optimize]`

## follow-builders

AI Builders 内容聚合。追踪 X/Twitter 和 YouTube 播客上的 AI builder，生成可读的摘要。

- 无需 API key，所有内容从中心化 feed 获取
- 支持每日/每周定时推送
- 支持 Telegram、Email、stdout 等多种投递方式
