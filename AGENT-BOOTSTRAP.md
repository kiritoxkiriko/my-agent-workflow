# AGENT-BOOTSTRAP

> 给 Codex CLI / Codex App、Claude Code、OpenCode 等 Agent 执行的安装入口。人类操作说明见 [README.md](./README.md)。

## 0. 执行约定

按 §1 → §2 → §3 → §4 → §5 执行。在同一个 Bash 会话中运行代码块，保留变量与函数；不要直接把 Markdown 文件交给 shell 执行。

- 安装范围：5 个个人扩展 Skill、目标平台的软链接、通用规则和目标平台规则副本。
- Skill 实体统一保存在 `~/.agents/skills`，各平台共用同一份文件。
- 先预检，再安装。已有 Skill、独立目录、错误软链接均不得直接覆盖；有冲突时先报告具体路径，保留现场。
- 规则文件有差异时，先展示 diff，结合本次用户授权决定是否合并或覆盖；覆盖前备份。
- 不要求 clone 本仓库。规则从 raw URL 下载，Skill 从各自上游仓库 clone。
- 完成报告需区分：文件已就位、链接已验证、Agent 会话是否实际识别。

## 1. 目标平台与预检

优先采用用户指定的平台；未指定时可使用当前 Agent 对应平台，无法判断时再询问。以下示例默认 `codex`，也支持 `claude`、`opencode`、`agents`。

```bash
PLATFORM=codex
COMMON_SKILLS_DIR="$HOME/.agents/skills"
COMMON_AGENTS_FILE="$HOME/.agents/AGENTS.md"
RAW_AGENTS_URL="https://raw.githubusercontent.com/kiritoxkiriko/my-workflow/main/global/AGENTS.md"
SKILL_REPOS=(research-note-wrap session-wrap commit-daily-summary project-daily-summary worktree-closeout)

case "$PLATFORM" in
  codex)
    SKILLS_DIR="$HOME/.codex/skills"
    AGENTS_TARGET="$HOME/.codex/AGENTS.md"
    ;;
  claude)
    SKILLS_DIR="$HOME/.claude/skills"
    AGENTS_TARGET="$HOME/.claude/CLAUDE.md"
    ;;
  opencode)
    SKILLS_DIR="$HOME/.config/opencode/skills"
    AGENTS_TARGET="$COMMON_AGENTS_FILE"
    ;;
  agents)
    SKILLS_DIR="$COMMON_SKILLS_DIR"
    AGENTS_TARGET="$COMMON_AGENTS_FILE"
    ;;
  *) echo "[fail] 未识别的平台：$PLATFORM" >&2; exit 1 ;;
esac

for repo in "${SKILL_REPOS[@]}"; do
  for root in "$COMMON_SKILLS_DIR" "$HOME/.codex/skills" "$HOME/.claude/skills" "$HOME/.config/opencode/skills"; do
    target="$root/$repo"
    if [ -L "$target" ]; then
      echo "[link] $target -> $(readlink "$target")"
    elif [ -e "$target" ]; then
      echo "[existing] $target"
    fi
  done
done
for target in "$COMMON_AGENTS_FILE" "$AGENTS_TARGET"; do
  if [ -e "$target" ] || [ -L "$target" ]; then
    echo "[existing] $target"
  else
    echo "[missing] $target"
  fi
done
```

预检发现平台目录中有独立 Skill 安装时，先检查它的上游、提交和未提交改动。若需要迁移，先确认要保留的版本并备份，再将实体迁入共享目录，最后建立软链接。下面脚本会阻止直接覆盖，也不会自动迁移旧目录。

## 2. 安装共享 Skill

