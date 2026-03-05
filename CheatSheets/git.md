# Git Reference Guide
> A comprehensive reference for developers at every level — from initial setup to advanced workflows.

---

## What is Git?

Git is a **distributed version control system (VCS)**. It tracks changes to files over time, enables team collaboration, and allows you to revert your project to any previous state. Every developer maintains a full copy of the project history locally — there is no single point of failure.

### Core Terminology

| Term | Definition |
|------|------------|
| **Repository (repo)** | A project folder tracked by Git, containing all files and their full history |
| **Commit** | A saved snapshot of your files at a specific point in time |
| **Branch** | An independent line of development, isolated from other branches |
| **Remote** | A copy of your repository hosted on a server (e.g. GitHub, GitLab) |
| **HEAD** | A pointer to the commit you are currently working from |
| **Staging Area (Index)** | A preparation area where changes are collected before being committed |
| **Working Directory** | The actual files on your disk that you read and edit |

### The Three States of Git

```
Working Directory  ->  Staging Area (Index)  ->  Repository (.git)
    (edit files)          (git add)                 (git commit)
```

---

## First-Time Setup

These commands configure your global Git identity and preferences. Run them once on any new machine.

```bash
# Set your name and email (required before your first commit)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set the default branch name to 'main'
git config --global init.defaultBranch main

# Set your preferred text editor
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "vim"           # Vim

# Enable colored terminal output
git config --global color.ui auto

# View all configuration settings
git config --list

# View a specific setting
git config user.name
```

---

## Starting a Repository

```bash
# Initialize a new Git repository in the current folder
git init

# Initialize with a specific default branch name
git init -b main

# Clone an existing remote repository to your machine
git clone https://github.com/user/repo.git

# Clone into a custom local folder name
git clone https://github.com/user/repo.git my-folder

# Clone a specific branch only
git clone -b develop https://github.com/user/repo.git

# Shallow clone — downloads only the latest snapshot (faster for large repos)
git clone --depth 1 https://github.com/user/repo.git
```

---

## Checking Status and History

```bash
# Show modified, staged, and untracked files
git status

# Compact status output
git status -s

# View the full commit history
git log

# View history in condensed one-line format
git log --oneline

# View a visual graph of all branches
git log --oneline --graph --all --decorate

# Show file change statistics per commit
git log --stat

# Show the full diff for each commit
git log -p

# Show only the last N commits
git log -5

# Search commit history by message text
git log --grep="fix bug"

# Filter commits by author
git log --author="Alice"

# Filter commits within a date range
git log --after="2024-01-01" --before="2024-06-01"

# Show the full details of a specific commit
git show <commit-hash>

# Show the state of a file at a specific commit
git show <commit-hash>:path/to/file.txt
```

---

## Staging and Committing

```bash
# Stage a specific file
git add filename.txt

# Stage all changes in the current directory
git add .

# Stage all changes including deletions (entire repo)
git add -A

# Interactively stage parts of a file (hunk by hunk)
git add -p filename.txt

# Unstage a file while keeping your changes in the working directory
git restore --staged filename.txt

# Commit staged changes with an inline message
git commit -m "Your commit message"

# Stage all tracked files and commit in a single step
git commit -am "Your message"

# Open your configured editor to write a detailed commit message
git commit

# Amend the most recent commit (change its message or content)
git commit --amend -m "Corrected message"

# Amend the most recent commit without changing its message
git commit --amend --no-edit
```

> **Writing good commit messages:** Use the imperative mood — write "Add login feature" rather than "Added login feature". Keep the subject line under 72 characters.

---

## Viewing Differences

```bash
# Show unstaged changes (working directory vs. staging area)
git diff

# Show staged changes (staging area vs. last commit)
git diff --staged
git diff --cached       # Equivalent to the above

# Compare two branches
git diff main..feature-branch

# Compare two specific commits
git diff abc123..def456

# List only the names of changed files
git diff --name-only

# Show the diff for a specific file only
git diff filename.txt

# Show a word-level diff (more readable for prose or config changes)
git diff --word-diff
```

---

## Branches

```bash
# List all local branches
git branch

# List all branches — local and remote
git branch -a

# List remote branches only
git branch -r

# Create a new branch without switching to it
git branch feature-login

# Switch to an existing branch
git switch feature-login
git checkout feature-login      # Older equivalent

# Create a new branch and switch to it immediately
git switch -c feature-login
git checkout -b feature-login   # Older equivalent

# Rename a specific branch
git branch -m old-name new-name

# Rename the current branch
git branch -m new-name

# Delete a branch (safe — will not delete unmerged branches)
git branch -d feature-login

# Force-delete a branch regardless of merge status
git branch -D feature-login

# Visualize which branch each commit belongs to
git log --oneline --graph --all
```

