# Git from the Command Line

A comprehensive reference for Git — from everyday commands to advanced workflows, power tools, and terminal setup.

---

## Table of Contents

- [Everyday Commands](#everyday-commands)
- [Advanced Commands](#advanced-commands)
- [Searching & History](#searching--history)
- [Branching & Merging](#branching--merging)
- [Rewriting History](#rewriting-history)
- [Worktrees](#worktrees)
- [Submodules](#submodules)
- [Git Aliases](#git-aliases)
- [Git Hooks](#git-hooks)
- [Tools to Enhance Git](#tools-to-enhance-git)
- [Terminal Setup with Oh My Zsh](#terminal-setup-with-oh-my-zsh)

---

## Everyday Commands

**Remove untracked files that are not added to the staging area**

```bash
git clean -fdx
```

**Pull commits and rebase your changes on top**

```bash
git pull --rebase --autostash
```

**List all the commands that were executed**

```bash
git reflog
```

**Reset unpushed commits**

```bash
git reset HEAD~1 --soft   # keep changes staged
git reset HEAD~1 --hard   # discard changes entirely
```
*A commit hash can be used instead of `HEAD~1` to reset to a specific commit.*

**Amend the last unpushed commit**

```bash
git commit --amend          # add staged files to the last commit
git commit --amend -m "New message"  # change the last commit message
git commit --amend --no-edit         # amend without changing the message
```

**Stage parts of a file interactively (hunk by hunk)**

```bash
git add -p
```

**Show what changed in the last commit**

```bash
git show
git show HEAD~2             # two commits ago
git show <commit>:<file>    # specific file at a specific commit
```

**Compare working tree, staging area, and commits**

```bash
git diff                   # unstaged changes
git diff --staged          # staged changes (about to be committed)
git diff main..feature     # difference between two branches
git diff HEAD~3            # changes since 3 commits ago
```

---

## Advanced Commands

### Stashing

**Stash with a descriptive name**

```bash
git stash push -m "WIP: login form validation"
git stash list
git stash pop              # apply and drop the latest stash
git stash apply stash@{2} # apply a specific stash without dropping it
git stash drop stash@{0}  # delete a specific stash
```

**Stash only unstaged changes (keep staged work intact)**

```bash
git stash push --keep-index
```

**Stash including untracked files**

```bash
git stash push -u
```

### Cherry-pick

**Apply a specific commit from another branch**

```bash
git cherry-pick <commit-hash>
git cherry-pick <hash1>..<hash2>   # apply a range of commits
git cherry-pick --no-commit <hash> # apply changes without committing
```

### Bisect — Find the Commit That Introduced a Bug

```bash
git bisect start
git bisect bad                     # current commit is broken
git bisect good v1.2.0             # last known good state
# Git checks out a midpoint — test it, then mark it:
git bisect good                    # or: git bisect bad
# Repeat until the culprit is found, then:
git bisect reset
```

**Automate bisect with a test script**

```bash
git bisect run npm test            # any command that exits 0 (pass) or 1 (fail)
```

### Fixup Commits

Create a commit that is intended to be squashed into an earlier one:

```bash
git commit --fixup <commit-hash>
git rebase -i --autosquash HEAD~5  # squash fixups automatically
```

### Sparse Checkout — Work with a Subset of a Large Repo

```bash
git clone --filter=blob:none --sparse <url>
cd repo
git sparse-checkout set src/frontend docs
```

### Signing Commits

```bash
git config --global user.signingkey <GPG-KEY-ID>
git config --global commit.gpgsign true
git commit -S -m "Signed commit"
git log --show-signature
```

### Git Notes — Attach Metadata Without Changing History

```bash
git notes add -m "Reviewed by Alice" <commit>
git log --show-notes
```

---

## Searching & History

**Search commit messages**

```bash
git log --all --grep="login bug"
```

**Search for when a string was added or removed (pickaxe)**

```bash
git log -S "functionName"          # commits that added/removed the exact string
git log -G "regex.*pattern"        # commits where diff matches a regex
```

**Show the full history of a file, including renames**

```bash
git log --follow -p -- path/to/file
```

**Find who last changed each line of a file**

```bash
git blame -w -C -C -C path/to/file
# -w ignores whitespace, -C -C -C tracks copies across files
```

**Ignore a bulk-formatting commit in blame**

```bash
echo "<formatting-commit-hash>" >> .git-blame-ignore-revs
git blame --ignore-revs-file .git-blame-ignore-revs path/to/file
# Add this to the repo so everyone benefits:
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

**Search file contents across the entire repo**

```bash
git grep "TODO" -- "*.ts"
git grep -n "apiKey"               # with line numbers
```

**Pretty log with graph**

```bash
git log --oneline --graph --decorate --all
```

**Log a specific file**

```bash
git log --stat -- path/to/file
```

---

## Branching & Merging

**Create and switch to a new branch**

```bash
git switch -c feature/my-feature   # modern syntax
git checkout -b feature/my-feature # classic syntax
```

**Rename the current branch**

```bash
git branch -m new-name
```

**Delete a remote branch**

```bash
git push origin --delete feature/old-branch
```

**Track a remote branch**

```bash
git branch --set-upstream-to=origin/main main
```

**Merge with a commit even when fast-forward is possible**

```bash
git merge --no-ff feature/my-feature
```

**Merge strategies**

```bash
git merge -X ours feature/conflicting   # prefer our changes on conflict
git merge -X theirs feature/conflicting # prefer their changes on conflict
```

**Prune deleted remote branches from your local list**

```bash
git fetch --prune
# or set it automatically:
git config --global fetch.prune true
```

---

## Rewriting History

**Interactive rebase — reorder, squash, edit, or drop commits**

```bash
git rebase -i HEAD~5       # last 5 commits
git rebase -i main         # all commits since branching off main
```

Inside the editor, each commit can be marked:
| Command | Effect |
|---------|--------|
| `pick`  | keep commit as-is |
| `reword`| keep commit, edit message |
| `edit`  | pause to amend the commit |
| `squash`| merge into previous commit |
| `fixup` | like squash, discard message |
| `drop`  | remove the commit entirely |

**Rebase onto a different base**

```bash
git rebase --onto main feature-base feature-branch
```

**Abort or continue a rebase after resolving conflicts**

```bash
git rebase --abort
git rebase --continue
```

**Filter-repo — rewrite history at scale** *(replaces `filter-branch`)*

```bash
pip install git-filter-repo
git filter-repo --path src/legacy --invert-paths   # remove a directory from all history
git filter-repo --replace-text replacements.txt    # scrub secrets from history
```

---

## Worktrees

Check out multiple branches simultaneously in separate directories — no stashing needed.

```bash
git worktree add ../hotfix hotfix/critical-bug
git worktree list
git worktree remove ../hotfix
```

---

## Submodules

**Add a submodule**

```bash
git submodule add https://github.com/org/lib libs/lib
```

**Clone a repo with all its submodules**

```bash
git clone --recurse-submodules <url>
# Or, after a plain clone:
git submodule update --init --recursive
```

**Update all submodules to their latest remote commit**

```bash
git submodule update --remote --merge
```

---

## Git Aliases

Add these to `~/.gitconfig` under `[alias]`:

```ini
[alias]
  s       = status -sb
  l       = log --oneline --graph --decorate --all
  ll      = log --pretty=format:"%C(yellow)%h%Creset %ad %C(cyan)%an%Creset %s" --date=short
  co      = checkout
  sw      = switch
  br      = branch -vv
  st      = stash
  undo    = reset HEAD~1 --soft
  aliases = config --get-regexp alias
  wip     = !git add -A && git commit -m "WIP"
  unwip   = reset HEAD~1 --soft
  ignored = ls-files --others --ignored --exclude-standard
  contributors = shortlog --summary --numbered --email
```

**Use them from the shell:**

```bash
git l                        # pretty graph log
git ll                       # detailed log with date and author
git undo                     # soft-reset the last commit
git wip                      # quickly save work-in-progress
git contributors             # who has committed the most
```

---

## Git Hooks

Hooks are scripts that run automatically at key points. They live in `.git/hooks/` or, for shared hooks, in a directory committed to the repo.

**Set a shared hooks directory (committed to the repo)**

```bash
git config core.hooksPath .githooks
```

**Common hook examples**

`pre-commit` — lint and format before every commit:

```bash
#!/bin/sh
npm run lint --silent || exit 1
```

`commit-msg` — enforce conventional commit format:

```bash
#!/bin/sh
grep -qE "^(feat|fix|docs|chore|refactor|test|style)(\(.+\))?: .{1,72}" "$1" \
  || { echo "Commit message must follow Conventional Commits"; exit 1; }
```

`pre-push` — run tests before pushing:

```bash
#!/bin/sh
npm test || exit 1
```

**Recommended tool: [Husky](https://github.com/typicode/husky)**  
Manages Git hooks via npm scripts, ideal for JavaScript/TypeScript projects.

---

## Tools to Enhance Git

### TUIs (Terminal UIs)

| Tool | Description | Install |
|------|-------------|---------|
| [lazygit](https://github.com/jesseduffield/lazygit) | Fast, keyboard-driven Git UI in the terminal | `brew install lazygit` |
| [tig](https://github.com/jonas/tig) | Ncurses-based text-mode Git browser | `brew install tig` |
| [gitui](https://github.com/extrawurst/gitui) | Blazing-fast TUI written in Rust | `brew install gitui` |

### Better Diffs

| Tool | Description | Install |
|------|-------------|---------|
| [delta](https://github.com/dandavison/delta) | Syntax-highlighted, side-by-side diffs | `brew install git-delta` |
| [difftastic](https://github.com/Wilfred/difftastic) | Structural diffs that understand syntax | `brew install difftastic` |

**Configure delta as the default pager:**

```ini
# ~/.gitconfig
[core]
  pager = delta
[delta]
  navigate = true
  side-by-side = true
  line-numbers = true
[interactive]
  diffFilter = delta --color-only
```

### GitHub / Forge CLIs

| Tool | Description | Install |
|------|-------------|---------|
| [gh](https://github.com/cli/cli) | Official GitHub CLI — PRs, issues, actions, and more | `brew install gh` |
| [lab](https://github.com/zaquestion/lab) | GitLab CLI wrapper | `brew install lab` |

**Useful `gh` commands:**

```bash
gh pr create --fill                      # open a PR from current branch
gh pr checkout 123                       # check out a PR locally
gh pr view --web                         # open the PR in the browser
gh run watch                             # watch CI run in real time
gh issue list --assignee @me             # your open issues
gh repo clone org/repo                   # clone any repo
```

### Productivity & Extras

| Tool | Description | Install |
|------|-------------|---------|
| [git-extras](https://github.com/tj/git-extras) | 60+ extra git commands (`git summary`, `git effort`, `git obliterate`, …) | `brew install git-extras` |
| [forgit](https://github.com/wfxr/forgit) | fzf-powered interactive git commands | See repo |
| [git-absorb](https://github.com/tummychow/git-absorb) | Automatically absorb staged changes into the right commit | `brew install git-absorb` |
| [git-filter-repo](https://github.com/newren/git-filter-repo) | Fast, safe history rewriting | `brew install git-filter-repo` |
| [commitizen](https://github.com/commitizen/cz-cli) | Interactive Conventional Commits prompt | `npm install -g commitizen` |
| [git-branchless](https://github.com/arxanas/git-branchless) | Stacked diffs and advanced history manipulation | `brew install git-branchless` |

### Editor Integrations

| Tool | Description |
|------|-------------|
| [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) | VS Code — inline blame, history, PR annotations |
| [lazygit in Neovim](https://github.com/kdheepak/lazygit.nvim) | Float lazygit inside Neovim |

---

## Terminal Setup with Oh My Zsh

[Oh My Zsh](https://ohmyz.sh) supercharges your terminal with themes, plugins, and hundreds of Git-aware shortcuts.

### Install

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Essential Plugins

Enable plugins in `~/.zshrc`:

```bash
plugins=(git gitfast git-extras z zsh-autosuggestions zsh-syntax-highlighting fzf)
```

**Install the community plugins first:**

```bash
# zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### The Built-in `git` Plugin

The Oh My Zsh `git` plugin provides ~150 aliases. Key ones:

| Alias | Command |
|-------|---------|
| `g` | `git` |
| `ga` | `git add` |
| `gaa` | `git add --all` |
| `gc` | `git commit --verbose` |
| `gc!` | `git commit --verbose --amend` |
| `gcm` | `git checkout main` |
| `gco` | `git checkout` |
| `gd` | `git diff` |
| `gds` | `git diff --staged` |
| `gf` | `git fetch` |
| `gl` | `git pull` |
| `gp` | `git push` |
| `grb` | `git rebase` |
| `grbi` | `git rebase -i` |
| `gst` | `git status` |
| `gsta` | `git stash push` |
| `gstp` | `git stash pop` |
| `glol` | pretty graph log |
| `glog` | verbose graph log |

Full list: `alias | grep "^g"` or see the [plugin source](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git/git.plugin.zsh).

### Recommended Theme: Powerlevel10k

A fast, richly-configurable prompt that shows Git branch, status, dirty state, and more at a glance.

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# In ~/.zshrc:
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Then restart your shell and run `p10k configure` for the interactive wizard.

### FZF — Fuzzy Finder for Everything

```bash
brew install fzf
$(brew --prefix)/opt/fzf/install
```

With the `fzf` plugin active, you get:
- `Ctrl+R` — fuzzy search command history
- `Ctrl+T` — fuzzy search files
- `Alt+C` — fuzzy cd into subdirectories

**Combine with forgit for fuzzy Git:**

```bash
glo   # fuzzy log
gad   # fuzzy git add
gcf   # fuzzy checkout file
gbd   # fuzzy branch delete
```

---

## Further Reading

- [Pro Git Book](https://git-scm.com/book/en/v2) — free, comprehensive, official
- [Dangit, Git!?!](https://dangitgit.com) — plain-English fixes for common mistakes
- [Conventional Commits](https://www.conventionalcommits.org) — a commit message standard
- [Oh My Zsh Wiki](https://github.com/ohmyzsh/ohmyzsh/wiki)
- [Powerlevel10k](https://github.com/romkatv/powerlevel10k)
