---
name: release-assistant
description: "Use when user asks 发版/发行/发布 Release/release notes/版本号升级/打 tag/要不要发个版本，or a git project is about to ship a new version. Reads commits since last tag, recommends semver bump, offers version-file bump + push, generates paste-ready GitHub web-form release notes, advises tag strategy (local tag first), syncs and verifies tags after publish. Never installs tools (no gh CLI), never publishes on the user's behalf — the web form is operated by the user. Not for commit messages (commit-message-assistant), npm publish, or CI setup."
---

# Release Assistant

## Goal
带本地 git 项目走完「判断是否发版 → 定版本号 → 版本号对齐 → 写 release notes → 打 tag → GitHub 网页发布 → 本地同步验证」的完整闭环。产出可直接复制粘贴的表单内容；git 写操作仅在用户明确同意后执行。

## Workflow
1. **摸底**：确认目标目录是 git 仓库（非仓库→告知并停）。收集四样东西：最近一个 tag（`git tag -l --sort=-creatordate`）、自该 tag 以来的提交（`git log --oneline <tag>..HEAD`，无 tag 则看全部历史）、版本清单当前版本（package.json / pyproject.toml / Cargo.toml 等）、remote 地址。
2. **判断是否值得发版**：自上个 tag 无提交 → 告知无需发版并停；仅有琐碎 chore → 说明理由，交用户决定；有 feat / fix / 安全改动 → 建议发版并给出证据（提交清单）。
3. **定版本号**：破坏性变更 → 升 major；有 feat → 升 minor；仅 fix/chore → 升 patch；0.x 阶段谨慎升 major。对比清单当前版本给出建议，向用户确认。
4. **版本号对齐**：若清单版本 ≠ 目标版本，提议修改版本文件并（经确认后）commit + push，让 tag 打在「版本号升级」这条提交上——tag 与版本号永不悬空。
5. **生成 release notes**：读 `references/release-notes-format.md`，按提交分组（新功能/安全/修复/其他，每条带短 hash），写标题与 Full Changelog compare 链接（`<prev-tag>...<new-tag>`）。安全加固必须单独成节突出。
6. **发布（网页表单，tag 策略二选一）**：
   - **推荐·本地 tag 先行**：经用户确认后 `git tag vX.Y.Z` + `git push origin vX.Y.Z`（裸 `git push` 不推送 tag，必须显式推送 tag 引用），网页表单里**选已有 tag**——发布后无需 fetch。
   - **网页建 tag**：表单 Tag 下拉输入新 tag 名（选 create new tag，目标=主分支），发布后必须补 `git fetch --tags`。
7. **发布后同步与验证**：`git fetch --tags`（网络超时先告知用户查网络/代理，重试至多一次，不盲试）；`git tag -l --format='%(refname:short) -> %(objectname:short) %(subject)'` 确认新 tag 指向预期提交。
8. **闭环汇报**：给出 tag → commit 对应表；提醒用户下次优先走「本地 tag 先行」以省掉 fetch 步骤。

## Decision Tree
- 首次发版（仓库无 tag）→ 建议 v1.0.0（或 0.x 惯例），notes 按「首个正式版」写全量特性。
- 用户问「要不要发版」→ 用步骤 2 的提交证据回答，给建议不强推。
- 用户提议安装 gh/winget 等工具 → 拒绝改走网页表单；任何安装必须单次单独征求同意。
- 附件区（Attach files）→ 一般留空；Web 应用不需要附加二进制。
- Release label → 正式版选「无」；实验/预览版才选「预发行版」。
- 表单里的「包/Packages」区 → 应用类项目跳过（那是发 npm/容器包用的）。
- 用户在网页发布（tag 由 GitHub 创建）→ 主动预告：发布后需要 `git fetch --tags` 同步。

## Constraints
- **不安装任何软件**（gh、CLI、依赖），除非用户对单次安装明确批准；未经批准装过东西要立即卸载并验证残留。
- git 写操作（commit / push / tag）执行前必须向用户复述确切命令并获得同意；push/fetch 失败不自动重试超过一次。
- 不改动代码，只动版本号文件与 git 元数据。
- release notes 语言跟随项目 README 的主语言。
- 用户跳过某步骤时尊重选择，不重复追问。

## Validation
- 新 tag 指向的提交 = 版本号升级提交（或用户指定的提交）。
- `git log <new-tag> -1 --oneline` 主题正确。
- release notes 中引用的每个短 hash 都能在 `git log` 中找到。
- Full Changelog 链接的 prev/next tag 拼写无误、compare 路径正确。

## Resources
- `references/release-notes-format.md`：生成 notes 前必读——分组结构、措辞规则、标题惯例、compare 链接模板。
