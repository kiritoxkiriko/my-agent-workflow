# AGENTS.md（项目级规则）

> **本文件是仓库 `kiritoxkiriko/my-agent-workflow` 的项目级 Agent 规则。**
>
> Codex / Claude Code / OpenCode 进入本仓库工作时会优先读取本文件；其后再叠加用户的全局规则（用户已安装到 `~/.agents/AGENTS.md` / `~/.codex/AGENTS.md` / `~/.claude/CLAUDE.md`）。
>
> ⚠️ 角色区分：
> - 本文件 = **项目级规则**：仅服务于「在本仓库内迭代 workflow 资产」的 agent。
> - [`global/AGENTS.md`](./global/AGENTS.md) = **要分发给下游用户的全局规则**：通过 raw URL 被 `curl` 下载到本机各 agent 规则位。
> - 二者更新触发条件不同，**永远不要写到一起**。

---

## 0. 优先级与适用范围

1. 用户当前会话的明确要求
2. 本 `AGENTS.md`（项目级）
3. [`global/AGENTS.md`](./global/AGENTS.md) 中适用于所有仓库的稳定个人偏好

适用受众：在本仓库工作的 agent；或人类维护者按本文件做开发自检。

---

## 1. 仓库内容物地图

| 路径 | 角色 | 是否对外分发 |
|---|---|---|
| [`global/AGENTS.md`](./global/AGENTS.md) | 全局规则真相源；通过 raw URL 被下游 `curl` 下载到 `~/.agents/AGENTS.md` 等 | ✅ |
| [`AGENT-BOOTSTRAP.md`](./AGENT-BOOTSTRAP.md) | 给新 agent 直接读取并执行的安装剧本 | ✅ raw URL |
| [`README.md`](./README.md) | 给人类的项目导览 + 快速开始 | 仅 GitHub 阅读 |
| [`AGENTS.md`](./AGENTS.md)（本文件） | 项目级 agent 迭代规则；仅本仓库内生效 | ❌ |
| [`LICENSE`](./LICENSE) | MIT | ✅ |
| [`.gitignore`](./.gitignore) | macOS / 编辑器 / 备份残留 | ✅ |

> 不要把全局规则写进本文件；不要把项目级流程写进 `global/AGENTS.md`。

---

## 2. 何时改哪个文件

| 触发情境 | 该改哪个文件 | 同步要点 |
|---|---|---|
| 新增 / 删除 / 重命名一个 skill | `README.md` §「我提供的 5 个个人扩展 Skill」表 + `AGENT-BOOTSTRAP.md` §3 / §6 的 `for repo in ...` 列表 | 两处的 skill 名必须 100% 一致 |
| 新增支持一种 agent（例：cursor / cline） | `README.md` §「我支持哪些 Agent？」表 + `AGENT-BOOTSTRAP.md` §1 派生路径 case + §2 安装方式 + §5 验证 case | 派生路径优先复用 `~/.agents/AGENTS.md` 通用位 |
| 调整下游用户的全局工作流（流程升级 / 降级、新触发条件等） | `global/AGENTS.md` 对应章节 | 只动一处，下游 `curl` 重跑即可 |
| 修改安装步骤 / 升级方式 | `AGENT-BOOTSTRAP.md`（agent 视角）+ `README.md` §快速开始（人类视角） | 两份步骤的小节序号对齐，便于互相引用 |
| 调整本仓库迭代流程 | 本 `AGENTS.md` | 不影响下游 |
| 措辞 / 排版微调 | 谁的就改谁的 | — |

判断口诀：**新规则给下游 → `global/AGENTS.md`；新动作 → `AGENT-BOOTSTRAP.md`；新解释 → `README.md`；本仓库内的开发流程 → 本文件。**

---

## 3. 默认开发流程（轻量为主）

本仓库的多数改动都属于轻量任务，默认按以下流程执行。

```
理解请求 → 影响面分析 → 直接修改 → 自检 → 输出报告
```

升级到中流程的信号：

- 同时改 ≥ 3 个文件 / 章节，且彼此存在引用关系
- 引入新的目标 agent / 新的安装机制 / 新的分发协议
- 涉及现有 skill 的兼容性（例如改 skill 触发词命名规则）

升级时先明确目标、边界、风险与验证方式，再进入实现，并在仓库内以 issue / PR 描述沉淀短计划。

---

## 4. 改动前的强制核对清单

每次准备修改前，agent 必须默念并核对：

