# 🧠 The Ultimate Git Cheat Sheet
> From zero to expert — everything you need to understand and master Git.

---

## 📖 What is Git?

Git is a **distributed version control system** (VCS). It tracks changes to files over time, lets multiple people collaborate, and allows you to revert to any previous state. Every developer has a full copy of the entire project history on their machine — no single point of failure.

**Key concepts before you start:**

| Term | What it means |
|------|--------------|
| **Repository (repo)** | A folder tracked by Git, containing your project and its full history |
| **Commit** | A snapshot of your files at a point in time |
| **Branch** | An independent line of development (like a parallel universe of your code) |
| **Remote** | A version of your repo hosted elsewhere (e.g. GitHub, GitLab) |
| **HEAD** | A pointer to the current commit you're working from |
| **Staging area (index)** | A holding area where you prepare changes before committing |
| **Working directory** | The actual files on your disk that you edit |

**The three states of Git:**
```
Working Directory  →  Staging Area (Index)  →  Repository (.git)
    (edit files)       (git add)                  (git commit)
```

---

## ⚙️ First-Time Setup

```bash
# Set your identity (required before your first commit)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set default branch name to 'main'
git config --global init.defaultBranch main

# Set your preferred editor (e.g. VS Code, vim, nano)
git config --global core.editor "code --wait"
git config --global core.editor "vim"

# Enable helpful color output
git config --global color.ui auto

# View all your config settings
git config --list

# View a specific setting
git config user.name
```

---

## 🚀 Starting a Repository

```bash
# Initialize a new Git repo in the current folder
git init

# Initialize with a specific branch name
git init -b main

# Clone an existing remote repo to your machine
git clone https://github.com/user/repo.git

# Clone into a specific folder name
git clone https://github.com/user/repo.git my-folder

# Clone a specific branch only
git clone -b develop https://github.com/user/repo.git

# Shallow clone (only latest snapshot, faster for large repos)
git clone --depth 1 https://github.com/user/repo.git
```

---

## 📊 Checking Status & History

```bash
# See what's changed (modified, staged, untracked)
git status

# Short/compact status view
git status -s

# View commit history
git log

# Compact one-line log
git log --oneline

# Visual branch graph
git log --oneline --graph --all --decorate

# Log with file change stats
git log --stat

# Log with full diff
git log -p

# Show last N commits
git log -5

# Search commits by message
git log --grep="fix bug"

# Show commits by a specific author
git log --author="Alice"

# Show commits in a date range
git log --after="2024-01-01" --before="2024-06-01"

# Show what changed in a specific commit
git show <commit-hash>

# Show the diff of a file in a commit
git show <commit-hash>:path/to/file.txt
```

---

## ✏️ Staging & Committing

```bash
# Stage a specific file
git add filename.txt

# Stage all changes in the current directory
git add .

# Stage all changes everywhere (including deletions)
git add -A

# Stage parts of a file interactively (hunk by hunk)
git add -p filename.txt

# Unstage a file (keep changes in working directory)
git restore --staged filename.txt

# Commit staged changes with a message
git commit -m "Your commit message"

# Stage all tracked files AND commit in one step
git commit -am "Your message"

# Open editor to write a detailed commit message
git commit

# Amend the most recent commit (message or content)
git commit --amend -m "Corrected message"

# Amend without changing the commit message
git commit --amend --no-edit
```

> 💡 **Good commit messages:** Use the imperative mood. Write "Add login feature" not "Added login feature". Keep the subject under 72 characters.

---

## 🔍 Viewing Differences

```bash
# Show unstaged changes (working dir vs staging area)
git diff

# Show staged changes (staging area vs last commit)
git diff --staged
git diff --cached   # same thing

# Compare two branches
git diff main..feature-branch

# Compare two commits
git diff abc123..def456

# Show only the names of changed files
git diff --name-only

# Show diff for a specific file
git diff filename.txt

# Word-level diff (easier to read for text)
git diff --word-diff
```

---

## 🌿 Branches

