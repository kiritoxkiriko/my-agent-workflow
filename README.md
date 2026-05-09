# my-workflow

> 跨平台个人 AI Coding Agent 工作流：一份 `AGENTS.md` 规则 + 一组 Codex Skill，统一你的 **Codex CLI / Claude Code / OpenCode** 行为。

- 仓库地址：<https://github.com/kiritoxkiriko/my-agent-workflow>
- 作者偏好：简体中文沟通、Superpowers 主干、最短路径优先、轻量任务直接干。

---

## 这是什么？

`my-workflow` 把我对 AI coding agent 的所有「硬约束 / 默认偏好 / 触发流程」沉淀成两类资产：

| 资产 | 文件 / 仓库 | 作用 |
|---|---|---|
| 规则 | [`global/AGENTS.md`](./global/AGENTS.md) | 单文件全局规则：指令优先级、轻量任务策略、并行准入、commit 规范、沟通风格……<br/>来自 [Linux Do](https://linux.do) 用户 **leonsong**，本仓库在其基础上增改 |
| 主工作流 skills | [`obra/superpowers`](https://github.com/obra/superpowers) | brainstorming / writing-plans / executing-plans / TDD / code-review / worktrees… |
| 个人扩展 skills | 5 个 [`leonsong09/*`](https://github.com/leonsong09) 仓库 | 调研笔记、会话收尾、提交日报、项目日报、worktree 收口 |

读完 [`global/AGENTS.md`](./global/AGENTS.md) 你就能知道我希望下游 agent 在什么时机做什么事；读 [`AGENT-BOOTSTRAP.md`](./AGENT-BOOTSTRAP.md) 你（或一个 agent）可以一键把这套环境复刻到本机；本仓库根目录的 [`AGENTS.md`](./AGENTS.md) 仅服务于在本仓库内迭代这套 workflow 的 agent，与下游用户无关。

---

## 核心理念（30 秒速读）

- **Superpowers 是主干**：`brainstorming → writing-plans → implementation → review → verification` 的纪律层。
- **不强制 full Superpowers**：轻量任务（小 bug、文案、配置）默认走最短路径，不要把 1 行 fix 升级成 5 步流程。
- **真相源唯一**：本机 `~/.codex/skills/` 之类的目录决定 skill 是否可用，[`global/AGENTS.md`](./global/AGENTS.md) 仅承载引用与触发说明。
- **沟通**：默认简体中文 + 英文术语；结论先行，再补依据与权衡。
- **安全**：无破坏性 git 命令、不操作 `.git`、不硬编码密钥。

---

## 我支持哪些 Agent？

规则分发采用「**两个原生位 + 一个通用位**」策略：

- **原生位**：Codex、Claude Code 保留各自官方约定的文件名，因为它们不认别的文件。
- **通用位**：`~/.agents/AGENTS.md`，给所有遵循 [agents.md](https://agents.md) 开放标准的 agent 使用（OpenCode、未来的新 agent 等）。

| Agent | Skill 安装位置 | 规则文件位置 | Superpowers 安装方式 |
|---|---|---|---|
| **Codex CLI / Codex App** | `~/.codex/skills/<name>/SKILL.md` | `~/.codex/AGENTS.md` | 在 Codex 内 `/plugins` → 搜索 `superpowers` → 安装 |
| **Claude Code** | `~/.claude/skills/<name>/SKILL.md` | `~/.claude/CLAUDE.md` | `/plugin install superpowers@claude-plugins-official` |
| **OpenCode / 其他 AGENTS.md 兼容 agent** | `~/.config/opencode/skills/<name>/SKILL.md`（或其等价目录） | `~/.agents/AGENTS.md`（OpenCode 需在 `opencode.json` 用 `instructions` 字段引用） | 在 `opencode.json` 加 `"plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]` |

> 三家 skill 目录格式一致（`<dir>/SKILL.md` + YAML frontmatter），所以 5 个 `leonsong09/*` skill 用同一套 `git clone` 流程即可适配。

---

## 快速开始（人类操作版）

> ✨ **不需要 clone 本仓库**。`AGENTS.md` 和 5 个 skill 都按需直接从各自的 raw URL 下载，本仓库本体只是这些资产的索引与文档。

### 1. 下载 `AGENTS.md` 到目标位置

```bash
RAW="https://raw.githubusercontent.com/kiritoxkiriko/my-agent-workflow/main/global/AGENTS.md"

# 通用位（OpenCode 等 AGENTS.md 兼容 agent 都从这里读）
mkdir -p "$HOME/.agents" && curl -fsSL "$RAW" -o "$HOME/.agents/AGENTS.md"

# Codex 原生位（仅 Codex 用户需要执行）
mkdir -p "$HOME/.codex" && curl -fsSL "$RAW" -o "$HOME/.codex/AGENTS.md"

# Claude Code 原生位（仅 Claude Code 用户需要执行；Claude Code 默认读 CLAUDE.md）
mkdir -p "$HOME/.claude" && curl -fsSL "$RAW" -o "$HOME/.claude/CLAUDE.md"
```

> 升级时重跑同样的 `curl` 命令即可。

### 2. 安装 Superpowers（按你用的 agent 选一个）

- **Codex CLI**：在交互界面输入 `/plugins`，搜索 `superpowers` 并安装。
- **Claude Code**：在交互界面输入 `/plugin install superpowers@claude-plugins-official`。
- **OpenCode**：编辑 `~/.config/opencode/opencode.json`，把 `superpowers` 加入 `plugin` 数组后重启 OpenCode。

### 3. 安装 5 个个人扩展 skill（任何平台都执行；用 `PLATFORM` 切换目标目录）

```bash
PLATFORM=codex   # 可选：codex | claude | opencode
case "$PLATFORM" in
  codex)    DEST="$HOME/.codex/skills" ;;
  claude)   DEST="$HOME/.claude/skills" ;;
  opencode) DEST="$HOME/.config/opencode/skills" ;;
esac

mkdir -p "$DEST"
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  if [ -d "$DEST/$repo/.git" ]; then
    echo "[skip] $repo 已存在"
  else
    git clone --depth 1 "https://github.com/leonsong09/$repo.git" "$DEST/$repo"
  fi
done
```

> 这一步用 `git clone --depth 1` 而不是 `curl`，因为每个 skill 仓库可能包含 `SKILL.md` + 脚本 + 模板等多文件，需要保留目录结构；后续 `git pull` 也方便。

### 4. OpenCode 的额外一步（仅 OpenCode 用户）

让 `~/.agents/AGENTS.md` 真正被 OpenCode 加载，二选一：

```jsonc
// 方式 A（推荐）：在 ~/.config/opencode/opencode.json 通过 instructions 引用
{ "instructions": ["~/.agents/AGENTS.md"] }
```

```bash
# 方式 B：再下一份到 OpenCode 的全局位
mkdir -p "$HOME/.config/opencode"
curl -fsSL "$RAW" -o "$HOME/.config/opencode/AGENTS.md"
```

### 5. 验证

```bash
# 通用：检查 SKILL.md 是否就位
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  f="$DEST/$repo/SKILL.md"
  [ -f "$f" ] && echo "[ok] $repo" || echo "[missing] $repo"
done
```

在对应 agent 中说一句「总结今天的调研输出笔记」或「会话收尾」，命中触发词即代表安装成功。

---

## 快速开始（让 Agent 自己装）

把这一句话直接发给一个全新的 agent：

> 请阅读并按 <https://raw.githubusercontent.com/kiritoxkiriko/my-agent-workflow/main/AGENT-BOOTSTRAP.md> 完成本机安装；目标平台是 `codex`（或 `claude` / `opencode`）。

agent 会按 [`AGENT-BOOTSTRAP.md`](./AGENT-BOOTSTRAP.md) 的步骤完成 Superpowers + 5 个本地 skill + AGENTS.md 下载 + 自检。**全程不需要 clone 本仓库。**

---

## 我提供的 5 个个人扩展 Skill

| Skill | 触发场景 | 上游仓库 |
|---|---|---|
| `research-note-wrap` | 把调研 / 分析整理为 Obsidian 风格 Markdown 笔记 | <https://github.com/leonsong09/research-note-wrap> |
| `session-wrap` | 把当前 coding 会话压缩成结论 / 验证 / 风险 / 下一步 | <https://github.com/leonsong09/session-wrap> |
| `commit-daily-summary` | 把一天的 git 提交聚合为中文日报 | <https://github.com/leonsong09/commit-daily-summary> |
| `project-daily-summary` | 按项目聚合 sessions / commits / 未提交改动出日报 | <https://github.com/leonsong09/project-daily-summary> |
| `worktree-closeout` | 跨会话只读巡检 worktree / branch，给出后续 prompt | <https://github.com/leonsong09/worktree-closeout> |

> Superpowers 自带的 14 个主干 skill（brainstorming / writing-plans / executing-plans / subagent-driven-development / dispatching-parallel-agents / using-git-worktrees / systematic-debugging / requesting-code-review / receiving-code-review / verification-before-completion / finishing-a-development-branch / test-driven-development / using-superpowers / writing-skills）由 [obra/superpowers](https://github.com/obra/superpowers) 提供，按上面 §2 安装即可，无需单独 clone。

---

## 文档导航

| 给谁看 | 文件 | 内容 |
|---|---|---|
| 👤 人类 | [`README.md`](./README.md)（本文） | 项目概览、理念、各 agent 快速开始 |
| 🤖 下游 Agent | [`AGENT-BOOTSTRAP.md`](./AGENT-BOOTSTRAP.md) | 单段提示 + 一键执行脚本 + 下载 `global/AGENTS.md` 流程 |
| 👤 + 🤖 下游 | [`global/AGENTS.md`](./global/AGENTS.md) | **要分发的全局规则**（会被 `curl` 下载到本机各 agent 规则位） |
| 🛠️ 本仓库维护者 / agent | [`AGENTS.md`](./AGENTS.md) | **项目级规则**：在本仓库迭代 workflow 时 agent 必读；定义改谁、自检、commit 规范 |

---

## 升级 / 卸载

- **升级 `AGENTS.md`**：重跑 §1 的 `curl` 命令即可拉到最新版。
- **升级 Superpowers**：按各 agent 自带方式（Codex `/plugins`、Claude Code `/plugin update`、OpenCode 重新解析 plugin 包）。
- **升级 5 个本地 skill**：`AGENT-BOOTSTRAP.md` §3 提供 `git pull` 循环。
- **卸载某个 skill**：`rm -rf "<DEST>/<skill-name>"`，并在 [`global/AGENTS.md`](./global/AGENTS.md) 「技能（Skills）」章节移除对应行。

---

## License & 致谢

- 本仓库自身：MIT。
- [obra/superpowers](https://github.com/obra/superpowers)：MIT，本仓库的工作流主干。
- [`global/AGENTS.md`](./global/AGENTS.md)：原始版本来自 [Linux Do](https://linux.do) 用户 **leonsong**，本仓库在其基础上做了个人化增改（致谢 🙏）。
- 5 个 `leonsong09/*` skill：MIT，由其作者维护。

如果你 fork 后做了改动，欢迎在 issues 交流；但请记得 [`global/AGENTS.md`](./global/AGENTS.md) 是高度个人化的偏好集合，建议从最小子集开始适配。
