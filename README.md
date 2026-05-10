# Law Skills

## 安装方法

```bash
npx skills add https://github.com/Rinsonlaw/law-skills.git
```

## 更新方法

```bash
npx skills update
```

## Magazine 预览

本地预览 magazine（需启动 HTTP server）：

```bash
cd law-skills && python3 -m http.server 8124
```

然后访问 http://localhost:8124/magazine.html

## builder-digest

调用 follow-builders 获取 AI Builders Digest 并写入 Obsidian。

- 自动获取日期和时间
- 调用 follow-builders 的 Content Delivery 流程获取内容
- 写入 Obsidian vault 的 `builders_digest/` 目录
- Markdown 格式包含 X/Twitter 精选、播客精选、趋势归纳
- 支持中英双语/Bilingual 格式（由 follow-builders 配置决定）

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

## news-analysis

每日热点新闻分析并写入 Obsidian。

- 使用 mmx search 搜索今日热点新闻（国际动态、国内时政、财经新闻、科技·AI）
- 100 字核心内容摘要，保留原文链接
- 自动写入 Obsidian news 文件夹，文件名精确到秒
