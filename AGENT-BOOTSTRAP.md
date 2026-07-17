# AGENT-BOOTSTRAP

> 这份文档是给 **AI agent**（Codex CLI / Claude Code / OpenCode 等）直接执行的安装入口。人类用户请优先看 [`README.md`](./README.md)。

---

## 0. 给 Agent 的元指令

请按本文件 §1 → §2 → §3 → §4 → §5 顺序执行，并在关键节点给用户输出进度（结论先行，符合 [`global/AGENTS.md`](./global/AGENTS.md) 的「工作方式」约束）。

- 默认执行模式：**轻量任务**，不要升级为 brainstorming / writing-plans 流程。
- 每一步执行前先读取本机当前状态（幂等，不要覆盖已有 skill）；**安装依赖前必须先跑 §1.5 预检**，根据结果决定是否跳过 §2 / §3 / §4。
- 仅在用户明确同意后才执行：删除文件、修改 git 配置、操作 `~/.git*` 之外的全局配置。
- 完成后输出一份「安装报告」，包含：目标平台、Superpowers 状态、5 个本地 skill 状态、AGENTS.md 链接位置。

---

## 1. 探测目标平台

按以下优先级判定 `PLATFORM`，并在用户消息中没有明确指定时主动询问一次：

| PLATFORM 取值 | 判定信号（任一命中即可） |
|---|---|
| `codex` | 存在 `~/.codex/`，或当前 agent 自报为 Codex CLI / Codex App |
| `claude` | 存在 `~/.claude/`，或当前 agent 自报为 Claude Code |
| `opencode` | 存在 `~/.config/opencode/opencode.json`，或当前 agent 自报为 OpenCode |
| `agents` | 任何遵循 [agents.md](https://agents.md) 开放标准的 agent，且不属于 codex / claude | 

派生路径（**Codex / Claude 走原生位，其余统一走 `~/.agents/AGENTS.md`**）：

```bash
COMMON_AGENTS_FILE="$HOME/.agents/AGENTS.md"

case "$PLATFORM" in
  codex)
    SKILLS_DIR="$HOME/.codex/skills"
    AGENTS_TARGET="$HOME/.codex/AGENTS.md"
    ;;
  claude)
    SKILLS_DIR="$HOME/.claude/skills"
    AGENTS_TARGET="$HOME/.claude/CLAUDE.md"   # Claude Code 不认 AGENTS.md
    ;;
  opencode)
    SKILLS_DIR="$HOME/.config/opencode/skills"
    AGENTS_TARGET="$COMMON_AGENTS_FILE"
    ;;
  agents)
    SKILLS_DIR="$HOME/.agents/skills"          # 通用 fallback
    AGENTS_TARGET="$COMMON_AGENTS_FILE"
    ;;
  *)
    echo "未识别 PLATFORM=$PLATFORM；请向用户确认目标平台" >&2
    exit 1
    ;;
esac
```

`REPO_DIR` 不再需要——本流程**不要求用户 clone 本仓库**。`AGENTS.md` 通过 raw URL 直接下载到目标位置：

```bash
RAW_AGENTS_URL="https://raw.githubusercontent.com/kiritoxkiriko/my-agent-workflow/main/global/AGENTS.md"
```

---

## 1.5 预检：当前环境已安装资产探测（**安装依赖前必跑**）

在执行 §2 / §3 / §4 之前，先列出当前 agent 已经存在的 Superpowers / 5 个本地 skill / `AGENTS.md`，避免重复安装、覆盖用户已有改动。

```bash
echo "=== Pre-flight Check ==="
echo "Platform  : $PLATFORM"
echo "Skills dir: $SKILLS_DIR"
echo

# 1) Superpowers 是否已装
echo "-- Superpowers --"
SP_PRESENT=0
case "$PLATFORM" in
  codex)
    if ls "$HOME/.codex/plugins/" 2>/dev/null | grep -qi superpowers; then
      echo "[present] codex plugin superpowers"; SP_PRESENT=1
    else
      echo "[absent]  codex plugin superpowers"
    fi ;;
  claude)
    if [ -f "$HOME/.claude/skills/superpowers/SKILL.md" ] \
       || ls "$HOME/.claude/plugins/" 2>/dev/null | grep -qi superpowers; then
      echo "[present] claude superpowers"; SP_PRESENT=1
    else
      echo "[absent]  claude superpowers"
    fi ;;
  opencode|agents)
    if grep -q "superpowers" "$HOME/.config/opencode/opencode.json" 2>/dev/null; then
      echo "[present] opencode plugin superpowers"; SP_PRESENT=1
    else
      echo "[absent]  opencode plugin superpowers"
    fi ;;
esac

# 2) 5 个本地 skill 是否已装
echo
echo "-- Local skills --"
SKILL_MISSING=()
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  if [ -f "$SKILLS_DIR/$repo/SKILL.md" ]; then
    echo "[present] $repo"
  else
    echo "[absent]  $repo"
    SKILL_MISSING+=("$repo")
  fi
done

# 3) AGENTS.md 是否已就位
echo
echo "-- AGENTS.md --"
for f in "$COMMON_AGENTS_FILE" "$AGENTS_TARGET"; do
  [ -z "$f" ] && continue
  if [ -f "$f" ]; then
    echo "[present] $f ($(wc -c < "$f") B)"
  else
    echo "[absent]  $f"
  fi
done
echo "========================="
```

> 探测结果用于决策：
> - `SP_PRESENT=1` → §2 跳过 Superpowers 安装，仅做版本提示。
> - `SKILL_MISSING` 为空 → §3 仅做幂等校验，不再触发 `git clone`。
> - 已存在的 `AGENTS.md` 与上游内容不同 → §4 的 `backup_if_real_file` 会自动备份后再覆盖；如用户在本地手改过，需先确认是否合并后再覆盖。

---

## 2. 安装 Superpowers（主工作流）

> ⚠️ 进入本节前**必须先跑 §1.5**。若 `SP_PRESENT=1`，跳过本节，仅向用户输出「Superpowers 已就位，跳过」。

Superpowers 各平台安装方式不同；agent 可执行的最优策略：

### 2.1 Codex CLI / Codex App

Codex 的 plugin 安装是 **TUI 交互式**，agent 通常无法在子进程中模拟。请输出以下提示让用户手动确认：

```
请在 Codex 交互界面执行：/plugins
然后搜索 superpowers 并选择 Install Plugin。完成后回复继续。
```

可选验证：`ls ~/.codex/plugins/ 2>/dev/null | grep -i superpowers`。

### 2.2 Claude Code

```bash
# 方案 A：让用户在 Claude Code 中输入
echo '请在 Claude Code 输入：/plugin install superpowers@claude-plugins-official'
# 方案 B：直接 clone 到个人 skills（无 plugin 容器，但可被识别）
mkdir -p "$HOME/.claude/skills"
[ -d "$HOME/.claude/skills/superpowers/.git" ] || \
  git clone --depth 1 https://github.com/obra/superpowers.git "$HOME/.claude/skills/superpowers"
```

> 优先用方案 A；方案 B 仅在用户拒绝交互或 plugin marketplace 不可用时降级使用。

### 2.3 OpenCode

```bash
CFG="$HOME/.config/opencode/opencode.json"
mkdir -p "$(dirname "$CFG")"
if [ ! -f "$CFG" ]; then
  echo '{ "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"] }' > "$CFG"
else
  # 已有 opencode.json：提示用户手动合并，避免破坏其他配置
  echo "$CFG 已存在；请确认其中包含：\"superpowers@git+https://github.com/obra/superpowers.git\""
fi
```

> 修改 `opencode.json` 属于 **根配置**，按 [`global/AGENTS.md`](./global/AGENTS.md) 的「安全与变更边界」需用户确认；agent 在已有配置文件时禁止自动覆盖。

---

## 3. 安装 5 个本地扩展 Skill（幂等）

> ⚠️ 进入本节前**必须先跑 §1.5**。脚本自身已幂等（`[ -d "$target/.git" ]` 判断），但建议优先对 `${SKILL_MISSING[@]}` 中缺失的 skill 进行安装，避免对已有 skill 做无谓的网络请求。

```bash
mkdir -p "$SKILLS_DIR"
# 仅安装缺失项（若未跑 §1.5 则退化为全量幂等循环）
if declare -p SKILL_MISSING >/dev/null 2>&1; then
  TARGETS=("${SKILL_MISSING[@]}")
else
  TARGETS=(research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout)
fi
for repo in "${TARGETS[@]}"; do
  target="$SKILLS_DIR/$repo"
  if [ -d "$target/.git" ]; then
    echo "[skip] $repo 已存在：$target"
  else
    git clone --depth 1 "https://github.com/leonsong09/$repo.git" "$target" \
      && echo "[ok]   $repo -> $target" \
      || echo "[fail] $repo 安装失败"
  fi
done
```

更新所有已安装 skill：

```bash
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  target="$SKILLS_DIR/$repo"
  [ -d "$target/.git" ] && git -C "$target" pull --ff-only && echo "[updated] $repo"
done
```

---

## 4. 让 Agent 真正读到 [`global/AGENTS.md`](./global/AGENTS.md)

> ⚠️ 进入本节前**必须先跑 §1.5**。若 §1.5 报告目标位置已存在 `AGENTS.md`，本节的 `backup_if_real_file` 会在内容不同时自动 `*.bak.<timestamp>` 备份；若用户曾在本地手改过该副本，先与用户确认是否需要把改动回流到上游再覆盖。

直接从 GitHub raw URL **下载（不软链、不 clone 仓库）** `AGENTS.md` 到目标位置；每次 `AGENTS.md` 升级后重跑本节即可同步。

```bash
backup_if_real_file() {
  local target="$1" tmp="$2"
  if [ -f "$target" ] && ! cmp -s "$target" "$tmp"; then
    cp "$target" "$target.bak.$(date +%Y%m%d%H%M%S)"
    echo "[backup] $target"
  fi
}

download_agents_md() {
  local target="$1"
  local tmp; tmp="$(mktemp)"
  if curl -fsSL "$RAW_AGENTS_URL" -o "$tmp"; then
    mkdir -p "$(dirname "$target")"
    backup_if_real_file "$target" "$tmp"
    mv -f "$tmp" "$target"
    echo "[copy]   $target"
  else
    rm -f "$tmp"
    echo "[fail]   下载 $RAW_AGENTS_URL 失败"
    return 1
  fi
}

# 4.1 始终先建立通用位（多 agent 共享）
download_agents_md "$COMMON_AGENTS_FILE"

# 4.2 再处理本次平台的原生位（与通用位相同则跳过第二次下载）
if [ "$AGENTS_TARGET" != "$COMMON_AGENTS_FILE" ]; then
  download_agents_md "$AGENTS_TARGET"
fi
```

> Codex / Claude 走原生位（`~/.codex/AGENTS.md` / `~/.claude/CLAUDE.md`），其余 agent（OpenCode 等）走通用位 `~/.agents/AGENTS.md`；当 `PLATFORM=opencode` 或 `agents` 时 §4.2 自动跳过，避免重复下载。
>
> 若 OpenCode 没有自动加载通用位，按需在 `~/.config/opencode/opencode.json` 增加 `{ "instructions": ["~/.agents/AGENTS.md"] }`，或再 `download_agents_md "$HOME/.config/opencode/AGENTS.md"`（修改根配置前需用户确认）。
>
> **同步更新**：每次仓库 `AGENTS.md` 变更后，重跑本节即可。如果发现下游目标被人手动改过，`backup_if_real_file` 会自动备份；按需把备份内容人工合并回上游再重跑。

---

## 5. 自检与安装报告

```bash
echo "=== Install Report ==="
echo "Platform     : $PLATFORM"
echo "Skills dir   : $SKILLS_DIR"
echo "AGENTS file  : $AGENTS_TARGET ($([ -f "$AGENTS_TARGET" ] && echo 'present' || echo 'MISSING'))"
echo
echo "-- Local skills --"
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  f="$SKILLS_DIR/$repo/SKILL.md"
  [ -f "$f" ] && echo "[ok]      $repo" || echo "[missing] $repo"
done
echo
echo "-- Superpowers --"
case "$PLATFORM" in
  codex)    ls "$HOME/.codex/plugins/" 2>/dev/null | grep -qi superpowers && echo "[ok] codex plugin" || echo "[unknown] 请在 Codex 中确认 /plugins" ;;
  claude)   ls "$HOME/.claude/skills/superpowers/SKILL.md" 2>/dev/null && echo "[ok] claude" || echo "[unknown] 请在 Claude Code 中确认 /plugin list" ;;
  opencode) grep -q "superpowers" "$HOME/.config/opencode/opencode.json" 2>/dev/null && echo "[ok] opencode plugin in config" || echo "[missing] 请检查 opencode.json" ;;
esac
```

执行结束后，请把上面的报告原文贴给用户，并提示：

1. 在对应 agent 中说「使用 superpowers 的 brainstorming 帮我开始一个新功能」验证主干。
2. 在对应 agent 中说「总结今天的调研输出笔记」验证 `research-note-wrap`。

---

## 6. Skill 清单与本机状态同步

[`global/AGENTS.md`](./global/AGENTS.md) 只维护跨仓库稳定偏好，不记录 skill 名称或触发规则。Skill 管理遵守以下原则：

- **可用性真相源**：`$SKILLS_DIR/<name>/SKILL.md` 是否存在。
- **分发清单真相源**：本文件 §1.5 / §3 / §5 与 `README.md` 的个人扩展 skill 表；增删或重命名时同步更新这些位置。
- **本地缺失**：提示用户运行 §3 安装，不自动删除文档条目。
- **本地多出**：保留用户自行安装的 skill，不自动删除，也不写入 `global/AGENTS.md`。

状态检测命令：

```bash
for repo in research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout; do
  if [ -f "$SKILLS_DIR/$repo/SKILL.md" ]; then
    echo "[present] $repo"
  else
    echo "[missing] $repo"
  fi
done
```

---

## 7. 安全与边界（Agent 必须遵守）

- 不执行破坏性 git 命令（`reset --hard` / `push --force` / `clean -fdx` 等）。
- 不修改 `.git/` 内部文件，只用 `git` 子命令。
- 不向 [`global/AGENTS.md`](./global/AGENTS.md) 之外的全局配置（`~/.zshrc`、`~/.bashrc`、`~/.gitconfig` 等）写入内容。
- 修改任何已存在的配置文件（`opencode.json`、`AGENTS.md`、`CLAUDE.md`）前先输出 diff 并请求确认。
- 仅在 §3 / §4 中明确列出的目录下创建文件。

---

## 8. 排错速查

| 现象 | 排查 |
|---|---|
| skill 未触发 | 确认 `$SKILLS_DIR/<name>/SKILL.md` 存在且 frontmatter 的 `name` 与目录同名 |
| Codex 找不到 AGENTS.md | 确认 `~/.codex/AGENTS.md` 存在（重跑 §4 即可），并重启 Codex 会话 |
| Claude Code 不读规则 | Claude Code 默认读 `CLAUDE.md`，确认 `~/.claude/CLAUDE.md` 已下载 |
| OpenCode 不加载 superpowers | `opencode run --print-logs "hello" 2>&1 \| grep -i superpowers` 查看插件加载日志 |
| `curl` / `git clone` 报 SSL / proxy | 在命令前 `export GIT_SSL_NO_VERIFY=false` 或为 curl 加 `-k`；或换用 SSH URL / 镜像源 |
