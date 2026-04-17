# Git: A Complete Progressive Tutorial

---

## 1. What & Why

Git is a distributed version control system. It records every change made to files in a project, lets you move back to any past state, and allows multiple people to work on the same codebase simultaneously without overwriting each other's work.

Before version control, teams emailed zip files called `project_final_v3_ACTUAL_FINAL.zip`. When something broke, finding what changed was archaeology. Git replaces that chaos with a precise, time-stamped history of every modification, who made it, and why — if you write commit messages properly.

"Distributed" means every developer has the complete history on their machine. There is no single server that must be up for you to commit, branch, or view history. GitHub and GitLab are just shared copies of that history — convenient for collaboration, not required for the tool to function.

You will use Git every single day as a professional developer. Commands like `add`, `commit`, `push`, `pull`, `branch`, and `merge` need to become muscle memory. The more advanced concepts — `rebase`, `cherry-pick`, `bisect`, `reflog` — will save you from disasters that would otherwise cost hours.

---

## 2. Mental Model

Git stores data as a series of **snapshots**, not as a list of diffs.

Think of each commit as a photograph of your entire project at one moment in time. Git stores that snapshot, then records a pointer to the previous snapshot (the "parent" commit). The history is a chain of these photographs.

```
Time ─────────────────────────────────────────────────────>

Snapshot 1   Snapshot 2   Snapshot 3   Snapshot 4  (HEAD)
[a1b2c3] --> [d4e5f6] --> [g7h8i9] --> [j0k1l2]
 "Init"       "Add login"  "Fix bug"    "Add tests"
```

Three things you're always working with simultaneously:

```
Working Directory     Staging Area (Index)     Repository (.git)
─────────────────     ────────────────────     ────────────────
Your actual files     What your next           All saved
on disk               commit will contain      snapshots

      git add ──────────────>
                      git commit ───────────>
git restore <─────────────────────────────────
```

A branch is simply a named pointer to one of these snapshots. `main` is just a label that moves forward automatically each time you commit. `HEAD` is a pointer to wherever you currently are — usually the tip of your current branch.

---

## 3. Progressive Examples

### Level 1: Daily Workflow

```bash
# One-time setup on a new machine
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"   # or "vim", "nano"

# Start tracking a project
git init                      # creates .git/ — the entire database lives here
cd existing-project
git init                      # works on existing directories too

# Clone someone else's project (most common starting point)
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder    # custom local name

# The four commands you'll use every day:

# 1. Check what's changed
git status                    # files modified, staged, or untracked
git status -s                 # compact: M=modified, A=added, ??=untracked

# 2. Stage changes
git add README.md             # stage one file
git add src/                  # stage an entire directory
git add .                     # stage all changes in current directory
git add -A                    # stage everything including deletions
git add -p README.md          # interactively choose which hunks to stage (powerful)

# 3. Commit (save the snapshot)
git commit -m "Add user authentication endpoint"
git commit -am "Fix typo in README"   # -a stages all tracked files, then commits

# 4. View history
git log --oneline             # compact one-line history
git log --oneline --graph --all --decorate   # visual branch graph
git log --stat                # show which files changed in each commit
git show a1b2c3               # full details of a specific commit
```

### Level 2: Branching and Merging

```bash
# Branches let you develop features without touching the main line

# Create and immediately switch to a new branch
git switch -c feature/user-auth        # modern syntax (Git 2.23+)
git checkout -b feature/user-auth      # older equivalent (still works)

# See all branches
git branch                  # local branches (* marks current)
git branch -a               # local + remote tracking branches

# Work normally: add, commit, add, commit
git add auth.py
git commit -m "Add password hashing with bcrypt"
git add tests/test_auth.py
git commit -m "Add unit tests for auth module"

# Merge your feature back into main
git switch main
git pull origin main         # ensure main is up to date
git merge feature/user-auth

# Two merge outcomes:
#
# Fast-forward (main had no new commits while you were on the feature branch):
# main's pointer simply advances to your last feature commit. No merge commit.
#
# Three-way merge (main has moved on):
# Git finds the common ancestor, combines both sets of changes, creates a merge commit.

# Force a merge commit even when fast-forward is possible (preserves branch history)
git merge --no-ff feature/user-auth

# Squash all feature commits into one staged change (then commit manually)
git merge --squash feature/user-auth
git commit -m "Add user authentication (squashed)"

# Clean up
git branch -d feature/user-auth        # -d refuses if unmerged
git branch -D feature/user-auth        # force delete regardless

# Resolve a merge conflict:
# When git reports a conflict, open the file — it contains markers like:
#
# <<<<<<< HEAD
# result = connect(host, port=443)
# =======
# result = connect(host, port=8080, timeout=30)
# >>>>>>> feature/user-auth
#
# Edit the file to the correct content, then:
git add conflicted_file.py
git commit           # Git auto-writes the merge commit message
```

