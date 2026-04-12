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
