# 研究代码重构 Skill（中文版）

**[English](README_EN.md)** | **[中文](README_CN.md)**

---

一个面向学术研究项目（R / Stata / Python）的结构化代码重构框架，专为 **Claude Code** 设计，强制执行安全、增量、可审计的代码变更。

## 功能概述

- 强制执行**重构顺序**（格式化 -> 命名 -> 文档 -> 配置提取 -> 去重 -> 模块化 -> 函数抽象 -> 流程重构）
- 维护**依赖图**，确保任何变量或合并键的改名都经过完整的血缘追溯
- 要求**原子提交**：一次验证通过的变更 = 一次提交 = 一个回滚点
- 在 `progress.md` 中追踪进度，标注风险等级（LOW / MEDIUM / HIGH / CRITICAL）
- 全程提供 R、Stata、Python 三语言示例

---

## 风险提示

> **重要：使用本工具的风险由使用者自行承担。**
>
> 本 skill 按"原样"提供，不作任何担保。作者**不对以下情况负责**：
> - 代码丢失、文件损坏或意外修改
> - AI 辅助重构导致的分析结果错误
> - 重构过程中的数据损坏或丢失
> - 使用本工具产生的任何后果
>
> **使用前必须做到：**
> 1. **备份你的项目**（完整目录复制，或 `git stash` / 提交所有未保存的工作）
> 2. **在独立分支上工作**——绝对不要在 `main` 分支或没有备份的分支上重构
> 3. **逐条审查 AI 的每项建议**——不要盲目接受 AI 的输出
> 4. **每次变更后验证输出**——对比回归结果、数据维度和表格
>
> AI 辅助重构可能引入难以察觉的细微错误。请始终保持独立验证和回滚能力。

---

## 最佳实践

1. **始终在独立分支上重构：**
   ```bash
   git checkout -b refactor/extract-config
   ```
   绝对不要直接在 `main` 或任何未保护的分支上重构。

2. **频繁提交**——每次验证通过的变更独立提交，这样你可以随时 `git revert` 单个错误变更而不影响其他。

3. **重构前拍输出快照**——保存参考表格/数据，以便重构后对比。

4. **每次只改一个**——本 skill 会强制执行这一点，但值得反复强调：永远不要把多个逻辑变更混在一起。

---

## 安装方法

### 方式一：安装为 Claude Code Skill（推荐）

安装后可通过斜杠命令（如 `/research-refactor`）在所有项目中使用。

#### 通过终端（macOS / Linux）

```bash
# 1. 克隆本仓库
git clone https://github.com/YOUR_USERNAME/research-refactor-skill.git

# 2. 复制 skill 目录到你的 Claude Code 个人 skills 文件夹
mkdir -p ~/.claude/skills/research-refactor
cp research-refactor-skill/SKILL.md ~/.claude/skills/research-refactor/SKILL.md
cp research-refactor-skill/progress.md ~/.claude/skills/research-refactor/progress.md
```

安装完成后，在 Claude Code 中输入 `/research-refactor` 即可立即使用。

#### 按项目安装

如果你只想在单个项目中使用（并纳入版本控制）：

```bash
# 在你的项目根目录下执行
mkdir -p .claude/skills/research-refactor
cp /path/to/research-refactor-skill/SKILL.md .claude/skills/research-refactor/SKILL.md
cp /path/to/research-refactor-skill/progress.md .claude/skills/research-refactor/progress.md

# 提交到项目仓库
git add .claude/skills/research-refactor/
git commit -m "chore: add research refactoring skill"
```

### 方式二：安装为 Claude Code Plugin

通过插件市场分发和团队共享：

```bash
# 1. 添加市场源（如果已发布）
# 在 Claude Code 中执行：
/plugin marketplace add YOUR_USERNAME/research-refactor-skill

# 2. 安装插件
/plugin install research-refactor@marketplace-name
```

### 方式三：在 VS Code Claude 扩展中使用

1. 在 VS Code 中安装 Claude 扩展
2. 在 Claude 输入框中输入 `/` 查看可用的 skill 列表
3. 如果已通过方式一或方式二安装，skill 会自动出现在列表中
4. 也可以在 VS Code Claude 面板中通过 `/plugins` 管理插件

---

## 使用方法

### 在 Claude Code（终端）中使用

```bash
# 在你的研究项目目录下启动 Claude Code
cd /path/to/your/research/project
claude

# 调用 skill
/research-refactor 将硬编码的年份筛选提取到 config.yaml
```

skill 将会：
1. 读取 `SKILL.md` 和 `progress.md` 了解项目状态
2. 分析依赖图
3. 按照强制重构顺序提出变更建议
4. 等待你审查后再执行任何修改

### 在 VS Code 中使用

1. 在 VS Code 中打开你的项目
2. 打开 Claude 面板（侧边栏或 `Cmd+Shift+P` -> "Claude: Open"）
3. 输入 `/research-refactor` 后跟你的任务描述
4. 在 diff 视图中审查建议的变更后接受

### 快速开始清单

```bash
# 1. 备份你的工作
git stash  # 或: cp -r /project /project_backup

# 2. 创建重构分支
git checkout -b refactor/initial-setup

# 3. 在项目中设置 skill 文件
mkdir -p docs
cp /path/to/SKILL.md docs/skill.md
cp /path/to/progress.md docs/progress.md

# 4. 根据你的项目定制
# 编辑 docs/progress.md 以反映你的实际项目结构

# 5. 启动 Claude Code 并调用 skill
claude
# 然后输入: /research-refactor 审查我的项目结构
```

---

## 文件结构

```
research-refactor-skill/
├── README.md           # 语言选择入口
├── README_EN.md        # 英文文档
├── README_CN.md        # 中文文档
├── SKILL.md            # 主 skill 指令文件（含 frontmatter）
└── progress.md         # 进度追踪模板
```

在你的研究项目中，推荐的结构为：

```
your-project/
├── config/
│   ├── config.yaml
│   └── variables.yaml
├── data/
│   ├── raw/
│   ├── interim/
│   └── final/
├── src/
│   ├── 00_master.R
│   ├── 01_clean.R
│   ├── 02_construct.R
│   └── 03_analysis.R
├── output/
│   ├── tables/
│   ├── figures/
│   └── logs/
├── tests/
├── docs/
│   ├── skill.md        # 重构规则（本 skill）
│   └── progress.md     # 重构进度追踪
└── .claude/
    └── skills/
        └── research-refactor/
            └── SKILL.md
```

---

## 本 Skill 强制执行的核心规则

| 规则 | 说明 |
|---|---|
| 永不静默更改结果 | 回归系数、样本量、输出必须保持一致 |
| 配置与代码分离 | 所有参数写入 `config.yaml`，绝不硬编码在脚本中 |
| 仅增量变更 | 每次提交只包含一个逻辑变更，每个提交可独立回滚 |
| 先建依赖图 | 没有完成依赖图就不允许重构 |
| 错误不得静默忽略 | 禁止裸 `tryCatch`、禁止无日志的 `capture` |
| AI 必须先询问再行动 | 遇到歧义 = 停下来问，绝不猜测 |

---

## 许可证

MIT License. 详见 [LICENSE](LICENSE)。

## 贡献

欢迎贡献。请在 GitHub 上提交 Issue 或 Pull Request。