```bash
# List all local branches
git branch

# List all branches (local + remote)
git branch -a

# List remote branches only
git branch -r

# Create a new branch
git branch feature-login

# Switch to a branch
git switch feature-login
git checkout feature-login   # older syntax

# Create AND switch to a new branch
git switch -c feature-login
git checkout -b feature-login   # older syntax

# Rename a branch
git branch -m old-name new-name

# Rename the current branch
git branch -m new-name

# Delete a branch (safe — won't delete unmerged)
git branch -d feature-login

# Force delete a branch (even if unmerged)
git branch -D feature-login

# See which branch each commit is on
git log --oneline --graph --all
```

---

## 🔀 Merging

Merging integrates changes from one branch into another.

```bash
# Merge a branch into the current branch
git merge feature-login

# Merge with a commit even if fast-forward is possible
git merge --no-ff feature-login

# Squash all commits from branch into one (then commit manually)
git merge --squash feature-login
git commit -m "Add login feature"

# Abort a merge that has conflicts
git merge --abort
```

**Fast-forward vs. No-fast-forward:**
- **Fast-forward:** If no new commits on the target branch, Git just moves the pointer forward (no merge commit created).
- **No-fast-forward (--no-ff):** Always creates a merge commit, preserving branch history.

---

## 🔁 Rebasing

Rebasing replays commits from one branch onto another, creating a **linear history**.

```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase — rewrite, squash, reorder last N commits
git rebase -i HEAD~3

# Abort a rebase in progress
git rebase --abort

# Continue after resolving conflicts
git rebase --continue

# Skip a commit during rebase
git rebase --skip
```

**Interactive rebase commands (shown in editor):**

| Command | Action |
|---------|--------|
| `pick` | Keep the commit as-is |
| `reword` | Keep commit, but edit the message |
| `edit` | Pause to amend the commit |
| `squash` | Combine with the previous commit |
| `fixup` | Like squash, but discard this commit's message |
| `drop` | Delete the commit entirely |

> ⚠️ **Golden rule:** Never rebase commits that have been pushed to a shared/public branch. It rewrites history and causes problems for other collaborators.

---

## 🌐 Working with Remotes

```bash
# View remote connections
git remote -v

# Add a remote
git remote add origin https://github.com/user/repo.git

# Rename a remote
git remote rename origin upstream

# Remove a remote
git remote remove origin

# Change the URL of a remote
git remote set-url origin https://github.com/user/new-repo.git

# Fetch all changes from remote (doesn't merge)
git fetch origin

# Fetch a specific branch
git fetch origin main

# Pull = fetch + merge into current branch
git pull

# Pull with rebase instead of merge
git pull --rebase

# Push current branch to remote
git push origin main

# Push and set upstream tracking
git push -u origin feature-branch

# Push all branches
git push --all origin

# Push tags to remote
git push --tags

# Delete a remote branch
git push origin --delete feature-branch

# Force push (use with extreme caution!)
git push --force
git push --force-with-lease   # safer: fails if remote has new commits
```

---

## ↩️ Undoing Changes

This is one of the most important — and nuanced — areas of Git.

```bash
# Discard unstaged changes in a file (unrecoverable!)
git restore filename.txt
git checkout -- filename.txt   # older syntax

# Discard ALL unstaged changes
git restore .

# Unstage a file (keep changes in working dir)
git restore --staged filename.txt

# Undo the last commit, keep changes staged
git reset --soft HEAD~1

# Undo the last commit, keep changes unstaged
git reset --mixed HEAD~1   # default behavior of git reset HEAD~1

# Undo the last commit AND discard all changes (destructive!)
git reset --hard HEAD~1

# Undo a commit by creating a NEW commit that reverses it (safe for shared branches)
git revert <commit-hash>

# Revert multiple commits
git revert HEAD~3..HEAD

# Revert without auto-committing
git revert --no-commit <commit-hash>
```

**When to use what:**

| Scenario | Command |
|----------|---------|
| Discard local file changes | `git restore filename` |
| Unstage file | `git restore --staged filename` |
| Undo last commit (keep work) | `git reset --soft HEAD~1` |
| Undo commits on private branch | `git reset --hard` |
| Undo commits on shared branch | `git revert` |

---

## 🏷️ Tags

Tags mark specific points in history, typically for releases.

```bash
# List all tags
git tag

# Create a lightweight tag
git tag v1.0.0

# Create an annotated tag (recommended for releases)
git tag -a v1.0.0 -m "Version 1.0.0 release"

# Tag a specific commit
git tag -a v0.9.0 <commit-hash> -m "Beta release"

# Show tag details
git show v1.0.0

# Push a specific tag to remote
git push origin v1.0.0

# Push all tags
git push origin --tags

# Delete a local tag
git tag -d v1.0.0

# Delete a remote tag
git push origin --delete v1.0.0
```