```bash
install_shared_skills() {
  local repo target root legacy
  # 先检查整份清单，发现冲突就停止，避免安装到一半才发现旧目录。
  for repo in "${SKILL_REPOS[@]}"; do
    target="$COMMON_SKILLS_DIR/$repo"
    if [ -L "$target" ]; then
      echo "[conflict] 共享实体位置是软链接：$target" >&2
      return 1
    elif [ -e "$target" ]; then
      if [ ! -f "$target/SKILL.md" ]; then
        echo "[conflict] 已有路径缺少 SKILL.md：$target" >&2
        return 1
      fi
    else
      for root in "$HOME/.codex/skills" "$HOME/.claude/skills" "$HOME/.config/opencode/skills"; do
        legacy="$root/$repo"
        if [ -e "$legacy" ] || [ -L "$legacy" ]; then
          echo "[conflict] 请先核对并迁移已有安装：$legacy" >&2
          return 1
        fi
      done
    fi
  done
  mkdir -p "$COMMON_SKILLS_DIR" || return 1
  for repo in "${SKILL_REPOS[@]}"; do
    target="$COMMON_SKILLS_DIR/$repo"
    if [ -f "$target/SKILL.md" ]; then
      echo "[skip] $target"
    else
      git clone --depth 1 "https://github.com/leonsong09/$repo.git" "$target" || return 1
      [ -f "$target/SKILL.md" ] || { echo "[fail] 缺少 SKILL.md：$target" >&2; return 1; }
    fi
  done
}
install_shared_skills || exit 1
```

已有共享 Skill 不自动更新，避免改变其他 Agent 正在共用的版本。升级方式见 §6。

## 3. 为目标平台建立软链接

`agents` 平台直接使用共享目录，跳过链接创建。其他平台只在路径不存在时创建链接；已有独立目录、错误链接或失效链接都保留并报错。

```bash
link_platform_skills() {
  local repo source target
  [ "$SKILLS_DIR" = "$COMMON_SKILLS_DIR" ] && return 0
  for repo in "${SKILL_REPOS[@]}"; do
    source="$COMMON_SKILLS_DIR/$repo"
    target="$SKILLS_DIR/$repo"
    [ -f "$source/SKILL.md" ] || { echo "[fail] 缺少共享 Skill：$source" >&2; return 1; }
    if [ -e "$target" ] || [ -L "$target" ]; then
      if [ ! -L "$target" ] || [ ! "$target" -ef "$source" ]; then
        echo "[conflict] 保留已有路径，请先处理：$target" >&2
        return 1
      fi
    fi
  done
  mkdir -p "$SKILLS_DIR" || return 1
  for repo in "${SKILL_REPOS[@]}"; do
    target="$SKILLS_DIR/$repo"
    if [ ! -L "$target" ]; then
      ln -s "$COMMON_SKILLS_DIR/$repo" "$target" || return 1
    fi
    echo "[ok] $target -> $COMMON_SKILLS_DIR/$repo"
  done
}
link_platform_skills || exit 1
```

安装另一个平台时，用新的 `PLATFORM` 重跑 §1–§5；已有共享实体会跳过，只补充该平台的链接与规则。

## 4. 同步全局规则

先下载到临时文件，与通用位及目标平台规则比较。确认差异在用户授权范围内后，再运行第二个代码块；若需要保留本地规则，先合并，不能直接覆盖。

```bash
RULES_TMP="$(mktemp)"
curl -fsSL "$RAW_AGENTS_URL" -o "$RULES_TMP" || { rm -f "$RULES_TMP"; exit 1; }
[ -s "$RULES_TMP" ] || { rm -f "$RULES_TMP"; echo "[fail] 下载内容为空" >&2; exit 1; }
for target in "$COMMON_AGENTS_FILE" "$AGENTS_TARGET"; do
  if [ -f "$target" ]; then
    diff -u "$target" "$RULES_TMP"
  fi
done
# diff 返回 1 表示存在差异；此处不要启用 set -e。
```

```bash
copy_rules() {
  local target="$1" backup
  if [ -L "$target" ] || { [ -e "$target" ] && [ ! -f "$target" ]; }; then
    echo "[conflict] 规则位置不是普通文件：$target" >&2
    return 1
  fi
  if [ -f "$target" ] && cmp -s "$target" "$RULES_TMP"; then
    echo "[skip] $target"
    return 0
  fi
  mkdir -p "$(dirname "$target")" || return 1
  if [ -f "$target" ]; then
    backup="$(mktemp "$target.bak.XXXXXX")" || return 1
    cp -p "$target" "$backup" || return 1
    echo "[backup] $backup"
  fi
  cp "$RULES_TMP" "$target" || return 1
  echo "[copy] $target"
}
copy_rules "$COMMON_AGENTS_FILE" || exit 1
if [ "$AGENTS_TARGET" != "$COMMON_AGENTS_FILE" ]; then
  copy_rules "$AGENTS_TARGET" || exit 1
fi
rm -f "$RULES_TMP"
```

