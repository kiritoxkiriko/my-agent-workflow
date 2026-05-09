# my-workflow

本仓库托管个人 `AGENTS.md`（全局 Agent 规则）与配套的 Codex skill 安装指引。

- 仓库地址：<https://github.com/kiritoxkiriko/my-agent-workflow>
- 推荐克隆位置：`~/Dev/workspace/my-workflow`

```bash
git clone https://github.com/kiritoxkiriko/my-agent-workflow.git ~/Dev/workspace/my-workflow
```

[AGENTS.md](./AGENTS.md) 中以「主干整合」与「本地扩展」形式引用了若干 skill，其中 5 个本地扩展 skill **不属于 Superpowers 官方仓库**，需要单独安装到 `~/.codex/skills/` 才能命中。本 README 用作 agent 与人类操作者的安装入口。

---

## 一、缺失 skill 总览（与 AGENTS.md 对齐）

下列 skill 在 [AGENTS.md](./AGENTS.md) 「技能（Skills）」章节被引用，但默认未安装，agent 在命中触发词时会失败：

| Skill 名称 | 触发场景（来自 AGENTS.md / 上游 SKILL.md） | 上游仓库 | 本地目标路径 |
|---|---|---|---|
| `research-note-wrap` | 调研总结 / 输出笔记（`总结调研`、`输出结论`、`总结分析`、`输出笔记`、`调研纪要`、`分析纪要`、`会话结论`） | https://github.com/leonsong09/research-note-wrap | `~/.codex/skills/research-note-wrap` |
| `session-wrap` | 会话收尾（汇总产出、验证、风险、下一步） | https://github.com/leonsong09/session-wrap | `~/.codex/skills/session-wrap` |
| `commit-daily-summary` | 提交总结 / 日报（按 git 提交聚合中文日报） | https://github.com/leonsong09/commit-daily-summary | `~/.codex/skills/commit-daily-summary` |
| `project-daily-summary` | 项目级日报（按项目聚合 sessions / commits / 未提交改动） | https://github.com/leonsong09/project-daily-summary | `~/.codex/skills/project-daily-summary` |
| `worktree-closeout` | worktree / branch / parallel 收口（只读巡检 + 后续 prompt） | https://github.com/leonsong09/worktree-closeout | `~/.codex/skills/worktree-closeout` |

> 上游仓库结构均为 `SKILL.md` 位于仓库根目录，`name` 字段与仓库名一致，因此「克隆到 `~/.codex/skills/<repo-name>`」即可被 Codex 识别。

