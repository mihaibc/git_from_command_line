# Optional Git Tools and Terminal Setup

[Back to the command reference](../README.md)

Git works without these additions. Pick the tools that help your workflow; the terminal setup below assumes **macOS, Homebrew already installed, and Zsh**. Homebrew also requires Apple's command line tools; follow its [installation prerequisites](https://docs.brew.sh/Installation). On other platforms, use each tool's upstream installation guide.

## Tools worth trying

| Tool | Use | macOS installation |
| --- | --- | --- |
| [lazygit](https://github.com/jesseduffield/lazygit#installation) | Browse changes, stage hunks, and manage branches in a terminal UI | `brew install lazygit` |
| [delta](https://dandavison.github.io/delta/installation.html) | Read highlighted diffs with optional side-by-side display | `brew install git-delta` |
| [GitHub CLI](https://github.com/cli/cli/blob/trunk/docs/install_macos.md) | Work with GitHub pull requests, issues, and workflow runs | `brew install gh` |
| [fzf](https://github.com/junegunn/fzf#installation) | Search shell history, files, and directories interactively | `brew install fzf` |

For specialized tasks, explore [difftastic](https://github.com/Wilfred/difftastic) for syntax-aware diffs, [git-absorb](https://github.com/tummychow/git-absorb) for assigning staged changes to earlier commits, and [git-filter-repo](https://github.com/newren/git-filter-repo) for repository-wide history rewriting. Read their upstream guides before changing history.

### Configure delta

After installing delta, merge these settings into `~/.gitconfig`, keeping any unrelated settings. The [delta setup guide](https://dandavison.github.io/delta/get-started.html) explains its pager and interactive diff integration.

```ini
[core]
    pager = delta
[interactive]
    diffFilter = delta --color-only
[delta]
    navigate = true
    side-by-side = true
    line-numbers = true
```

## Personal Git aliases

Merge the aliases you want into `~/.gitconfig`. These are personal settings; cloning or pulling a repository does not propagate them. See [Git's configuration documentation](https://git-scm.com/docs/git-config) for configuration scopes and alias behavior.

```ini
[alias]
    s = status -sb
    l = log --oneline --graph --decorate --all
    sw = switch
    br = branch -vv
    ignored = ls-files --others --ignored --exclude-standard
    contributors = shortlog --summary --numbered --email HEAD
```

For example, run `git l` for the history graph or `git br` to inspect local branches and their upstreams. Keep history-changing commands explicit so their effect is visible when you run them.

## Shared hook files, local activation

[Git hooks](https://git-scm.com/docs/githooks) are executable scripts with exact names such as `pre-commit` and `pre-push`, without an extension. Git normally looks in `.git/hooks/`; a tracked `.githooks/` directory lets a team version the scripts.

The examples below require **Node.js and npm on `PATH`**, installed project dependencies, and working `lint` and `test` scripts in the project's `package.json`. Adapt the commands for other project types. These hooks are examples for your own project, not requirements for this documentation repository.

From the repository root, create the directory:

```bash
mkdir -p .githooks
```

Save this as `.githooks/pre-commit`:

```sh
#!/bin/sh
exec npm run lint --silent
```

Save this as `.githooks/pre-push`:

```sh
#!/bin/sh
exec npm test
```

Make the files executable and activate the directory in this clone:

```bash
chmod +x .githooks/pre-commit .githooks/pre-push
git config --local core.hooksPath .githooks
git add .githooks/pre-commit .githooks/pre-push
```

Commit the hook files to share them. **Each clone must activate `core.hooksPath` locally**: repository configuration is not propagated by a commit, clone, or pull. Check for an existing hook manager before replacing its hooks path. These examples check the working tree, including unstaged edits; they do not isolate staged content. Local hooks can be bypassed, so enforce required checks in CI too.

For commit messages, use the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) and a dedicated validator such as [commitlint](https://commitlint.js.org/). For npm-managed hook setup, see [Husky](https://typicode.github.io/husky/).

## Zsh setup

### Oh My Zsh and plugins

[Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh#basic-installation) requires Zsh, Git, and curl or wget. Follow its upstream installer instructions, then edit the existing `plugins=(...)` line in `~/.zshrc`; do not add competing plugin lists. Start with the bundled Git plugin:

```zsh
plugins=(git)
```

Optional community plugins must be installed separately. With Oh My Zsh installed, run:

```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions \
  "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
```

Then update the plugin list before the existing `source "$ZSH/oh-my-zsh.sh"` line, with syntax highlighting last:

```zsh
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

Open a new terminal to load the changes. Upstream instructions: [autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md) and [syntax highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md).

Oh My Zsh shortcuts are shell aliases, so you type them directly rather than after `git`. Inspect an alias with `alias gst`, or consult the [Git plugin source](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git/git.plugin.zsh) before relying on it.

### Fuzzy shell search

After installing a current fzf release, add its [Zsh integration](https://github.com/junegunn/fzf#setting-up-shell-integration) to `~/.zshrc`:

```zsh
source <(fzf --zsh)
```

Use this integration once; do not also enable the Oh My Zsh `fzf` plugin. Open a new terminal, then use `Ctrl+R` to search history, `Ctrl+T` to select files, or `Alt+C` to change directories. Older fzf releases may require the shell scripts described upstream.

### Powerlevel10k support status

[Powerlevel10k](https://github.com/romkatv/powerlevel10k) is an optional Git-aware Zsh prompt, but its maintainer reports **very limited support**, no planned features, and that most bugs will remain unfixed. Consider that before adopting it. Its upstream guide covers installation and fonts; existing users can run `p10k configure` to reopen the setup wizard.

## Editor integrations

| Integration | Use |
| --- | --- |
| [GitLens for VS Code](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) | Inline blame and history exploration |
| [lazygit.nvim](https://github.com/kdheepak/lazygit.nvim) | Open lazygit inside Neovim; requires lazygit separately |

Follow the linked project instructions for editor and plugin-manager requirements.