- [ ] 这个改动是「全局规则」「安装动作」「人类导览」「项目级流程」的哪一类？只动对应文件。
- [ ] 是否动了 `global/AGENTS.md` 里被 `curl` 分发的部分？如果是，下游用户重跑安装才能拿到新版，必要时在 commit message 提示。
- [ ] 是否动了 `AGENT-BOOTSTRAP.md` 里 `for repo in ...` 等硬编码列表？必须与 `README.md` 的个人扩展 skill 清单保持一致。
- [ ] `README.md` 与 `AGENT-BOOTSTRAP.md` 的小节编号是否还能对齐（README §1↔ Bootstrap §4，README §3↔ Bootstrap §3 等）？
- [ ] 命令是否仍然 **不需要 clone 本仓库** 即可完成？（这是本项目的硬约束）

---

## 5. 自检（提交前必跑）

```bash
# 5.1 两处 skill 列表一致
readme_skills=$(grep -oE '`(research-note-wrap|session-wrap|commit-daily-summary|project-daily-summary|worktree-closeout)`' README.md | sort -u)
boot_skills=$(grep -oE '(research-note-wrap|session-wrap|commit-daily-summary|project-daily-summary|worktree-closeout)' AGENT-BOOTSTRAP.md | sort -u)
diff <(echo "$readme_skills" | tr -d '`') <(echo "$boot_skills") && echo "[ok] README.md ↔ AGENT-BOOTSTRAP.md skill 列表一致"

# 5.2 raw URL 域名拼写（必须指向 main/global/AGENTS.md）
grep -nE 'raw\.githubusercontent\.com/kiritoxkiriko/my-agent-workflow/main/global/AGENTS\.md' README.md AGENT-BOOTSTRAP.md \
  || echo "[warn] 未匹配到预期 raw URL"

# 5.3 文件大小 sanity check（避免误清空）
for f in AGENTS.md global/AGENTS.md AGENT-BOOTSTRAP.md README.md LICENSE; do
  size=$(wc -c < "$f")
  [ "$size" -gt 200 ] && echo "[ok]  $f ($size B)" || echo "[warn] $f 仅 $size B，疑似被清空"
done
```

输出全部 `[ok]` 才能进入 §6 的 commit / push。

---

## 6. Commit / Push 规范

本仓库采用以下 commit 规范：

- 格式：`<type>(scope): <summary>`
- `summary` 中文、动词开头、≤ 50 字、不加句号
- 常用 `scope`：`global` / `bootstrap` / `readme` / `agents` / `license`

示例：

```
feat(global): 增加 cursor 平台规则分发说明
docs(readme): 重写快速开始为 curl 直装方式
chore(bootstrap): 更新 skill 仓库列表
refactor(agents): 调整本仓库迭代自检命令
```

⚠️ 仅在用户明确要求时才 `commit` / `push`，并优先 `git add <file>` 显式列出文件，禁用 `git add -A` / `git add .`，避免把 `*.bak.*` 误提交。

---

## 7. 发布与「下游同步」时机

本仓库没有传统 release，但需要意识到：

- `global/AGENTS.md` / `AGENT-BOOTSTRAP.md` 一旦 push 到 `main`，下游用户**任意时间重跑 README §1 的 `curl`** 都会拿到新版本。
- 因此：避免在 `main` 上保留半成品状态；大改建议走特性分支或临时 commit 后立刻完成。
- 若需要钉版本，在 commit message 里写明 SHA，提示用户使用 `https://raw.githubusercontent.com/kiritoxkiriko/my-agent-workflow/<sha>/global/AGENTS.md`。

---

## 8. 安全 / 边界（项目级补充）

本仓库遵守以下安全边界：

- 禁止把私人路径（如 `/Users/bytedance/...`）写入任何会被 `curl` 分发的文件（`global/AGENTS.md` / `AGENT-BOOTSTRAP.md` / `README.md`）；只允许出现 `$HOME` / `~/`。
- 禁止把内部链接、内网 ID、token、API Key 写入任何文件。
- 本 `AGENTS.md` 允许出现仓库内绝对路径作为示意，但同样不放凭证。

---

## 9. 后续路线（可选）

按需扩展，不强制。新增项请同时更新本节与 [README.md](./README.md)：

- [ ] 增加 GitHub Actions：`global/AGENTS.md` 改动时自动跑 §5 自检。
- [ ] 提供一个 `install.sh` 一键脚本，整合 README 五个步骤；放仓库根，让用户 `curl ... | bash`（仍不要求 clone）。
- [ ] 支持版本号钉死：在 README 提供「钉到 commit SHA」的高级用法段。
- [ ] 适配更多 agent：cursor / cline / aider / continue（每加一个，按 §2 表格同步三处）。

---

## 10. Agent 在本仓库工作时的开场白模板

agent 第一次进入本仓库时，建议输出：

```
✅ 已识别项目：kiritoxkiriko/my-agent-workflow
🧠 已加载：AGENTS.md（项目级流程）+ global/AGENTS.md（要分发的全局规则，仅参考）
📌 默认按轻量任务执行；涉及 ≥3 文件 / 新 agent 平台 / 新分发协议时升级为中流程
```

然后再处理用户实际请求。