### Level 3: Remotes and Collaboration

```bash
# Remote = a named URL pointing to another copy of the repo
git remote -v                              # list remotes and their URLs
git remote add origin https://github.com/you/repo.git   # add a remote
git remote set-url origin git@github.com:you/repo.git   # change URL (e.g., to SSH)

# Push: send your commits to the remote
git push origin main                       # push local main to remote main
git push -u origin feature/auth            # -u sets upstream tracking (needed once)
git push                                   # after -u: pushes current branch to tracked remote

# Fetch: download remote changes WITHOUT merging them
git fetch origin                           # updates remote tracking branches
git fetch --all                            # fetch from all remotes

# Pull: fetch + merge in one step
git pull                                   # pulls from tracked upstream
git pull origin main                       # explicit
git pull --rebase                          # fetch + rebase instead of merge (cleaner history)

# Delete a remote branch
git push origin --delete feature/old-thing

# Force push — overwrites remote history
git push --force                           # DANGEROUS: can delete others' work
git push --force-with-lease               # SAFE: refuses if remote has commits you haven't fetched
# Always use --force-with-lease over --force

# Syncing a fork with the original repo:
git remote add upstream https://github.com/original/repo.git   # one-time
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

### Level 4: Undoing Things

```bash
# Undo a staged file (move from staging area back to working directory)
git restore --staged auth.py

# Discard working directory changes (IRREVERSIBLE — no recycle bin)
git restore auth.py           # discard changes to one file
git restore .                 # discard ALL unstaged changes

# Fix the last commit (before pushing)
git commit --amend -m "Correct commit message"
git commit --amend --no-edit  # add staged changes to last commit, keep message

# Undo commits — the three modes of git reset
git reset --soft HEAD~1       # undo last commit, keep changes STAGED
git reset --mixed HEAD~1      # undo last commit, keep changes UNSTAGED (default)
git reset --hard HEAD~1       # undo last commit, DISCARD changes permanently

# HEAD~1 = one commit before HEAD
# HEAD~3 = three commits before HEAD
# You can also use a specific hash: git reset --soft a1b2c3

# Safe undo for shared branches — creates a NEW commit that reverses changes
git revert HEAD               # revert the last commit
git revert a1b2c3             # revert a specific commit
# This is safe because it doesn't rewrite history — others can still pull

# Emergency recovery: reflog keeps a record of every position HEAD has been in
git reflog                    # shows all recent HEAD movements with their hashes
git reset --hard HEAD@{3}     # go back to HEAD's position 3 moves ago
git branch recovery HEAD@{5}  # create a branch from a past state
# Reflog entries are kept for 90 days by default — nearly nothing is truly lost
```

### Level 5: Rebase, Stash, and Advanced Operations

```bash
# --- Rebase: rewrite history for a clean linear timeline ---
#
# Scenario: you branched from main, main has moved on, you want your
# commits to appear ON TOP of the current main:
#
# Before:           After rebase:
# A-B-C (main)      A-B-C-D (main)
#   \                       \
#    D-E (feature)           D'-E' (feature)
#
git switch feature/auth
git rebase main               # replays your commits on top of current main

# Interactive rebase: clean up your commits before a PR
git rebase -i HEAD~4          # rewrite the last 4 commits interactively
# In the editor, you can:
# pick   = keep the commit
# reword = keep but edit the message
# squash = combine with previous commit
# fixup  = combine and discard this commit's message
# drop   = delete this commit entirely

git rebase --abort            # bail out if anything goes wrong
git rebase --continue         # after resolving a conflict, resume

# RULE: never rebase commits that have been pushed to a shared branch.
# Rebase rewrites commit hashes. Others' history will diverge.

# --- Stash: temporarily shelve your work ---
git stash                             # save current changes
git stash push -m "WIP: add caching" # save with a label
git stash -u                          # also stash untracked files

git stash list                        # show all stashes
git stash pop                         # restore most recent stash and delete it
git stash apply stash@{2}            # restore a specific stash, keep it in the list
git stash branch fix/auth stash@{0} # create a branch from a stash entry
git stash drop stash@{1}            # delete one stash
git stash clear                      # delete all stashes

# --- Cherry-pick: apply one specific commit to current branch ---
git cherry-pick a1b2c3               # copy that commit here
git cherry-pick abc..def             # copy a range of commits
git cherry-pick --no-commit a1b2c3  # apply changes but don't auto-commit

