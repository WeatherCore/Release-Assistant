# Description

## 中文版

release-assistant 是收口 git 项目「tag 前最后一公里」的 DSH 技能，串起发版判断、semver 定版、版本对齐、notes、tag、网页发布与验证。产出按「三框交付契约」给出：Tag 框指引、Release title、正文 md 源码逐块可复制。易漏点钉成步骤：版本号升级提交与 tag 强制对齐，tag 永不悬空；本地 tag 先行再显式推送，避开裸 git push 不带 tag；不装 gh CLI；notes 交付后两道确认闸门，git 写操作复述命令后由助手执行；发布后 fetch --tags 加 tag→commit 表验证；短 hash 逐一核对，破坏性变更与安全加固置顶。纯提示词，不改业务代码，适合个人与小团队发版收口。

## English

release-assistant is a DSH skill that walks a local git repo through the last mile before a tag: decide whether a release is warranted, pick the semver bump, align the version file, write GitHub release notes, tag, publish through the web form, then verify. Notes arrive as a three-block paste contract — Tag guidance, Release title, and the Description body as markdown source — so each web-form field gets exactly what belongs in it. Its value sits in the details that quietly go wrong: the version-bump commit and the tag are forced to match, so a tag never floats above a stale manifest; the tag is created locally and pushed by explicit ref, sidestepping the bare git push that carries no tags; no gh CLI gets installed; two confirmation gates stand between the delivered notes and any git write, which the assistant executes itself after quoting the exact commands; and after publishing, git fetch --tags plus a tag-to-commit table proves what happened instead of assuming it. Breaking changes and security hardening get their own sections at the top, and every short hash is checked against git log. Prompt-only by design, no code, business files never touched. Built for solo developers and small teams closing out a version.