未列入此表的 skill（`brainstorming` / `writing-plans` / `executing-plans` / `subagent-driven-development` / `dispatching-parallel-agents` / `using-git-worktrees` / `systematic-debugging` / `requesting-code-review` / `receiving-code-review` / `verification-before-completion` / `finishing-a-development-branch` / `test-driven-development` / `using-superpowers` / `writing-skills`）均由 [obra/superpowers](https://github.com/obra/superpowers) 提供，按上游 README 安装 Superpowers 插件即可，本 README 不重复。

---

## 二、给 Agent 的安装指南（可直接执行）

> 适用前提：本机已存在 `~/.codex/`（Codex 默认配置目录）。
> 默认操作均为 **加性**（`git clone` / `git pull`），不会改写既有 skill 内容；仅在用户明确同意时才覆盖。

### 2.1 一键安装全部缺失 skill（幂等）

Agent 可直接执行下面这段命令；已存在的目录会被跳过，缺失的会被克隆：

```bash
mkdir -p ~/.codex/skills
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  target="$HOME/.codex/skills/$repo"
  if [ -d "$target/.git" ]; then
    echo "[skip] $repo 已存在：$target"
  else
    git clone --depth 1 "https://github.com/leonsong09/$repo.git" "$target" \
      && echo "[ok]   $repo -> $target" \
      || echo "[fail] $repo 安装失败"
  fi
done
```

### 2.2 一键更新全部已安装 skill

```bash
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  target="$HOME/.codex/skills/$repo"
  if [ -d "$target/.git" ]; then
    git -C "$target" pull --ff-only && echo "[updated] $repo" || echo "[fail] $repo 更新失败"
  else
    echo "[missing] $repo 未安装，跳过；请先执行 2.1 安装命令"
  fi
done
```

### 2.3 安装后自检（检查 SKILL.md 是否存在）

```bash
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  f="$HOME/.codex/skills/$repo/SKILL.md"
  if [ -f "$f" ]; then echo "[ok] $repo"; else echo "[missing] $repo"; fi
done
```

### 2.4 卸载单个 skill（仅在用户明确要求时执行）

```bash
# 将 <skill-name> 替换为实际 skill 名
rm -rf "$HOME/.codex/skills/<skill-name>"
```

---

## 三、自动更新 AGENTS.md（与本地实际状态保持一致）

[AGENTS.md](./AGENTS.md) 是 agent 的真相源。安装 / 卸载完成后，agent 应执行以下同步动作，避免「文档说有、本地实际没有」或反之：

### 3.1 同步原则

1. **真相源**：`~/.codex/skills/<name>/SKILL.md` 是否存在 = 该 skill 是否可用。
2. **AGENTS.md 仅承载引用与触发说明**，不应承载安装状态以外的元信息。
3. 当本地状态与 AGENTS.md 不一致时，按以下优先级处理：
   - 本地存在、AGENTS.md 缺引用 → 在 [AGENTS.md](./AGENTS.md) 「技能（Skills）」章节追加一行（保持现有排版风格）。
   - 本地缺失、AGENTS.md 已引用 → **不要**自动删除引用；改为提示用户运行 §2.1 安装，或经用户同意后注释掉对应行。
   - 本地与 AGENTS.md 一致 → 不修改。

### 3.2 Agent 同步流程（推荐顺序）

Agent 收到「同步 AGENTS.md」请求时按以下步骤执行：

1. 读取 [AGENTS.md](./AGENTS.md) 「技能（Skills）」章节，提取所有以反引号包裹的 skill 名。
2. 读取 `~/.codex/skills/` 下所有子目录，校验是否含 `SKILL.md`。
3. 对比两侧得到差集：`only_in_agents_md`（缺安装）、`only_in_local`（缺引用）。
4. 输出差异表给用户，并给出建议动作（不在用户确认前直接修改 AGENTS.md）。
5. 用户确认后，使用编辑工具最小化修改 [AGENTS.md](./AGENTS.md)：
   - 新增引用：在「技能（Skills）」列表末尾追加一行 `- <用途简述>：\`<skill-name>\``。
   - 不要触碰其他章节、不要重排顺序、不要附加说明性段落。

### 3.3 一行差异检测命令（agent 可直接调用）

下面命令打印「AGENTS.md 引用了但本地缺失」的 skill 名清单，作为同步前的快速诊断：

```bash
agents_md="$(pwd)/AGENTS.md"
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  in_doc=$(grep -c "\`$repo\`" "$agents_md" || true)
  has_local=$([ -f "$HOME/.codex/skills/$repo/SKILL.md" ] && echo 1 || echo 0)
  if [ "$in_doc" -gt 0 ] && [ "$has_local" -eq 0 ]; then
    echo "[need-install] $repo"
  fi
  if [ "$in_doc" -eq 0 ] && [ "$has_local" -eq 1 ]; then
    echo "[need-doc]     $repo"
  fi
done
```

---

## 四、安全与边界

- 上述命令仅写入 `~/.codex/skills/` 与本仓库 [AGENTS.md](./AGENTS.md)；不修改任何应用代码、git 历史或全局配置。
- 不引入额外依赖（仅使用系统 `git` 与 `bash`）。
- Agent 在执行 §3 的写文档动作前，必须先输出差异并等待用户确认，符合 [AGENTS.md](./AGENTS.md) 中「文档维护」与「Safety Rules」的约束。
- 上游仓库均为公开 MIT 仓库；安装即克隆源码，便于审阅其 `SKILL.md`。

---

## 五、新增 / 移除一个本地扩展 skill 时

1. 在本 README 的 §2.1 / §2.2 / §2.3 / §3.3 命令中，将该 skill 名加入 / 移出 `for repo in ... ; do` 列表。
2. 在 [AGENTS.md](./AGENTS.md) 「技能（Skills）」章节同步增删一行。
3. 运行 §3.3 自检，确认两侧一致。
