# Description

## 中文版

release-assistant 是收口 git 项目「tag 前最后一公里」的 DSH 技能，把发版判断、semver 定版、版本对齐、notes、打 tag、网页发布到同步验证串成一条线。含金量是把易漏点钉成步骤：版本号升级提交与 tag 强制对齐，tag 永不悬空；本地 tag 先行再显式推送 tag 引用，避开裸 git push 不带 tag；不装 gh CLI，走网页表单；发布后用 git fetch --tags 与 tag→commit 对应表验证而非假设；notes 每个短 hash 回 git log 核对，安全加固单独置顶。纯提示词，仅 SKILL.md 与 release-notes-format.md，写操作先报命令再执行、不改代码。适合个人与小团队发版收口。

## English

release-assistant is a DSH skill that walks a local git repo through the last mile before a tag: decide whether a release is warranted, pick the semver bump, align the version file, write paste-ready GitHub release notes, tag, publish through the web form, then verify. Its value sits in the details that quietly go wrong: the version-bump commit and the tag are forced to match, so a tag never floats above a stale manifest; the tag is created locally and pushed by explicit ref, sidestepping the bare git push that carries no tags; no gh CLI gets installed, so the release form stays in your hands; and after publishing, git fetch --tags plus a tag-to-commit table proves what happened instead of assuming it. Release-worthiness is argued from commits, security hardening gets its own section at the top, and every short hash in the notes is checked against git log. Prompt-only by design: two markdown files, SKILL.md and references/release-notes-format.md, no code. Every git write is quoted back for approval first, business code is never touched, and a failure is not blindly retried. Built for solo developers and small teams closing out a version.