# --- Bisect: binary-search for the commit that introduced a bug ---
git bisect start
git bisect bad                        # current commit is broken
git bisect good v2.1.0               # this tag was working
# Git checks out the midpoint — test it, then mark:
git bisect good                       # if the midpoint is fine
git bisect bad                        # if the midpoint is broken
# Repeat until Git identifies the exact bad commit
git bisect reset                      # return to your original branch

# --- Tags: named markers for releases ---
git tag v1.0.0                        # lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0" # annotated tag (recommended — has metadata)
git push origin v1.0.0               # push a specific tag
git push origin --tags               # push all tags
git tag -d v1.0.0                    # delete local tag
git push origin --delete v1.0.0     # delete remote tag
```

### Level 6: Professional Workflows

```bash
# Feature branch workflow (the industry standard for teams):

# 1. Start from an up-to-date main
git switch main && git pull origin main

# 2. Create a descriptively named branch
git switch -c feature/TICKET-123-user-auth

# 3. Make small, focused commits (one logical change per commit)
git add -p          # stage only the relevant changes
git commit -m "Add bcrypt password hashing"
git add -p
git commit -m "Add login endpoint with rate limiting"

# 4. Keep your branch up to date while others work
git fetch origin
git rebase origin/main    # replay your commits on top of latest main

# 5. Clean up commits before the PR
git rebase -i origin/main   # squash WIP commits, fix messages

# 6. Push and open PR
git push -u origin feature/TICKET-123-user-auth

# 7. After PR is merged, clean up
git switch main && git pull origin main
git branch -d feature/TICKET-123-user-auth

# Hotfix workflow:
git switch main && git pull
git switch -c hotfix/CVE-2024-1234
# fix the vulnerability
git commit -am "Patch SQL injection in login endpoint"
git switch main
git merge --no-ff hotfix/CVE-2024-1234
git tag -a v1.0.1 -m "Security patch"
git push origin main --tags
git branch -d hotfix/CVE-2024-1234

# Remove a file accidentally committed (credentials, secrets):
# First, rotate the credentials immediately — treat them as compromised.
git filter-repo --path .env --invert-paths   # modern approach (install: pip install git-filter-repo)
git push origin --force --all
git push origin --force --tags
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Committing directly to main in a team environment**

```bash
# WRONG: working directly on main
git switch main
# ... make changes ...
git commit -am "Add login feature"
git push origin main    # now your experimental work is on the shared branch

# CORRECT: always branch
git switch -c feature/login
# ... make changes ...
git commit -am "Add login feature"
git push -u origin feature/login   # open a pull request for review
```

**Mistake 2: Using `git reset --hard` on shared commits**

```bash
# You pushed 3 commits. A colleague pulled them. Now you:
git reset --hard HEAD~3    # rewrites your local history
git push --force           # OVERWRITES the remote history

# Your colleague's next git pull will fail with conflicts.
# Their local commits now diverge from the rewritten remote.

# CORRECT: if you need to undo shared commits, use revert
git revert HEAD~3..HEAD    # creates three new "undo" commits
git push                   # safe — no history rewrite
```

**Mistake 3: Misunderstanding what `git add .` stages**

```bash
# git add . stages everything in the CURRENT DIRECTORY and below
# If you're in a subdirectory, it only stages changes in that subdirectory

cd src/
git add .    # stages only src/ changes, NOT changes in tests/ or root

# git add -A stages ALL changes in the ENTIRE REPOSITORY regardless of location
git add -A   # always stages everything — no location dependency
```

**Mistake 4: Confusing `git fetch` and `git pull`**

```bash
# git fetch: downloads remote changes, updates remote tracking branches,
# does NOT touch your working directory or local branches
git fetch origin    # safe — nothing in your workspace changes

# git pull: fetch + merge (or rebase with --rebase)
# CAN create merge commits or conflicts in your working directory
git pull            # may trigger a merge conflict

# Professional workflow: always fetch first, review, then merge/rebase
git fetch origin
git log HEAD..origin/main --oneline    # see what's incoming
git rebase origin/main                 # apply your commits on top
```

**Mistake 5: Writing terrible commit messages**

```bash
# WRONG: messages that describe what you did, not what changed or why
git commit -m "fix"
git commit -m "changes"
git commit -m "asdf"
git commit -m "WIP"
git commit -m "update stuff"

# CORRECT: imperative mood, specific, under 72 characters for subject
git commit -m "Fix null pointer exception in payment processor"
git commit -m "Add Redis caching to reduce database load by ~60%"
git commit -m "Remove deprecated getUserById API — replaced by getUser"
# For multi-line: leave a blank line between subject and body
git commit    # opens editor for full message with subject + body
```

**Mistake 6: Stashing instead of branching for long-lived work**