---

## Merging

Merging integrates the changes from one branch into another.

```bash
# Merge a branch into the current branch
git merge feature-login

# Merge and always create a merge commit (even when fast-forward is possible)
git merge --no-ff feature-login

# Squash all commits from the branch into one staged change, then commit manually
git merge --squash feature-login
git commit -m "Add login feature"

# Abort a merge that is in progress (e.g. when conflicts arise)
git merge --abort
```

**Fast-Forward vs. No-Fast-Forward**

| Strategy | Behavior |
|----------|----------|
| **Fast-forward** | If no new commits exist on the target branch, Git moves the pointer forward. No merge commit is created. |
| **No-fast-forward (`--no-ff`)** | Always creates a merge commit, preserving the explicit history of the feature branch. |

---

## Rebasing

Rebasing replays commits from one branch on top of another, producing a clean, **linear history**.

```bash
# Rebase the current branch onto main
git rebase main

# Interactive rebase — rewrite, squash, or reorder the last N commits
git rebase -i HEAD~3

# Abort a rebase in progress
git rebase --abort

# Continue a rebase after resolving conflicts
git rebase --continue

# Skip the current commit during a rebase
git rebase --skip
```

### Interactive Rebase Commands

| Command | Effect |
|---------|--------|
| `pick` | Keep the commit unchanged |
| `reword` | Keep the commit but edit its message |
| `edit` | Pause at this commit to make amendments |
| `squash` | Combine this commit with the one before it |
| `fixup` | Like `squash`, but discard this commit's message |
| `drop` | Remove this commit entirely |

> **Important:** Never rebase commits that have already been pushed to a shared or public branch. Rewriting shared history causes serious problems for other collaborators.

---

## Working with Remotes

```bash
# List all remote connections
git remote -v

# Add a new remote
git remote add origin https://github.com/user/repo.git

# Rename a remote
git remote rename origin upstream

# Remove a remote
git remote remove origin

# Update the URL of an existing remote
git remote set-url origin https://github.com/user/new-repo.git

# Fetch all changes from a remote without merging
git fetch origin

# Fetch a specific branch from a remote
git fetch origin main

# Pull — fetch and merge into the current branch
git pull

# Pull using rebase instead of merge
git pull --rebase

# Push the current branch to a remote
git push origin main

# Push and set the upstream tracking branch
git push -u origin feature-branch

# Push all local branches
git push --all origin

# Push all tags to the remote
git push --tags

# Delete a branch on the remote
git push origin --delete feature-branch

# Force push (use with caution — overwrites remote history)
git push --force

# Safer force push — aborts if the remote has received new commits
git push --force-with-lease
```

---

## Undoing Changes

### Command Overview

| Scenario | Command |
|----------|---------|
| Discard local file changes | `git restore filename` |
| Unstage a file | `git restore --staged filename` |
| Undo last commit, keep work staged | `git reset --soft HEAD~1` |
| Undo commits on a private branch | `git reset --hard` |
| Undo commits on a shared branch | `git revert <hash>` |

```bash
# Discard unstaged changes in a file (this is irreversible)
git restore filename.txt
git checkout -- filename.txt    # Older equivalent

# Discard all unstaged changes across the entire repo
git restore .

# Unstage a file while keeping the changes in your working directory
git restore --staged filename.txt

# Undo the last commit and keep changes staged
git reset --soft HEAD~1

# Undo the last commit and leave changes unstaged (default behavior)
git reset --mixed HEAD~1

# Undo the last commit and permanently discard all changes
git reset --hard HEAD~1

# Create a new commit that reverses a previous one (safe for shared branches)
git revert <commit-hash>

# Revert a range of commits
git revert HEAD~3..HEAD

# Revert a commit without automatically creating a new commit
git revert --no-commit <commit-hash>
```

---

## Tags

Tags mark specific points in history — most commonly used for version releases.

```bash
# List all tags
git tag

# Create a lightweight tag (a named pointer only)
git tag v1.0.0

# Create an annotated tag with a message (recommended for releases)
git tag -a v1.0.0 -m "Version 1.0.0 release"

# Tag a specific past commit
git tag -a v0.9.0 <commit-hash> -m "Beta release"

# Show the details of a tag
git show v1.0.0

# Push a specific tag to the remote
git push origin v1.0.0

# Push all tags to the remote
git push origin --tags

# Delete a tag locally
git tag -d v1.0.0

# Delete a tag on the remote
git push origin --delete v1.0.0
```