---

## 📦 Stashing

Stash saves your uncommitted work temporarily so you can switch context.

```bash
# Stash current changes
git stash

# Stash with a description
git stash push -m "work in progress on login form"

# Stash including untracked files
git stash -u

# List all stashes
git stash list

# Apply the most recent stash (keeps stash)
git stash apply

# Apply a specific stash
git stash apply stash@{2}

# Pop the most recent stash (applies and removes it)
git stash pop

# Show what's in a stash
git stash show -p stash@{0}

# Create a branch from a stash
git stash branch new-branch stash@{0}

# Drop a specific stash
git stash drop stash@{1}

# Clear ALL stashes
git stash clear
```

---

## 🍒 Cherry-Picking

Apply a specific commit from another branch onto the current branch.

```bash
# Apply a specific commit to current branch
git cherry-pick <commit-hash>

# Cherry-pick without auto-committing
git cherry-pick --no-commit <commit-hash>

# Cherry-pick a range of commits
git cherry-pick abc123..def456

# Abort a cherry-pick
git cherry-pick --abort

# Continue after resolving conflicts
git cherry-pick --continue
```

---

## ⚡ Git Bisect (Find Bugs Fast)

Binary search through your commit history to find which commit introduced a bug.

```bash
# Start bisecting
git bisect start

# Mark the current commit as bad (has the bug)
git bisect bad

# Mark a known good commit (before the bug existed)
git bisect good v1.2.0

# Git checks out a midpoint commit — test it, then mark it:
git bisect good   # or
git bisect bad

# Git will narrow down to the exact bad commit automatically
# When done, reset back to where you started:
git bisect reset
```

---

## 📁 .gitignore

The `.gitignore` file tells Git which files/folders to never track.

```gitignore
# Ignore a specific file
secret.env

# Ignore all .log files
*.log

# Ignore a folder
node_modules/
dist/

# Ignore all files in any folder named 'build'
**/build/

# Ignore files with a pattern but NOT one specific file
*.txt
!important.txt

# Ignore files in root only (not subdirectories)
/config.local.js
```

```bash
# Check if a file is being ignored and why
git check-ignore -v filename.txt

# Remove a file from tracking (but keep it on disk)
git rm --cached filename.txt

# Remove a folder from tracking (keep on disk)
git rm -r --cached foldername/
```

