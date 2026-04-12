# Skills

## 安装方法

```bash
npx skills add git@github.com:Rinsonlaw/law-skills.git
```
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
