# my-workflow

> 个人 AI Coding Agent + 终端工作流：为 Codex CLI / Codex App、Claude Code、OpenCode 提供统一规则与共享 Skill，并复刻常用 macOS 终端环境。

- 仓库地址：<https://github.com/kiritoxkiriko/my-workflow>
- 作者偏好：简体中文沟通、最短路径优先、轻量任务直接处理，验证范围与改动风险匹配。

## 这是什么？

| 资产 | 文件 / 仓库 | 作用 |
|---|---|---|
| 全局规则 | [global/AGENTS.md](./global/AGENTS.md) | 指令优先级、工作方式、安全边界、完成标准与输出规范 |
| 个人扩展 Skill | 5 个 [leonsong09 的仓库](https://github.com/leonsong09) | 调研笔记、会话收尾、提交日报、项目日报、worktree 收口 |
| 安装剧本 | [AGENT-BOOTSTRAP.md](./AGENT-BOOTSTRAP.md) | 预检、共享安装、平台软链接、规则同步与验证 |
| 终端工具链 | [TERMINAL-SETUP.md](./TERMINAL-SETUP.md) | Yazi、zoxide、Neovim 与 Solarized Light / Dark |

全局规则只维护跨仓库稳定偏好，项目布局与构建命令留给项目级规则。本仓库的 [AGENTS.md](./AGENTS.md) 用于维护 workflow 资产，不分发到用户全局规则位置。

这份仓库分发下面列出的 5 个 Skill；用户自行安装的业务 Skill 和插件由各自来源管理。

## 核心理念

- 先读取上下文和已有改动，再做与任务直接相关的修改。
- 轻量任务直接处理，复杂任务先明确目标、边界、风险和验证方式。
- Skill 只保存一份实体，各平台通过软链接共用；更新共享实体会影响所有引用它的平台。
- 安装时保留已有文件和本地修改，发现冲突先核对，避免覆盖。
- 完成结论必须有验证证据；提交、推送和发布按用户授权执行。

## 我支持哪些 Agent？

共享 Skill 实体统一位于 `~/.agents/skills/<name>/`。各平台安装位置如下：

| Agent | Skill 入口 | 规则文件 |
|---|---|---|
| Codex CLI / Codex App | `~/.codex/skills/<name>` → 共享实体 | `~/.codex/AGENTS.md` |
| Claude Code | `~/.claude/skills/<name>` → 共享实体 | `~/.claude/CLAUDE.md` |
| OpenCode | `~/.config/opencode/skills/<name>` → 共享实体 | `~/.agents/AGENTS.md`，通过 instructions 引用 |
| 其他兼容 Agent | 直接使用共享目录，或按平台要求设置入口 | 按平台要求加载 `~/.agents/AGENTS.md` |

规则采用「通用位 + 平台副本」，内容保持一致；Skill 采用共享实体和软链接。文件就位后仍需在目标 Agent 会话中确认已识别。

## 快速开始（让 Agent 安装）

把下面这句话发给目标 Agent，并替换平台名称：

> 请阅读并按 <https://raw.githubusercontent.com/kiritoxkiriko/my-workflow/main/AGENT-BOOTSTRAP.md> 完成本机安装；目标平台是 `codex`（或 `claude` / `opencode` / `agents`）。保留已有安装和本地修改，遇到冲突先展示差异。

**不需要 clone 本仓库。** 规则通过 raw URL 下载；5 个 Skill 从各自上游 clone，保留脚本和模板的目录结构。

## 快速开始（人类操作版）

以下步骤与 [AGENT-BOOTSTRAP.md](./AGENT-BOOTSTRAP.md) §1–§5 一一对应。完整命令以安装剧本为准，在同一个 Bash 会话中依次执行；不要把 Markdown 文件直接作为 shell 脚本运行。

### 1. 选择平台并预检

在安装剧本 §1 设置 `PLATFORM=codex`、`claude`、`opencode` 或 `agents`，初始化目录和 5 个 Skill 的清单，检查共享实体、各平台旧安装与规则文件。

已有独立安装时，先核对其上游、提交和未提交改动；确认要保留的版本并备份后迁移到共享目录。脚本不会覆盖旧目录或失效链接。

### 2. 安装共享 Skill

执行安装剧本 §2，将缺失的 Skill clone 到 `~/.agents/skills`。已有合法实体会跳过，不自动更新。

例如 `research-note-wrap` 的实体位置为 `~/.agents/skills/research-note-wrap/SKILL.md`。每个 Skill 只需安装一次。

### 3. 建立平台软链接

执行安装剧本 §3，为选定平台建立入口。例如：

```text
~/.codex/skills/research-note-wrap ──→ ~/.agents/skills/research-note-wrap
~/.claude/skills/research-note-wrap ─→ ~/.agents/skills/research-note-wrap
```

已有正确链接会跳过；独立目录、错误链接和失效链接会报冲突并保留。安装另一个平台时，切换 `PLATFORM` 重跑流程即可，已有共享实体不会重新 clone。

### 4. 同步规则

执行安装剧本 §4，从以下地址下载规则，比较差异后备份并写入通用位和目标平台副本：

<https://raw.githubusercontent.com/kiritoxkiriko/my-workflow/main/global/AGENTS.md>

OpenCode 还需把 `~/.agents/AGENTS.md` 合并到 `~/.config/opencode/opencode.json` 的 instructions 中，保留现有条目与其他配置。

### 5. 验证

执行安装剧本 §5，检查每个 `SKILL.md`、软链接指向和规则副本一致性。重新打开目标 Agent 会话，确认 Skill 清单，或说「总结今天的调研输出笔记」「会话收尾」验证触发。

安装脚本验证文件与链接，实际会话识别需要单独确认。

## 我提供的 5 个个人扩展 Skill

| Skill | 触发场景 | 上游仓库 |
|---|---|---|
| `research-note-wrap` | 将调研 / 分析整理为 Obsidian 风格 Markdown 笔记 | <https://github.com/leonsong09/research-note-wrap> |
| `session-wrap` | 总结当前 coding 会话的结论、验证、风险和下一步 | <https://github.com/leonsong09/session-wrap> |
| `commit-daily-summary` | 将一天的 Git 提交聚合为中文日报 | <https://github.com/leonsong09/commit-daily-summary> |
| `project-daily-summary` | 按项目聚合会话、提交与未提交改动生成日报 | <https://github.com/leonsong09/project-daily-summary> |
| `worktree-closeout` | 跨会话只读巡检 worktree / branch，给出后续操作提示 | <https://github.com/leonsong09/worktree-closeout> |

## 可选：同步终端工具链

macOS + Homebrew + Zsh 用户可以把下面这句话交给 Agent：

> 请阅读并按 <https://raw.githubusercontent.com/kiritoxkiriko/my-workflow/main/TERMINAL-SETUP.md> 配置本机终端；保留已有配置，修改前先备份并展示差异。

[TERMINAL-SETUP.md](./TERMINAL-SETUP.md) 会完成：

- 安装 Yazi、zoxide 和图片 / 视频 / PDF / 压缩包预览依赖。
- 配置 Yazi 目录书签、Solarized Light / Dark 自动切换，并将 Neovim 设为默认文本编辑器。
- 加入 `y` 的目录切换函数与 zoxide 的 `z` / `zi`，移除功能重叠的 jump。
- 为 Neovim 安装 `solarized.nvim`，匹配 Ghostty 的 Solarized Light / Dark。

终端配置独立于 Agent 安装流程。这份剧本不会安装 Neovim Codex 插件；卸载 jump 前仍需用户明确授权。

## 升级 / 卸载

- **规则升级**：对各已安装平台重跑安装剧本 §4，比较差异并备份后同步副本。
- **Skill 升级**：按安装剧本 §6，核对上游和本地改动后，在共享实体目录执行 `git pull --ff-only`；所有平台共用更新后的版本。
- **单个平台停用 Skill**：只移除该平台对应软链接。删除共享实体会影响其他平台，需先核对引用并取得授权。
- **调整分发清单**：同步更新本文 Skill 表与安装剧本 §1 的 `SKILL_REPOS`。
- **终端升级或回滚**：按 [TERMINAL-SETUP.md](./TERMINAL-SETUP.md) §2 / §8 执行，已有配置先备份再合并。

## License & 致谢

- 本仓库自身：MIT。
- [global/AGENTS.md](./global/AGENTS.md)：原始版本来自 [Linux Do](https://linux.do) 用户 **leonsong**，本仓库在其基础上做了个人化增改。
- 5 个 `leonsong09` Skill：MIT，由各自作者维护。

Fork 后可按个人偏好调整全局规则；项目约束继续保留在各仓库自己的 `AGENTS.md` 中。