OpenCode 还需在 `~/.config/opencode/opencode.json` 的 `instructions` 中引用 `~/.agents/AGENTS.md`。先读取已有配置，保留其他键和已有 instructions，只合并下面条目，不用示例覆盖整个文件：

```json
{ "instructions": ["~/.agents/AGENTS.md"] }
```

规则副本与 Skill 软链接分别管理。规则更新需对各已安装平台重跑本节，确保通用位、Codex 和 Claude 副本一致。

## 5. 验证与安装报告

```bash
verify_install() {
  local repo source target failed=0
  for repo in "${SKILL_REPOS[@]}"; do
    source="$COMMON_SKILLS_DIR/$repo"
    target="$SKILLS_DIR/$repo"
    if [ ! -f "$source/SKILL.md" ] || [ ! -f "$target/SKILL.md" ]; then
      echo "[missing] $repo"; failed=1
    elif [ "$SKILLS_DIR" != "$COMMON_SKILLS_DIR" ] && { [ ! -L "$target" ] || [ ! "$target" -ef "$source" ]; }; then
      echo "[conflict] $target 未链接到共享实体"; failed=1
    else
      echo "[ok] $repo"
    fi
  done
  if [ -s "$COMMON_AGENTS_FILE" ] && [ -s "$AGENTS_TARGET" ] && cmp -s "$COMMON_AGENTS_FILE" "$AGENTS_TARGET"; then
    echo "[ok] 通用规则与目标平台规则一致"
  else
    echo "[fail] 规则缺失或内容不同"; failed=1
  fi
  return "$failed"
}
verify_install || exit 1
```

报告包含目标平台、共享实体目录、平台链接目录、规则文件位置及验证结果。OpenCode 另需确认 instructions 引用已合并。重新打开目标 Agent 会话，检查 Skill 清单，或用「总结今天的调研输出笔记」验证触发；只检查文件不能证明会话已加载。

## 6. 升级与清单维护

分发清单真相源是 §1 的 `SKILL_REPOS` 与 README 的个人扩展 Skill 表。全局规则不维护 Skill 清单；用户自行安装的其他 Skill 保持原样。

升级会同时影响所有引用共享目录的 Agent。用户要求升级后，先查看各仓库的 origin、当前分支和 `git status --short`；确认来源正确、工作区干净，再逐个执行：

```bash
for repo in "${SKILL_REPOS[@]}"; do
  target="$COMMON_SKILLS_DIR/$repo"
  if [ -d "$target/.git" ]; then
    git -C "$target" pull --ff-only || exit 1
  else
    echo "[skip] 非 Git 安装，保留原样：$target"
  fi
done
```

只对某个平台停用 Skill 时，移除该平台对应的软链接即可；删除共享实体会影响所有平台，必须先核对引用并取得授权。不要对平台 Skill 路径使用递归删除作为默认卸载方式。

## 7. 安全边界

- 不执行破坏性 Git 命令，不直接修改 `.git/` 内部文件。
- 修改范围限于本文件列出的 Skill、软链接和规则文件；OpenCode 的 instructions 按授权合并。
- 不修改 shell、Git 或其他全局配置，不安装或卸载其他插件。
- 不自动覆盖或删除已有安装，不把私人路径或凭证写入分发文档。

## 8. 排错

| 现象 | 排查 |
|---|---|
| Skill 未触发 | 检查共享目录的 `SKILL.md`、平台软链接和 Agent 会话的实际 Skill 清单 |
| 旧独立目录阻塞安装 | 对比 origin、提交和本地改动，确认保留版本并备份后迁移 |
| 链接失效或指向错误 | 保留现场，核对共享实体位置，再按用户授权修复链接 |
| Codex / Claude 未读取规则 | 检查各自的 `AGENTS.md` / `CLAUDE.md`，重新打开会话 |
| OpenCode 未读取通用规则 | 检查配置中的 instructions 是否引用通用规则 |
| 下载或 clone 失败 | 检查网络、代理、证书与仓库访问权限，保持 TLS 校验开启 |
