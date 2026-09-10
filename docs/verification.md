# Verification notes

[Back to the guide](../README.md)

## Recipe verification

Verified on **2026-09-10**, using **Git 2.55.0 on macOS 26.6.2 (arm64)**. Fifteen scenario groups passed in disposable local repositories, with isolated Git configuration, a fixture author identity, and signing and hooks disabled.

| Scenario | Verified behavior |
| --- | --- |
| Quick start | Clone from a local bare remote, create a branch, edit, inspect, stage, commit, push, and establish upstream tracking |
| Cleanup | Preview before removal; ordinary cleanup preserves ignored files; `-x` includes them |
| Staging and stash | Staged, unstaged, and untracked changes; apply/drop; `--keep-index` keeps staged work while recording index state |
| Reset, revert, and recovery | Soft reset preserves edits; hard reset discards tracked edits; revert preserves history; reflog identifies a commit recovered onto a new branch |
| Merge conflicts | Resolve and continue; independently abort and restore the starting state |
| Rebase conflicts | Resolve and continue; independently abort and restore the starting state |
| Cherry-pick conflicts | Resolve and continue; independently abort and restore the starting state |
| Worktrees | Existing branch, new branch, refusal to remove dirty work, and branch retention after removal |
| Cherry-pick range | Start excluded and end included in a linear history |
| Submodules | Recursive clone, recorded revision update, remote merge update, and committed parent pointer |

The six conflict continuation/abort paths were exercised separately. An additional review reproduced a staged conflict-marker case and confirmed that `git diff --staged --check` detects it.

This does **not** establish compatibility with other operating systems or older Git versions. GitHub authentication, server-side policies, interactive editor sessions, partial-clone server behavior, and optional tool installations were not exercised. Tool instructions were checked against the upstream sources linked in the optional guide. Local submodule transport was enabled only for disposable fixtures.

## Documentation checks

Markdown lint uses markdownlint-cli2 **0.18.1**. Link checks use lychee **0.24.2** with `.lychee.toml`, including local files and heading fragments. The workflow pins its actions to immutable commits and runs on pull requests and pushes to `main`.

The local verification run passed Markdown lint for all three documents and checked 69 links (61 unique) with no errors. Markdown rendering produced the expected tables and fenced code blocks. Workflow YAML and action pins were checked locally; the workflow itself has not yet run on GitHub.

When updating recipes, repeat the relevant disposable-repository scenarios, run the documentation checks described in the README, and update this date and environment only after verification. An external link failure may be temporary; inspect it before changing the link or adding a narrowly justified exclusion.