---

## Stashing

Stashing temporarily shelves uncommitted changes so you can switch tasks without losing work.

```bash
# Stash current changes
git stash

# Stash with a descriptive label
git stash push -m "WIP: login form validation"

# Include untracked files in the stash
git stash -u

# List all saved stashes
git stash list

# Apply the most recent stash (stash entry is preserved)
git stash apply

# Apply a specific stash by index
git stash apply stash@{2}

# Apply and remove the most recent stash
git stash pop

# Inspect the contents of a specific stash
git stash show -p stash@{0}

# Create a new branch from a stash entry
git stash branch new-branch stash@{0}

# Delete a specific stash entry
git stash drop stash@{1}

# Delete all stash entries
git stash clear
```

---

## Cherry-Picking

Cherry-picking applies a specific commit from one branch onto the current branch — without merging the entire branch.

```bash
# Apply a specific commit to the current branch
git cherry-pick <commit-hash>

# Apply a commit without auto-committing (useful for review before commit)
git cherry-pick --no-commit <commit-hash>

# Apply a range of commits
git cherry-pick abc123..def456

# Abort a cherry-pick in progress
git cherry-pick --abort

# Continue a cherry-pick after resolving conflicts
git cherry-pick --continue
```

---

## Git Bisect — Locating Bugs with Binary Search

`git bisect` automates a binary search through your commit history to pinpoint the exact commit that introduced a bug.

```bash
# Begin a bisect session
git bisect start

# Mark the current commit as faulty
git bisect bad

# Mark a known-good commit (before the bug existed)
git bisect good v1.2.0

# Git will check out a midpoint commit — test it, then mark it accordingly
git bisect good
git bisect bad

# Git narrows down automatically until the offending commit is identified
# When finished, exit the bisect session and return to your original branch
git bisect reset
```

---

## .gitignore

The `.gitignore` file instructs Git to exclude specific files and directories from tracking.

```gitignore
# Ignore a specific file
secret.env

# Ignore all files with a given extension
*.log

# Ignore entire directories
node_modules/
dist/

# Ignore a folder name at any depth
**/build/

# Ignore a pattern, but exempt one specific file
*.txt
!important.txt

# Ignore a file only in the root directory (not subdirectories)
/config.local.js
```

```bash
# Check whether a file is being ignored and why
git check-ignore -v filename.txt

# Stop tracking a file without deleting it from disk
git rm --cached filename.txt

# Stop tracking an entire directory without deleting it
git rm -r --cached foldername/
```