```bash
# WRONG: using stash as a holding area for days
git stash    # hide unfinished feature
git stash    # hide another unfinished thing
# ... days pass ...
git stash list    # stash@{0}: stash@{1}: stash@{2}:
# Good luck remembering what each one contains

# CORRECT: branches are free, create one immediately
git switch -c wip/caching-experiment
git commit -am "WIP: initial caching implementation — do not merge"
# Your work is saved, documented, and easy to return to
```

---

## 5. The "Why Does This Work" Layer

### How Git Stores Data

Git is not a delta-based system (it doesn't store "what changed from the previous version"). It stores complete snapshots. But it uses content-addressable storage with SHA-1 hashing (moving toward SHA-256) to avoid storing the same content twice.

```
Every object in Git is identified by the SHA-1 hash of its contents.
The same file content always produces the same hash — same hash, same content.

Object types:
  blob    = contents of one file
  tree    = directory listing (filename -> blob hash pairs)
  commit  = pointer to a tree + parent commit hash + metadata
  tag     = pointer to a commit with a label and signature
```

When you commit, Git:
1. Creates blob objects for any new/changed file contents
2. Creates a tree object for each directory
3. Creates a commit object pointing to the top-level tree, parent commit, your name, timestamp, message
4. Moves the current branch pointer to point to the new commit

This design means history is immutable. A commit's hash is computed from its content AND its parent's hash. Change anything, and the hash changes, which changes every subsequent commit's hash. This is why "rewriting history" creates new commits with new hashes — the old ones still exist until garbage-collected.

### How Rebase Works Internally

`git rebase main` doesn't "move" your commits. It:
1. Finds the common ancestor of your branch and `main`
2. Temporarily saves the diffs of each of your commits since that ancestor
3. Fast-forwards your branch to the tip of `main`
4. Replays each saved diff as a new commit on top, generating new commit objects with new hashes

The old commits are orphaned — no branch points to them, but they exist in the reflog for 90 days. This is why `git rebase --abort` works: Git just moves HEAD back to where it was.

### The Staging Area Is Not Optional Ceremony

Beginners often bypass staging with `git commit -am`. The staging area exists for precision. In a single working session, you might:
- Fix a bug in `auth.py`
- Refactor a helper in `utils.py`
- Add a new feature in `dashboard.py`

Without staging, your choices are "commit everything" or "commit nothing." With staging, you commit each logical change separately, giving your history semantic meaning:

```bash
git add -p auth.py        # stage only the bug fix hunks
git commit -m "Fix token expiry check in auth middleware"

git add -p utils.py dashboard.py   # stage the refactor and feature
git commit -m "Refactor URL helpers, add dashboard stats endpoint"
```

This granularity makes `git log`, `git blame`, `git bisect`, and `git revert` dramatically more useful.

---

## 6. Quick Reference

### Daily Commands

| Task | Command |
|------|---------|
| Stage file | `git add <file>` |
| Stage all | `git add .` |
| Commit | `git commit -m "msg"` |
| Check status | `git status` |
| View history | `git log --oneline` |
| Show diff (unstaged) | `git diff` |
| Show diff (staged) | `git diff --staged` |

### Branching

| Task | Command |
|------|---------|
| Create + switch | `git switch -c <name>` |
| Switch branch | `git switch <name>` |
| List all branches | `git branch -a` |
| Delete branch | `git branch -d <name>` |
| Merge into current | `git merge <branch>` |
| Rebase onto branch | `git rebase <branch>` |

### Remotes

| Task | Command |
|------|---------|
| Push (first time) | `git push -u origin <branch>` |
| Push | `git push` |
| Pull | `git pull` |
| Fetch only | `git fetch origin` |
| Safe force push | `git push --force-with-lease` |

### Undoing

| Scenario | Command |
|----------|---------|
| Unstage file | `git restore --staged <file>` |
| Discard file changes | `git restore <file>` |
| Undo last commit (keep staged) | `git reset --soft HEAD~1` |
| Undo last commit (keep unstaged) | `git reset --mixed HEAD~1` |
| Undo last commit (discard) | `git reset --hard HEAD~1` |
| Safe undo (shared branch) | `git revert <hash>` |
| Recover anything | `git reflog` |

### Advanced

| Task | Command |
|------|---------|
| Stash changes | `git stash` |
| Restore stash | `git stash pop` |
| Copy specific commit | `git cherry-pick <hash>` |
| Find bad commit | `git bisect start` |
| Rewrite recent commits | `git rebase -i HEAD~N` |
| Create release tag | `git tag -a v1.0 -m "msg"` |
| Blame: who wrote this line | `git blame <file>` |
