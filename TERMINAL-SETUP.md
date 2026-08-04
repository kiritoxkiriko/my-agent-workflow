# TERMINAL-SETUP

> 面向 macOS + Homebrew + Zsh + Ghostty 的终端工具链安装剧本。配置基于 [lltx 的参考方案](https://gist.github.com/lltx/a61f98fdb761c9af7c5fd6cbfe963842)，并按本仓库当前使用方式适配 Yazi 26.x、zoxide、Neovim 与 Solarized。

---

## 0. 目标与边界

执行完成后，本机应具备：

- Yazi 文件管理器及常用预览依赖，默认文本编辑器为 Neovim。
- `y` 包装函数：退出 Yazi 时把当前 shell 切换到 Yazi 最后所在目录。
- zoxide 的 `z` / `zi` 命令，并移除功能重叠的 `jump`。
- Yazi 根据 Ghostty 上报的背景，在 Solarized Light / Dark 间自动选择。
- Neovim 使用 `solarized.nvim`，随终端背景在 Solarized Light / Dark 间切换。

本剧本不安装 Neovim 的 Codex 插件，也不属于 [`AGENT-BOOTSTRAP.md`](./AGENT-BOOTSTRAP.md) 的 Agent 环境安装范围。

如果由 Agent 执行，必须遵守以下边界：

1. 修改前读取现有 `~/.zshrc`、Yazi 和 Neovim 配置，先备份再合并，禁止无提示覆盖。
2. 卸载 `jump` 和移动 `~/.jump` 前，必须获得用户明确授权。
3. 只修改本文件列出的配置；不要顺手清理 Homebrew 的其他包或用户的 Neovim 插件。

---

## 1. 预检

```bash
uname -s
command -v brew zsh nvim yazi zoxide jump
brew list --versions yazi zoxide jump 2>/dev/null

for f in \
  "$HOME/.zshrc" \
  "$HOME/.config/yazi/yazi.toml" \
  "$HOME/.config/yazi/keymap.toml" \
  "$HOME/.config/yazi/theme.toml" \
  "$HOME/.config/yazi/flavors/solarized-light.yazi/flavor.toml" \
  "$HOME/.config/yazi/flavors/solarized-dark.yazi/flavor.toml" \
  "$HOME/.config/nvim/init.vim"; do
  [ -f "$f" ] && echo "[present] $f" || echo "[absent]  $f"
done
```

要求：

- 操作系统为 macOS，包管理器为 Homebrew，登录 shell 为 Zsh。
- 若目标配置已存在，先保存差异；不要直接用本文示例覆盖。
- 若 `jump` 不存在，跳过 §3 的卸载动作。

---

## 2. 安装 Yazi、zoxide 与预览依赖

```bash
brew install \
  yazi zoxide \
  ffmpegthumbnailer sevenzip jq poppler fd ripgrep fzf \
  resvg imagemagick

brew install --cask font-symbols-only-nerd-font
```

依赖用途：

- `ffmpegthumbnailer`：视频缩略图。
- `sevenzip`：压缩包预览与解压。
- `jq` / `poppler`：JSON / PDF 预览。
- `fd` / `ripgrep` / `fzf`：快速检索与选择。
- `resvg` / `imagemagick`：SVG 和图片处理。
- Symbols Only Nerd Font：补齐 Yazi 图标字符，不改变主要编程字体。

---

## 3. 用 zoxide 替换 jump

先定位旧初始化配置：

```bash
rg -n 'jump shell|(^|[[:space:]])jump([[:space:]]|$)' "$HOME/.zshrc" 2>/dev/null
```

确认后执行：

1. 从 `~/.zshrc` 删除 `eval "$(jump shell)"`，保留其他用户配置。
2. 卸载 Homebrew 的 `jump`：

```bash
brew uninstall jump
```

3. 若存在历史数据库，把它移到废纸篓而不是永久删除：

```bash
if [ -d "$HOME/.jump" ]; then
  mv "$HOME/.jump" "$HOME/.Trash/jump-backup-$(date +%Y%m%d-%H%M%S)"
fi
```

zoxide 的初始化会在 §5 加入 `~/.zshrc`。

---

## 4. 配置 Yazi

配置目录：

```bash
mkdir -p \
  "$HOME/.config/yazi" \
  "$HOME/.config/yazi/flavors/solarized-light.yazi" \
  "$HOME/.config/yazi/flavors/solarized-dark.yazi"
```

已有文件先备份：

```bash
backup_stamp=$(date +%Y%m%d-%H%M%S)
for f in yazi.toml keymap.toml theme.toml; do
  target="$HOME/.config/yazi/$f"
  [ -f "$target" ] && cp "$target" "$target.bak.$backup_stamp"
done

for flavor in solarized-light solarized-dark; do
  target="$HOME/.config/yazi/flavors/$flavor.yazi"
  [ -d "$target" ] && cp -R "$target" "$target.bak.$backup_stamp"
done
```

### 4.1 `~/.config/yazi/yazi.toml`

```toml
# Yazi 26.x overrides, adapted from the referenced terminal setup.

[mgr]
ratio = [1, 2, 5]
sort_by = "natural"
sort_sensitive = false
sort_reverse = false
sort_dir_first = true
linemode = "size"
show_hidden = false
show_symlink = true
scrolloff = 5
mouse_events = ["click", "scroll"]
title_format = "Yazi: {cwd}"

[preview]
wrap = "no"
tab_size = 2
max_width = 600
max_height = 900
image_filter = "lanczos3"
image_quality = 75

[opener]
edit = [
  { run = "nvim %s", desc = "Neovim", block = true, for = "unix" },
  { run = "cursor %s", desc = "Cursor", orphan = true, for = "macos" },
]
open = [
  { run = "open %s", desc = "Open", for = "macos" },
]
reveal = [
  { run = "open -R %s1", desc = "Reveal in Finder", for = "macos" },
]

[open]
prepend_rules = [
  { mime = "text/*", use = ["edit", "open", "reveal"] },
  { mime = "application/{json,ndjson,yaml,toml}", use = ["edit", "open", "reveal"] },
  { url = "*.{json,jsonc,js,jsx,mjs,cjs,ts,tsx,yaml,yml,toml,md}", use = ["edit", "open", "reveal"] },
]
```

`edit` 的第一项是 `nvim`，所以 Yazi 的默认文本编辑器为 Neovim；Cursor 仅保留为 macOS 上的第二选择。

### 4.2 `~/.config/yazi/keymap.toml`

```toml
# Directory bookmarks layered on top of Yazi's default keymap.

[[mgr.prepend_keymap]]
on = ["g", "h"]
run = "cd ~"
desc = "Go to home"

[[mgr.prepend_keymap]]
on = ["g", "c"]
run = "cd ~/.config"
desc = "Go to config"

[[mgr.prepend_keymap]]
on = ["g", "d"]
run = "cd ~/Downloads"
desc = "Go to Downloads"

[[mgr.prepend_keymap]]
on = ["g", "w"]
run = "cd ~/Dev/workspace"
desc = "Go to workspace"

[[mgr.prepend_keymap]]
on = ["g", "D"]
run = "cd ~/Desktop"
desc = "Go to Desktop"

[[mgr.prepend_keymap]]
on = ["g", "t"]
run = "cd /tmp"
desc = "Go to /tmp"
```

如工作区不在 `~/Dev/workspace`，只调整 `g w` 对应的 `run`。

### 4.3 `~/.config/yazi/theme.toml`

```toml
# Select the Solarized flavor that matches Ghostty's reported background.

[flavor]
light = "solarized-light"
dark = "solarized-dark"
```

Yazi 会通过终端背景查询识别当前 Light / Dark，并选择对应 flavor。`theme.toml` 不再放公共颜色覆盖，避免把两个 flavor 又覆盖成同一套颜色。

### 4.4 `~/.config/yazi/flavors/solarized-light.yazi/flavor.toml`

```toml
# Solarized Light palette for Yazi 26.x.

[app]
overall = { bg = "#fdf6e3" }

[mode]
normal_main = { fg = "#fdf6e3", bg = "#268bd2", bold = true }
normal_alt = { fg = "#268bd2", bg = "#eee8d5", bold = true }
select_main = { fg = "#fdf6e3", bg = "#859900", bold = true }
select_alt = { fg = "#859900", bg = "#eee8d5", bold = true }
unset_main = { fg = "#fdf6e3", bg = "#dc322f", bold = true }
unset_alt = { fg = "#dc322f", bg = "#eee8d5", bold = true }

[status]
overall = { fg = "#657b83", bg = "#eee8d5" }
sep_left = { open = "", close = "" }
sep_right = { open = "", close = "" }
perm_type = { fg = "#268bd2" }
perm_read = { fg = "#859900" }
perm_write = { fg = "#b58900" }
perm_exec = { fg = "#dc322f" }
perm_sep = { fg = "#93a1a1" }
progress_label = { fg = "#657b83", bold = true }
progress_normal = { fg = "#268bd2", bg = "#eee8d5" }
progress_error = { fg = "#dc322f", bg = "#eee8d5" }

[filetype]
rules = [
  { mime = "image/*", fg = "#d33682" },
  { mime = "video/*", fg = "#cb4b16" },
  { mime = "audio/*", fg = "#b58900" },
  { mime = "application/{zip,gzip,x-tar,x-bzip2,x-7z-compressed,x-rar,x-xz}", fg = "#dc322f" },
  { mime = "application/pdf", fg = "#2aa198" },
  { mime = "application/*doc*", fg = "#859900" },
  { mime = "application/*sheet*", fg = "#859900" },
  { mime = "application/*presentation*", fg = "#859900" },
  { url = "*", fg = "#657b83" },
  { url = "*/", fg = "#268bd2", bold = true },
]
```

### 4.5 `~/.config/yazi/flavors/solarized-dark.yazi/flavor.toml`

```toml
# Solarized Dark palette for Yazi 26.x.

[app]
overall = { bg = "#002b36" }

[mode]
normal_main = { fg = "#fdf6e3", bg = "#268bd2", bold = true }
normal_alt = { fg = "#268bd2", bg = "#073642", bold = true }
select_main = { fg = "#fdf6e3", bg = "#859900", bold = true }
select_alt = { fg = "#859900", bg = "#073642", bold = true }
unset_main = { fg = "#fdf6e3", bg = "#dc322f", bold = true }
unset_alt = { fg = "#dc322f", bg = "#073642", bold = true }

[status]
overall = { fg = "#839496", bg = "#073642" }
sep_left = { open = "", close = "" }
sep_right = { open = "", close = "" }
perm_type = { fg = "#268bd2" }
perm_read = { fg = "#859900" }
perm_write = { fg = "#b58900" }
perm_exec = { fg = "#dc322f" }
perm_sep = { fg = "#586e75" }
progress_label = { fg = "#839496", bold = true }
progress_normal = { fg = "#268bd2", bg = "#073642" }
progress_error = { fg = "#dc322f", bg = "#073642" }

[filetype]
rules = [
  { mime = "image/*", fg = "#d33682" },
  { mime = "video/*", fg = "#cb4b16" },
  { mime = "audio/*", fg = "#b58900" },
  { mime = "application/{zip,gzip,x-tar,x-bzip2,x-7z-compressed,x-rar,x-xz}", fg = "#dc322f" },
  { mime = "application/pdf", fg = "#2aa198" },
  { mime = "application/*doc*", fg = "#859900" },
  { mime = "application/*sheet*", fg = "#859900" },
  { mime = "application/*presentation*", fg = "#859900" },
  { url = "*", fg = "#839496" },
  { url = "*/", fg = "#268bd2", bold = true },
]
```

---

## 5. 配置 Zsh

把以下区块合并到 `~/.zshrc`。zoxide 初始化应放在 Oh My Zsh / `compinit` 之后，以便注册 `z` / `zi` 补全。

```zsh
# >>> yazi + zoxide >>>
# Start Yazi with `y`; quitting with `q` changes this shell to Yazi's CWD.
function y() {
    local tmp="$(mktemp -t "yazi-cwd.XXXXXX")" cwd
    command yazi "$@" --cwd-file="$tmp"
    IFS= read -r -d '' cwd < "$tmp"
    [ "$cwd" != "$PWD" ] && [ -d "$cwd" ] && builtin cd -- "$cwd"
    command rm -f -- "$tmp"
}

# Keep this after Oh My Zsh/compinit so z/zi completions are registered.
eval "$(zoxide init zsh)"
# <<< yazi + zoxide <<<
```

重新载入当前 shell：

```bash
exec zsh -l
```

---

## 6. 配置 Neovim Solarized Light / Dark

使用 Neovim 原生 package 机制安装主题，不引入插件管理器：

```bash
theme_dir="$HOME/.local/share/nvim/site/pack/themes/start/solarized.nvim"
mkdir -p "$(dirname "$theme_dir")"

if [ -d "$theme_dir/.git" ]; then
  git -C "$theme_dir" pull --ff-only
else
  git clone --depth 1 https://github.com/maxmx03/solarized.nvim.git "$theme_dir"
fi
```

把以下内容合并到 `~/.config/nvim/init.vim`：

```vim
source ~/.vimrc

" Match Ghostty's Solarized Light/Dark appearance. Neovim detects the
" terminal background and reloads this colorscheme when it changes.
set termguicolors
colorscheme solarized
```

Ghostty 对应的动态主题配置为：

```ini
theme = light:iTerm2 Solarized Light,dark:iTerm2 Solarized Dark
```

Yazi 与 Neovim 都根据终端背景在 Solarized Light / Dark 间选择；Ghostty 负责跟随系统外观提供对应背景。

---

## 7. 验证

```bash
yazi --version
yazi --debug 2>&1 | rg 'Dark/light flavor'
zoxide --version
zsh -lic 'type y; type z; type zi'

nvim --headless \
  '+set background=light' \
  '+colorscheme solarized' \
  '+lua print(vim.g.colors_name, vim.o.background)' \
  '+qa'

nvim --headless \
  '+set background=dark' \
  '+colorscheme solarized' \
  '+lua print(vim.g.colors_name, vim.o.background)' \
  '+qa'
```

预期：

- `type y` 显示 shell function，`type z` / `type zi` 显示 zoxide 生成的函数。
- Yazi 输出 `solarized-dark` / `solarized-light` 两个 flavor；实际使用哪一个由终端背景决定。
- 两次 Neovim 验证分别输出 `solarized light` 与 `solarized dark`。
- `command -v jump` 无输出，且 `~/.zshrc` 中不再存在 `jump shell`。
- 在 Yazi 中对文本文件按 Enter 时，第一选择为 Neovim。

Yazi 的图片、视频、PDF 预览建议在 Ghostty 交互会话中人工检查；headless 命令无法验证终端图形协议效果。

---

## 8. 回滚

- Yazi：恢复 `~/.config/yazi/*.bak.<timestamp>` 与 flavor 目录备份，或移走新增的三个 TOML 文件和两个 Solarized flavor 目录。
- Zsh：删除 `# >>> yazi + zoxide >>>` 到 `# <<< yazi + zoxide <<<` 区块并重启 shell。
- zoxide：`brew uninstall zoxide`。如需恢复 jump，重新 `brew install jump` 并恢复其初始化行与废纸篓中的数据库。
- Neovim：移除 `colorscheme solarized` 配置，并把 `solarized.nvim` 目录移到废纸篓。