> GitHub maintains an extensive collection of `.gitignore` templates at [github.com/github/gitignore](https://github.com/github/gitignore).

---

## Searching

```bash
# Search for a string across all tracked files
git grep "search term"

# Search with line numbers shown
git grep -n "search term"

# Search within a specific tagged commit or branch
git grep "search term" v1.0.0

# Find the commit that introduced or removed a specific string
git log -S "function myFunc"

# Find commits that changed a string matching a pattern (regex)
git log -G "myFunc.*args"

# View the last author to modify each line in a file
git blame filename.txt

# Blame output with date information
git blame -w --date=short filename.txt
```

---

## Advanced Commands

```bash
# Remove untracked files and directories from the working directory
git clean -fd           # -f = force, -d = include directories
git clean -n            # Dry run — preview what would be removed

# Apply a patch file without committing
git apply patch.diff

# Generate patch files from the last N commits
git format-patch HEAD~3

# Apply a series of formatted patch files
git am patches/*.patch

# Bundle the entire repository into a single portable file
git bundle create repo.bundle --all
git clone repo.bundle my-repo

# Export a snapshot of the repository as a ZIP archive
git archive --format=zip HEAD > snapshot.zip

# Optimize the repository by compressing and cleaning up objects
git gc

# Verify the integrity of the Git object database
git fsck

# Inspect the contents of a Git object by its hash
git cat-file -p <hash>

# Locate dangling (unreachable) commits
git fsck --lost-found
git reflog              # Review all recent HEAD movements
```

---

## Reflog — Emergency Recovery

The reflog is a local log of every position HEAD has occupied, including positions lost after resets and rebases. It is your primary tool for recovering from destructive operations.

```bash
# View the full reflog
git reflog

# Restore HEAD to a previous state
git reset --hard HEAD@{3}

# Create a recovery branch from a reflog entry
git branch recovery-branch HEAD@{5}

# Manually expire old reflog entries (default retention is 90 days)
git reflog expire --expire=90.days --all
```

---

## Submodules

Submodules allow you to embed one Git repository inside another, keeping them versioned independently.

```bash
# Add a submodule to your project
git submodule add https://github.com/user/lib.git libs/lib

# Initialize submodules after cloning a project
git submodule init
git submodule update

# Clone a repository and automatically initialize all submodules
git clone --recurse-submodules https://github.com/user/repo.git

# Update all submodules to their latest remote commits
git submodule update --remote

# Remove a submodule
git submodule deinit libs/lib
git rm libs/lib
```

---

## Common Workflows

### Feature Branch Workflow

```bash
git switch main
git pull origin main
git switch -c feature/new-thing

# Make your changes and commit them
git push -u origin feature/new-thing

# Open a Pull Request on GitHub or GitLab for review
# After the PR is approved and merged:

git switch main
git pull origin main
git branch -d feature/new-thing
```

### Squashing Commits Before a Pull Request

```bash
# Consolidate the last 4 commits into a single clean commit
git rebase -i HEAD~4
# In the editor: mark all commits except the first as 'squash' or 'fixup'
# Update the combined commit message as needed
git push --force-with-lease
```

### Syncing a Forked Repository

```bash
# Register the original repository as an upstream remote (one-time step)
git remote add upstream https://github.com/original/repo.git

# Fetch and merge upstream changes into your local main
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

# Apply and commit your fix
git commit -am "Fix critical bug"

git switch main
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix release"
git push origin main --tags
git branch -d hotfix/critical-bug
```

---

## Cleaning Up History

Use these commands to remove a file that was accidentally committed — such as credentials or sensitive configuration.

```bash
# Remove a file from all commit history — modern approach (Git 2.24+)
git filter-repo --path secret.env --invert-paths

# Remove a file from all commit history — older approach
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch secret.env" \
  --prune-empty --tag-name-filter cat -- --all

# Force push all branches and tags after rewriting history
git push origin --force --all
git push origin --force --tags
```

> **Security notice:** If you accidentally committed secrets or credentials, rotate them immediately — regardless of whether you clean the history. Treat them as compromised from the moment they were pushed.

---

## Quick Reference

| Task | Command |
|------|---------|
| Initialize a repo | `git init` |
| Clone a repo | `git clone <url>` |
| Check status | `git status` |
| Stage a file | `git add <file>` |
| Stage everything | `git add .` |
| Commit with message | `git commit -m "msg"` |
| View commit log | `git log --oneline` |
| Create a branch | `git switch -c <name>` |
| Switch branch | `git switch <name>` |
| Merge branch | `git merge <name>` |
| Rebase onto main | `git rebase main` |
| Fetch from remote | `git fetch origin` |
| Pull latest | `git pull` |
| Push branch | `git push origin <branch>` |
| Stash changes | `git stash` |
| Restore stash | `git stash pop` |
| Undo last commit | `git reset --soft HEAD~1` |
| Discard file changes | `git restore <file>` |
| Safe undo (shared branch) | `git revert <hash>` |
| Cherry-pick a commit | `git cherry-pick <hash>` |
| List all branches | `git branch -a` |
| Delete a branch | `git branch -d <name>` |
| Create a release tag | `git tag -a v1.0 -m "msg"` |
| View line authorship | `git blame <file>` |
| Find a bad commit | `git bisect start` |
| Recover lost work | `git reflog` |

---

## Best Practices

- **Write atomic commits.** Each commit should represent a single logical change. This makes commits easier to review, revert, and cherry-pick.
- **Commit frequently.** Small, focused commits are far easier to understand and debug than large, sweeping ones.
- **Never commit sensitive data.** Keep secrets, credentials, and environment-specific values out of version control. Use `.gitignore` and environment variables.
- **Prefer `--force-with-lease` over `--force`.** When force-pushing, `--force-with-lease` will refuse to overwrite if someone else has pushed to the branch since your last fetch.
- **Use `git reflog` as your safety net.** Almost nothing is truly lost in Git within the default 90-day retention window.
- **Prefer `git switch` and `git restore` over `git checkout`.** The `checkout` command carries multiple unrelated responsibilities. The newer commands are clearer and less error-prone.
- **Use SSH keys for remote authentication.** SSH is more secure than HTTPS and eliminates repeated password prompts.
- **Sign your commits.** Use a GPG key to verify authorship: `git config --global commit.gpgsign true`

---

*Reference based on Git 2.x. All commands are compatible with macOS, Linux, and Windows (Git Bash / WSL).*
