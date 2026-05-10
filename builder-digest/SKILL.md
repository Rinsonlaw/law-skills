---
name: builder-digest
description: AI builders digest 推送生成。调用 follow-builders 获取内容，格式化为推送文本。当用户提到"推送 builders digest"、"发送 builder 日报"、"ai digest 推送"时使用此 skill。
---

# Builder Digest Skill

将 AI Builders Digest 格式化为适合推送的文本。

## 工作流程

### 第一步：获取日期

```bash
DATE=$(date +%Y-%m-%d)
DATETIME=$(date +%Y-%m-%d_%H-%M-%S)
```

### 第二步：调用 follow-builders 获取内容

通过 `/follow-builders` skill 的 Content Delivery 工作流程（Step 1-6）生成 digest：

1. 执行 follow-builders 的 prepare-digest.js 脚本
2. 检查是否有内容；若有，将内容 remix 为 digest

等待 digest 生成完成后，获取生成的 digest 文本。

### 第三步：获取 Obsidian 路径

使用 `/obsidian` skill 获取本机 Obsidian vault 路径。

### 第四步：写入 Obsidian

1. 确定写入路径：`{Obsidian路径}/builders_digest/`
2. 文件命名格式：`{DATETIME}.md`
3. 按以下 Markdown 格式写入：

```markdown
# AI Builders Digest | {DATE}

---

## 📱 X/Twitter 精选

### 1. [Builder 姓名（@账号名）]
> 英文摘要 [3-5 句话内容摘要]
**摘要**：[翻译英文版本后的中文摘要]
**来源**：[原文链接](原文链接)

---

### 2. [Builder 姓名]
> 英文摘要 [3-5 句话内容摘要]
**摘要**：[翻译英文版本后的中文摘要]
**来源**：[原文链接](原文链接)

---

...

---

## 🎙️ 播客精选

### 1. [播客名称]
**节目**：[节目名](链接)
> 英文摘要 [3-5 句话内容摘要]
**摘要**：[翻译英文版本后的中文摘要]
**来源**：[原文链接](原文链接)

---

...

---

## 📊 趋势归纳

- 1. [趋势点 1，不超过 100 字]
- 2. [趋势点 2，不超过 100 字]
- 3. [趋势点 3，不超过 100 字]
- 4. [趋势点 4，不超过 100 字]
- 5. [趋势点 5，不超过 100 字]

---

*整理时间：$(date +%Y-%m-%d\ %H:%M:%S)*
*来源：follow-builders + {当前模型}*
```

### 第五步：返回文档路径

返回已保存文件的路径：`{Obsidian路径}/builders_digest/{DATETIME}.md`

## 错误处理

- 若 prepare-digest.js 无输出（无网络），输出"今日无新内容，请检查网络连接"并退出
- 若 `stats.podcastEpisodes` 和 `stats.xBuilders` 均为 0，输出"今日无 builders 更新"并退出
- 若 config.json 不存在，提示用户先运行 `/follow-builders` 完成 onboarding
