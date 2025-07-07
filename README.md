# prompt

A complete collection of my project.

## 📝 Git Markdown 文档仓库常见提交类型（Commit Types）

推荐使用的提交类型及其含义与示例。

| 类型         | 含义说明                                       | 示例                                               |
|--------------|------------------------------------------------|----------------------------------------------------|
| `feat`       | 新增文档或章节内容                             | `feat: add new note on git basics`                 |
| `fix`        | 修改拼写、语法、事实错误等内容                 | `fix: correct typo in python-lesson.md`            |
| `docs`       | 更新已有文档内容                               | `docs: update formatting of index.md`              |
| `style`      | 调整排版、样式、Markdown 格式等                | `style: format code blocks with syntax highlighting` |
| `refactor`   | 结构性调整（如重命名、拆分、合并文件）         | `refactor: split README into multiple files`       |
| `chore`      | 日常维护任务（如更新配置文件）                 | `chore: update .gitignore to ignore drafts`        |
| `perf`       | 性能优化（如图片压缩、加载优化）               | `perf: compress images in docs/images/`            |
| `test`       | 添加或修改测试文档                             | `test: add test cases for API docs`                |
| `build`      | 构建工具或流程相关（如静态网站生成）           | `build: generate site using mkdocs`                |
| `ci`         | 持续集成配置更改                               | `ci: update GitHub Actions workflow`               |
| `init`       | 初始化仓库结构                                 | `init: initial commit with basic folder structure` |
| `update`     | 更新已有文档内容                               | `update: revise chapter 3 content`                 |
| `remove` / `rm` | 删除无用文档                                | `rm: remove outdated draft notes`                  |
| `move` / `mv`   | 移动或重命名文件                              | `mv: move old-notes.md to archive/old-notes.md`    |

**特别说明**：本仓库中涉及到的提示词可能少部分来源于互联网，暂未找到出处，如有雷同，请及时与我取得联系进一步处理。

结构化提示词的各模块：

```markdown
# Role: <name> : 指定角色会让 GPT 聚焦在对应领域进行信息输出

## Profile author/version/description : Credit 和 迭代版本记录

## Goals: 一句话描述 Prompt 目标, 让 GPT Attention 聚焦起来

## Constrains: 描述限制条件, 其实是在帮 GPT 进行剪枝, 减少不必要分支的计算

## Skills: 描述技能项, 强化对应领域的信息权重

## Workflow: 重点中的重点, 你希望 Prompt 按什么方式来对话和输出

## Initialization: 冷启动时的对白, 也是一个强调需注意重点的机会
```
