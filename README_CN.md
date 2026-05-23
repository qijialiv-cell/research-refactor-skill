# 研究代码重构 Skill

**[English](README_EN.md)** | **[中文](README_CN.md)**

面向学术研究项目（R / Stata / Python）的结构化代码重构框架，专为 **Claude Code** 设计。

## 功能

- 强制**重构顺序**：格式化 -> 命名 -> 文档 -> 配置提取 -> 去重 -> 模块化 -> 函数抽象 -> 流程重构
- **依赖图**血缘追溯——任何变量改名前必须完整追溯
- **原子提交**：一次验证通过的变更 = 一次提交 = 一个回滚点
- `progress.md` 风险等级追踪（LOW / MEDIUM / HIGH / CRITICAL）
- 全程提供 R、Stata、Python 三语言示例

## 风险提示

> **使用风险由使用者自行承担。** 作者不对代码丢失、结果错误或任何损害负责。
>
> **使用前必须：**
> 1. 备份你的项目
> 2. 在**独立分支**上工作（`git checkout -b refactor/xxx`）——绝对不要在 `main` 上重构
> 3. 逐条审查 AI 的每项建议后再接受
> 4. 每次变更后验证输出（回归结果、数据维度、表格）

## 安装

```bash
git clone https://github.com/qijialiv-cell/research-refactor-skill.git

# 全局安装（所有项目可用）
mkdir -p ~/.claude/skills/research-refactor
cp research-refactor-skill/SKILL.md research-refactor-skill/progress.md ~/.claude/skills/research-refactor/

# 或按项目安装（在你的项目根目录下执行）
mkdir -p .claude/skills/research-refactor
cp ~/research-refactor-skill/SKILL.md ~/research-refactor-skill/progress.md .claude/skills/research-refactor/
```

## 使用

```bash
cd /你的项目 && claude
/research-refactor 将硬编码的年份筛选提取到 config.yaml
```

或在 **VS Code** 中：打开 Claude 面板 -> 输入 `/research-refactor <你的任务>`。

## 核心规则

| 规则 | 说明 |
|---|---|
| 永不静默更改结果 | 回归系数、样本量、输出保持一致 |
| 配置与代码分离 | 所有参数写入 `config.yaml`，绝不硬编码 |
| 仅增量变更 | 每次提交只包含一个逻辑变更 |
| 先建依赖图 | 没有完成依赖图就不允许重构 |
| 错误不得静默忽略 | 禁止裸 `tryCatch`、禁止无日志的 `capture` |
| AI 必须先询问再行动 | 遇到歧义 = 停下来问 |

## 许可证

MIT