> 💡 GitHub maintains a collection of useful `.gitignore` templates at [github.com/github/gitignore](https://github.com/github/gitignore)

---

## 🔎 Searching

```bash
# Search for a string in tracked files
git grep "search term"

# Search with line numbers
git grep -n "search term"

# Search in a specific commit
git grep "search term" v1.0.0

# Find which commit introduced/removed a string
git log -S "function myFunc"

# Find which commit changed a string (regex)
git log -G "myFunc.*args"

# See who last changed each line in a file
git blame filename.txt

# Blame with date info
git blame -w --date=short filename.txt
```

---

## 🔧 Advanced & Power Commands

```bash
# Clean untracked files from working directory
git clean -fd     # -f force, -d include directories
git clean -n      # dry run (preview what would be deleted)

# Temporarily apply a patch without committing
git apply patch.diff

# Create a patch file from commits
git format-patch HEAD~3

# Apply formatted patches
git am patches/*.patch

# Bundle a repo into a single file (for offline transfer)
git bundle create repo.bundle --all
git clone repo.bundle my-repo

# Archive a snapshot of the repo
git archive --format=zip HEAD > snapshot.zip

# Compact the repo and remove unreachable objects
git gc

# Verify the database integrity
git fsck

# Show the object type/content at a hash
git cat-file -p <hash>

# Find a lost commit (dangling commits)
git fsck --lost-found
git reflog   # see all recent HEAD movements
```

---

## 🪵 Reflog — Your Safety Net

The reflog records every movement of HEAD, even after resets and rebases. This is how you recover "lost" commits.

```bash
# View the reflog
git reflog

# Restore to a previous state (e.g. before a bad reset)
git reset --hard HEAD@{3}

# Create a branch from a reflog entry to recover work
git branch recovery-branch HEAD@{5}

# Expire old reflog entries (default 90 days)
git reflog expire --expire=90.days --all
```

---

## 🧩 Submodules

Submodules embed one Git repo inside another.

```bash
# Add a submodule
git submodule add https://github.com/user/lib.git libs/lib

# Initialize submodules after cloning
git submodule init
git submodule update

# Clone a repo AND initialize all submodules
git clone --recurse-submodules https://github.com/user/repo.git

# Update all submodules to latest
git submodule update --remote

# Remove a submodule
git submodule deinit libs/lib
git rm libs/lib
```

---

## 🔄 Common Workflows

### Feature Branch Workflow
```bash
git switch main
git pull origin main
git switch -c feature/new-thing
# ... make changes and commits ...
git push -u origin feature/new-thing
# Open a Pull Request on GitHub/GitLab
# After review and merge:
git switch main
git pull origin main
git branch -d feature/new-thing
```

### Squashing Before a PR
```bash
# Squash last 4 commits into one before merging
git rebase -i HEAD~4
# Mark all but the first as 'squash' or 'fixup'
# Edit the combined commit message
git push --force-with-lease
```

### Syncing a Forked Repo
```bash
# Add the original repo as upstream (one time)
git remote add upstream https://github.com/original/repo.git

# Fetch and merge upstream changes
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

### Hotfix on Production
```bash
git switch main
git pull origin main
git switch -c hotfix/critical-bug
# ... fix the bug ...
git commit -am "Fix critical bug"
git switch main
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix release"
git push origin main --tags
git branch -d hotfix/critical-bug
```

---

## 🧹 Cleaning Up History

```bash
# Remove a file from ALL commit history (e.g. accidentally committed secret)
# Modern approach (Git 2.24+):
git filter-repo --path secret.env --invert-paths

# Or with filter-branch (older):
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch secret.env" \
  --prune-empty --tag-name-filter cat -- --all

# After cleaning, force push all branches
git push origin --force --all
git push origin --force --tags
```

> ⚠️ If you exposed secrets, **rotate them immediately** regardless of history cleaning — assume they were already seen.

---

## 📋 Quick Reference Card

| Task | Command |
|------|---------|
| Init repo | `git init` |
| Clone repo | `git clone <url>` |
| Check status | `git status` |
| Stage file | `git add <file>` |
| Stage all | `git add .` |
| Commit | `git commit -m "msg"` |
| View log | `git log --oneline` |
| Create branch | `git switch -c <name>` |
| Switch branch | `git switch <name>` |
| Merge branch | `git merge <name>` |
| Rebase | `git rebase main` |
| Fetch remote | `git fetch origin` |
| Pull | `git pull` |
| Push | `git push origin <branch>` |
| Stash | `git stash` |
| Pop stash | `git stash pop` |
| Undo last commit | `git reset --soft HEAD~1` |
| Discard file changes | `git restore <file>` |
| Safe undo (public) | `git revert <hash>` |
| Cherry-pick | `git cherry-pick <hash>` |
| View all branches | `git branch -a` |
| Delete branch | `git branch -d <name>` |
| Tag a release | `git tag -a v1.0 -m "msg"` |
| See who changed lines | `git blame <file>` |
| Find a bug commit | `git bisect start` |
| Emergency recovery | `git reflog` |

---

## 💡 Pro Tips

- **Write atomic commits** — each commit should do one logical thing. Easier to review, revert, and cherry-pick.
- **Commit often** — small commits are easier to reason about than massive ones.
- **Never commit sensitive data** — use `.gitignore` and environment variables instead.
- **Use `--force-with-lease` instead of `--force`** when force-pushing — it won't overwrite if someone else pushed.
- **`git reflog` is your safety net** — almost nothing is truly lost in Git within 90 days.
- **Prefer `git switch` and `git restore`** over `git checkout` for clarity — checkout is overloaded with too many meanings.
- **Use SSH keys instead of HTTPS** for authentication with remotes — more secure and no password prompts.
- **Sign your commits** with a GPG key for verified authorship: `git config --global commit.gpgsign true`

---

*Git version used as reference: 2.x | All commands work on macOS, Linux, and Windows (Git Bash/WSL)*
