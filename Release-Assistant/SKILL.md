---
name: release-assistant
description: "Use when user asks 发版/发行/发布 Release/release notes/版本号升级/打 tag/要不要发个版本，or a git project is about to ship a new version. Reads commits since last tag, recommends semver bump, offers version-file bump + push, generates release notes and delivers them as three paste-ready blocks (Tag / Release title / Description) for the GitHub web form, advises tag strategy (local tag first), syncs and verifies tags after publish. Requires explicit confirmation before every git write, which the assistant executes itself. Never installs tools (no gh CLI), never publishes on the user's behalf. Not for commit messages (commit-message-assistant), npm publish, or CI setup."
---

# Release Assistant

## Goal
带本地 git 项目走完「判断是否发版 → 定版本号 → 版本号对齐 → 写 release notes → 三框交付 → 打 tag → GitHub 网页发布 → 本地同步验证」的完整闭环。产出可直接复制粘贴的表单内容；git 写操作仅在用户明确同意后**由助手执行**，从不丢给用户手敲。

## Workflow
1. **摸底**：确认目标目录是 git 仓库（非仓库→告知并停）。收集：最近一个 tag（`git tag -l --sort=-creatordate`）、自该 tag 以来的提交（`git log --oneline <tag>..HEAD`，无 tag 则看全部历史）、版本清单当前版本（package.json / pyproject.toml / Cargo.toml 等，**多个清单并存则全部列出**）、remote 地址、tag 前缀惯例（扫 `git tag -l`：已有非 `v` 前缀惯例则沿用并告知，无历史则默认 `v` 前缀）。
2. **判断是否值得发版**：自上个 tag 无提交 → 告知无需发版并停；仅有琐碎 chore → 说明理由，交用户决定；有 feat / fix / 安全改动 → 建议发版并给出证据（提交清单）。
3. **定版本号**：破坏性变更 → 升 major；有 feat → 升 minor；仅 fix/chore → 升 patch；0.x 阶段谨慎升 major。对比清单当前版本给出建议，向用户确认。
4. **版本号对齐（bump 前检查）**：先确认 ① `git status --porcelain` 为空（工作区干净）② 当前分支是发布分支（通常主分支，否则告知用户）③ 多清单并存时全部 bump。然后提议修改版本文件并（经确认后）**由助手执行** commit + push，让 tag 打在「版本号升级」这条提交上——tag 与版本号永不悬空。
5. **生成 release notes 并三框交付**：读 `references/release-notes-format.md`，按提交分组（破坏性变更/安全/新功能/修复/其他，每条带短 hash），写标题与 Full Changelog compare 链接（`<prev-tag>...<new-tag>`）。安全加固必须单独成节突出；破坏性变更节永远第一节。交付按该文件的「三框交付契约」：**① Tag 框指引 ② Release title（独立代码块）③ Describe 正文（4 反引号围栏包 md 源码）**，附 Generate release notes 按钮警告。
6. **闸门一·notes 确认**：三框交付后停下，等用户确认 notes 内容无误。此后才能进入任何 tag 写操作。
7. **闸门二·tag 写操作**：单独发一条消息，原样复述确切命令并声明**由助手执行**（例：`git tag vX.Y.Z` + `git push origin vX.Y.Z`；裸 `git push` 不推送 tag，必须显式推送 tag 引用），拿到用户同意才执行。tag 一律 **lightweight**（禁 `-a`/`-m`/`-s`，否则验证对应表会指向 tag 对象而非提交）。
8. **发布（网页表单）**：用户按三框契约粘贴发布。**本地 tag 先行**路径发布后无需 fetch；**网页建 tag** 路径（表单输入新 tag 名，目标=主分支）发布后必须补 `git fetch --tags`。
9. **发布后同步与验证**：`git fetch --tags`（网络超时先告知用户查网络/代理，重试至多一次，不盲试）；验证用 `git tag -l --format='%(refname:short) -> %(*objectname:short) %(subject)'`（`*` 解引用：annotated tag 也返回提交哈希而非 tag 对象哈希）确认新 tag 指向预期提交。
10. **闭环汇报**：给出 tag → commit 对应表；提醒用户下次优先走「本地 tag 先行」以省掉 fetch 步骤。

## Decision Tree
- 首次发版（仓库无 tag）→ 建议 v1.0.0（或 0.x 惯例），notes 按「首个正式版」写全量特性。
- 用户问「要不要发版」→ 用步骤 2 的提交证据回答，给建议不强推。
- 用户提议安装 gh/winget 等工具 → 拒绝改走网页表单；任何安装必须单次单独征求同意。
- 附件区（Attach files）→ 一般留空；Web 应用不需要附加二进制。
- Release label → 正式版选「无」；实验/预览版才选「预发行版」。
- 表单里的「包/Packages」区 → 应用类项目跳过（那是发 npm/容器包用的）。
- 用户在网页发布（tag 由 GitHub 创建）→ 主动预告：发布后需要 `git fetch --tags` 同步。
- 正文需要内嵌代码块（如安装命令）→ 最多 3 反引号，确保三框交付的 4 反引号外层不被击穿。

## Constraints
- **不安装任何软件**（gh、CLI、依赖），除非用户对单次安装明确批准；未经批准装过东西要立即卸载并验证残留。
- **两道闸门缺一不可**：闸门一（notes 内容确认）不过不谈 tag；闸门二（tag 命令复述 + 同意）不过不执行写操作。git 写操作（commit / push / tag）由助手执行，执行前必须向用户复述确切命令并获得同意；push/fetch 失败不自动重试超过一次。
- 不改动代码，只动版本号文件与 git 元数据。
- release notes 语言跟随项目 README 的主语言；无 README 时跟随提交信息主语言，拿不准问用户一次。
- 用户跳过某步骤时尊重选择，不重复追问。

## Validation
- 新 tag 指向的提交 = 版本号升级提交（或用户指定的提交）。
- `git log <new-tag> -1 --oneline` 主题正确。
- release notes 中引用的每个短 hash 都能在 `git log` 中找到。
- Full Changelog 链接的 prev/next tag 拼写无误、compare 路径正确。
- 三框交付完整：Tag 指引、title 块、正文块三块齐备；正文以 md 源码（非渲染文本）形式出现在 4 反引号围栏内；标题行未混入正文。
- tag 为 lightweight：`git cat-file -t <tag>` 输出 `commit`（若输出 `tag` 则为 annotated，需查明原因）。

## Resources
- `references/release-notes-format.md`：生成 notes 前必读——分组结构、措辞规则、三框交付契约、标题惯例、compare 链接模板。
