# Git from the Command Line

Practical Git recipes for everyday developers: inspect changes, collaborate on branches, and recover from mistakes. Start with the short workflow below, or jump directly to the task you need.

You need Git installed and a terminal. Examples use a POSIX shell (macOS/Linux, or Git Bash on Windows), a remote named `origin`, and a default branch named `main`. Substitute your actual names. Replace placeholders such as `<repository-url>`, `<commit>`, and `path/to/file` before running commands; angle brackets are not literal shell input. Commands in separate recipes are independent, not one script to run from top to bottom.

Optional tools, aliases, hooks, and Zsh customization live in [Tools and terminal setup](docs/tools-and-terminal.md).

## Contents

- [Quick start](#quick-start)
- [Inspect changes](#inspect-changes)
- [Stage and commit](#stage-and-commit)
- [Branches and remotes](#branches-and-remotes)
- [Undo and recover](#undo-and-recover)
- [Resolve conflicts](#resolve-conflicts)
- [Advanced recipes](#advanced-recipes)
- [Further reading](#further-reading)
- [Contributing and verification](#contributing-and-verification)

## Quick start

### Clone a repository and publish your first branch

Use an existing repository with at least one commit, write access, and working HTTPS or SSH authentication. If you cannot push to it, fork it on GitHub and clone your fork instead.

```bash
git clone <repository-url> my-project
cd my-project
git status
git switch -c feature/my-change
```

If Git does not already know your author identity, set it for this clone. Use the email you want recorded in commits (a GitHub-provided private email is also an option).

```bash
git config user.name "Your Name"
git config user.email "you@example.com"
```

Edit a file in your editor, then inspect and commit that specific file:

```bash
git diff -- path/to/file
git add path/to/file
git diff --staged
git commit -m "Describe the change"
git push -u origin feature/my-change
```

Your branch now exists remotely and tracks its counterpart. Open the repository on GitHub and create a pull request into `main`. Check the staged diff before committing to avoid including unrelated work or secrets.

Plain `git config` writes settings for this clone. Those settings are not shared by committing files; `--global` instead changes your defaults across repositories.

## Inspect changes

### See what is modified and what will be committed

```bash
git status --short --branch
git diff                     # unstaged changes to tracked files
git diff --staged            # changes staged for the next commit
git show                    # latest commit and its patch
```

Untracked files appear in status but not in a normal diff. Review their contents before staging them.

### Compare branches or inspect an older file

```bash
git diff main feature/my-change           # compare the branch tips
git diff main...feature/my-change         # changes from the merge base to feature
git show <commit>:path/to/file            # print the committed file
```

The three-dot form helps review what a feature branch introduced since its common ancestor with `main`. Neither command changes files. See [git diff](https://git-scm.com/docs/git-diff).

### Find a change in history

```bash
git log --oneline --graph --decorate --all
git log --all --grep="login bug"
git log -S "functionName" -- path/to/file
git log -G "regex.*pattern" -- path/to/file
git log --follow -p -- path/to/file
```

`-S` finds commits that change the number of occurrences of a string; `-G` matches added or removed lines against a regex. `--follow` follows one file through renames. See [git log](https://git-scm.com/docs/git-log).

### Search tracked files and explain a line

```bash
git grep -n "TODO" -- '*.ts'
git blame -w path/to/file
```

`git grep` searches tracked working-tree files by default. Blame shows the last commit affecting each line; `-w` ignores whitespace differences.

To ignore a known bulk-formatting commit, put its full hash on a line in `.git-blame-ignore-revs`, commit that file, and enable it in each clone:

```bash
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

The file is shared; the configuration is not. See [git blame](https://git-scm.com/docs/git-blame).

## Stage and commit

### Stage only part of your changes

With edits to an already tracked file:

```bash
git add -p -- path/to/file
git diff --staged
git commit -m "Describe one logical change"
```

Choose `y` to stage a hunk, `n` to leave it unstaged, or `s` to split it when possible. Other edits remain in your working tree. See [interactive staging](https://git-scm.com/docs/git-add).

### Unstage a file without losing changes

```bash
git restore --staged -- path/to/file
```

This resets that index entry to `HEAD`, keeping your working file intact. It requires an existing commit. See [git restore](https://git-scm.com/docs/git-restore).

### Amend your last local commit

Use this only for a commit you have not shared. First check that the index contains only changes intended for that commit.

```bash
git add path/to/file
git diff --staged
git commit --amend --no-edit
```

This replaces the last commit while retaining its message. To edit only the message, leave the index clean and run `git commit --amend`. Amending changes the commit ID; see [git commit](https://git-scm.com/docs/git-commit).

## Branches and remotes

### Create a branch from an up-to-date main

Commit or stash current work first. Then:

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/my-change
```

If `--ff-only` refuses because histories diverged, inspect `git log --oneline --graph --all` and choose a merge or rebase deliberately. Do not reset your branch just to silence the error.

### Check out a remote branch

After fetching, create a local tracking branch that does not already exist:

```bash
git fetch origin
git switch --track origin/feature/my-change
git branch -vv
```

If the local branch already exists, use `git switch feature/my-change`. See [git switch](https://git-scm.com/docs/git-switch).

### Bring main into your feature branch

With a clean working tree on the feature branch:

```bash
git fetch origin
git merge origin/main
```

This preserves existing commits and may create a merge commit. Follow [Resolve conflicts](#resolve-conflicts) if needed. For unpublished commits, rebasing is another option described under [Clean up local commits](#clean-up-local-commits).

### Rename or remove a finished branch

Rename your current local branch:

```bash
git branch -m new-name
```

To remove an already merged local branch, first switch away from it:

```bash
git switch main
git branch -d feature/my-change
```

If Git refuses, inspect what is unmerged instead of forcing deletion. Deleting a branch does not merge its work.

**Remote deletion:** after confirming the branch is no longer needed by collaborators, this removes it from the server:

```bash
git push origin --delete feature/my-change
```

Refresh your remote-tracking references afterward:

```bash
git fetch --prune
```

Pruning removes stale remote-tracking references, not your local branches. See [git fetch](https://git-scm.com/docs/git-fetch).

## Undo and recover

### Choose the right kind of undo

| What you want | Use | Effect |
| --- | --- | --- |
| Keep edits, undo staging | `git restore --staged -- path/to/file` | Changes the index only |
| Discard unstaged edits | `git restore -- path/to/file` | Overwrites the file from the index |
| Undo the last local commit, keep edits | `git reset --soft HEAD~1` | Moves the branch back; keeps index and files |
| Undo a published commit | `git revert <commit>` | Adds a new inverse commit |
| Find a previous local branch position | `git reflog` | Shows local reference updates |

Read the corresponding recipe before running an undo command.

### Discard edits to one tracked file

**Destructive:** this permanently discards unstaged changes in the selected file. Staged changes remain. Inspect `git diff -- path/to/file` and save any work you want first.

```bash
git restore -- path/to/file
```

### Undo your last local commit but keep its changes

Use this on an unpublished commit with a parent (not the repository's first commit):

```bash
git reset --soft HEAD~1
git diff --staged
```

Your branch moves back one commit. Its changes remain staged, along with anything already staged. Your working files are unchanged. See [git reset](https://git-scm.com/docs/git-reset).

### Discard the last local commit and tracked changes

**Destructive:** only use this when you intentionally want to discard the last unpublished commit plus staged and unstaged tracked changes. Untracked files obstructing tracked paths can also be deleted. Save anything needed first. The commit must have a parent.

```bash
git reset --hard HEAD~1
```

Reflog may recover the old commit, but it is not a backup of uncommitted edits.

### Revert a commit that other people may already use

With a clean working tree, choose an ordinary, non-merge commit:

```bash
git revert <commit>
```

Git adds a new commit reversing its changes, preserving shared history. If conflicts occur, edit the files, stage resolutions, then run `git revert --continue`; use `git revert --abort` to cancel. Reverting a merge requires choosing a mainline parent; consult [git revert](https://git-scm.com/docs/git-revert) before doing so.

### Recover a commit after a reset

```bash
git reflog
```

Find the commit you need, inspect it, and give it a branch name:

```bash
git show <commit>
git branch recovered-work <commit>
```

This preserves the recovered commit without overwriting current files. Switch to `recovered-work` when your working tree is clean.

Reflog records local reference updates, not every executed command. Entries expire, are not transferred when cloning, and cannot recover arbitrary untracked or uncommitted files. See [git reflog](https://git-scm.com/docs/git-reflog).

### Preview and remove untracked files

Preview first, from the repository root if you want to inspect the entire working tree:

```bash
git clean -nd
```

**Destructive:** after reviewing the exact list and saving anything needed, remove untracked files and directories:

```bash
git clean -fd
```

Ignored files are preserved by default. To include ignored files, preview separately:

```bash
git clean -ndx
```

**Destructive, including ignored files:** only after inspecting that preview, the following can delete local `.env` files, dependencies, build output, and other ignored data:

```bash
git clean -fdx
```

Git generally cannot recover files it never tracked. See [git clean](https://git-scm.com/docs/git-clean).

## Resolve conflicts

### Finish or cancel a merge, rebase, or cherry-pick

Start these operations with a clean working tree so aborting has a predictable starting point. When Git stops for conflicts:

1. Run `git status` to see the operation and affected files.
2. Edit each conflicted file: choose the intended result and remove conflict markers. For an intended deletion, use `git rm -- path/to/file` instead of staging it with `git add`.
3. Stage resolved files with `git add path/to/file`.
4. Review `git diff --staged`, check `git diff --staged --check`, and run the project's relevant tests.
5. Run **one** matching continuation command below. Rebase and cherry-pick may stop again at another commit; repeat as necessary.

| Operation | Continue after resolving | Cancel instead |
| --- | --- | --- |
| Merge | `git merge --continue` | `git merge --abort` |
| Rebase | `git rebase --continue` | `git rebase --abort` |
| Cherry-pick | `git cherry-pick --continue` | `git cherry-pick --abort` |

Cancellation abandons resolutions made during the operation. Do not blindly choose “ours” or “theirs”: their interpretation during a rebase is easy to confuse with normal merging. See [git merge](https://git-scm.com/docs/git-merge), [git rebase](https://git-scm.com/docs/git-rebase), and [git cherry-pick](https://git-scm.com/docs/git-cherry-pick).

## Advanced recipes

### Put unfinished work aside

```bash
git stash push -u -m "WIP: login form"
git stash list
```

`-u` includes untracked files, but not ignored files. Restore onto a clean working tree when possible:

```bash
git stash apply 'stash@{0}'
git status
```

`apply` retains the stash so you can inspect and test the result. It does not restore the original staging arrangement by default; `--index` attempts that too. If there are conflicts, resolve them manually; there is no `git stash --abort`. Keep the stash until your work is safe.

After verifying restoration, recheck `git stash list` and delete only the intended entry:

```bash
git stash drop 'stash@{0}'
```

Dropping removes that saved copy. Alternatively, `git stash pop` applies and removes a stash on success; it retains it when applying conflicts.

To leave staged changes in place while putting other edits aside:

```bash
git stash push --keep-index
```

The stash still records index state; “keep index” does not mean the stash contains only unstaged changes. See [git stash](https://git-scm.com/docs/git-stash).

### Apply selected commits from another branch

With a clean working tree on the receiving branch:

```bash
git cherry-pick <commit>
```

This creates a new commit applying the selected change. For a simple linear range, inspect the selection before applying it:

```bash
git log --oneline --reverse <start>..<end>
git cherry-pick <start>..<end>
```

The range excludes `<start>` and includes `<end>`; more generally, it selects commits reachable from the end but not the start. Merge commits require additional decisions. Use the [conflict workflow](#resolve-conflicts) if needed. See [git cherry-pick](https://git-scm.com/docs/git-cherry-pick).

### Find the commit that introduced a bug

Start with a clean working tree, a reproducible failure, and a known good commit or tag:

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Git checks out a candidate. Test it and run **one** verdict:

```bash
git bisect good
```

If it fails, use `git bisect bad` instead; if you cannot test that revision, use `git bisect skip`. Repeat until Git identifies the first bad commit or reports ambiguity.

Always return to your starting branch afterward:

```bash
git bisect reset
```

For automation, run `git bisect run <test-command>` after marking the endpoints. Exit `0` means good, `1`–`127` except `125` mean bad, `125` skips a revision, and other exit codes abort. Ensure the command exists across the tested history; a missing command can be misclassified as a bad commit. See [git bisect](https://git-scm.com/docs/git-bisect).

### Clean up local commits

**Rewrites history:** use on unpublished commits. Amending, rebasing, and autosquashing replace commit IDs. Coordinate any shared-history rewrite with collaborators; do not force-push simply to bypass a rejected push.

With a clean working tree on a simple feature branch based on `main`:

```bash
git rebase -i main
```

The editor lists commits to replay. Use `reword` to change a message, `squash` to combine with the preceding commit, `fixup` to combine and discard its message, or `drop` to remove a commit. Review the result and run tests. Use the [conflict workflow](#resolve-conflicts) to continue or abort.

For a correction intended for a specific local commit, stage the correction and run:

```bash
git commit --fixup <commit>
git rebase -i --autosquash main
```

The target must be among the commits selected for replay. Autosquash arranges the fixup in the editor; review and save the todo list to proceed. See [git rebase](https://git-scm.com/docs/git-rebase).

### Work on two branches in separate directories

For an existing branch that is not already checked out in another worktree:

```bash
git worktree add ../hotfix hotfix/critical-bug
```

Alternatively, create a new branch from `main` (use a fresh path and branch name):

```bash
git worktree add -b hotfix/new-fix ../new-fix main
```

Each directory has its own working files and index. List them with:

```bash
git worktree list
```

When finished, commit or save work and run `git worktree remove ../hotfix` from the original checkout. Removal deletes that working directory, retains the branch, and normally refuses dirty worktrees. Do not force removal to bypass unsaved work. See [git worktree](https://git-scm.com/docs/git-worktree).

### Check out only part of a large repository

For a server supporting partial clone:

```bash
git clone --filter=blob:none --sparse <repository-url> large-project
cd large-project
git sparse-checkout set src/frontend docs
```

Substitute real directory paths. Cone mode includes the selected directories and some ancestor/root files; other files can be fetched on demand. This reduces the checkout, not your access to repository history. See [git sparse-checkout](https://git-scm.com/docs/git-sparse-checkout).

### Clone and update submodules deliberately

To get the revisions recorded by a parent repository:

```bash
git clone --recurse-submodules <repository-url> my-project
```

For an existing clone, including after pulling changes to its recorded submodule revisions:

```bash
git submodule update --init --recursive
```

Save any edits inside submodules before updating. The command checks out the commits recorded by the parent, typically in detached HEAD state.

To deliberately advance one submodule to its configured remote branch, start with clean parent and submodule working trees:

```bash
git submodule update --remote --merge -- libs/lib
git diff --submodule=log
git add libs/lib
git commit -m "Update lib submodule"
```

Substitute the actual submodule path, inspect changes, and test before committing. The parent records a commit pointer, not a copy of the submodule's files. If your update creates a local merge commit inside the submodule, publish it to an accessible submodule remote before publishing the parent pointer. See [git submodule](https://git-scm.com/docs/git-submodule).

### Respond to an exposed secret

Revoke or rotate the secret first. Removing a file or rewriting Git history does not invalidate credentials or erase copies in forks, other clones, or caches.

After containing the exposure, follow GitHub's [sensitive-data removal procedure](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository) and the [git-filter-repo instructions](https://github.com/newren/git-filter-repo). Work from a fresh clone and coordinate the history rewrite and collaborator cleanup. This needs a repository-specific procedure, not a one-line cleanup command.

## Further reading

- [Pro Git](https://git-scm.com/book/en/v2) — concepts and deeper explanations.
- [Git command reference](https://git-scm.com/docs) — exact options and behavior.
- [GitHub pull request guide](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) — publish changes for review.
- [Tools and terminal setup](docs/tools-and-terminal.md) — optional aliases, hooks, and integrations.

## Contributing and verification

Corrections and focused recipes are welcome. Include the use case, prerequisites, expected result, and an official source for subtle behavior. Test commands that change files or history in disposable repositories; explain data loss and shared-history effects before the command. Prefer improving an existing recipe over expanding the tool list.

Documentation CI checks Markdown formatting and links on pull requests and pushes to `main`. Run the same lint locally with `npx --yes markdownlint-cli2@0.18.1 README.md 'docs/**/*.md'`; run links with `lychee --config .lychee.toml README.md 'docs/**/*.md'` after installing [lychee](https://github.com/lycheeverse/lychee). Investigate external-link failures before adding exclusions.

Verification details are recorded in [the validation notes](docs/verification.md). Examples are not a guarantee for every Git version, shell, server configuration, or repository history.

Licensed under [Apache-2.0](LICENSE).
