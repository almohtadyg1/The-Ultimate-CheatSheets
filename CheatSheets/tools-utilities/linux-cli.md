# The Linux Command Line: A Complete Progressive Tutorial

---

## 1. What & Why

The Linux command line — the shell — is a text interface to your operating system. You type a command, press Enter, the OS executes it, and prints output back to you. No mouse, no icons, just text.

Why does this matter? Three reasons. First, automation: a task that takes 20 clicks in a GUI becomes one line of text you can repeat, schedule, and pipe into other commands. Second, power: the tools built into every Linux system — `grep`, `awk`, `sed`, `find` — can process gigabytes of data faster than most GUI tools can open a file. Third, remote access: every server you will ever work with, you will manage through a terminal over SSH. There is no other interface.

The shell is also where debugging happens. When something breaks in production, you SSH in, tail the logs, check processes, inspect disk space, kill runaway processes. Competence here is not optional for anyone who ships software.

This tutorial covers `bash`, the most common shell on Linux systems. The syntax also works in `zsh` (default on macOS) with minor differences noted where relevant.

---

## 2. Mental Model

Think of the Linux filesystem as an inverted tree with the root `/` at the top. Every file and directory is a node in that tree. Your current location in the tree is your *working directory*.

```
/                        <- root (the top of everything)
├── bin/                 <- essential system binaries (ls, cp, bash)
├── etc/                 <- system configuration files
├── home/                <- home directories for users
│   ├── alice/           <- /home/alice — Alice's home
│   └── bob/             <- /home/bob — Bob's home
├── var/
│   └── log/             <- system log files
├── usr/
│   └── local/
│       └── bin/         <- programs you install yourself
└── tmp/                 <- temporary files, cleared on reboot
```

Every command you type is a program that lives somewhere in this tree (usually `/bin`, `/usr/bin`, or `/usr/local/bin`). When you type `ls`, the shell finds the `ls` binary, runs it, and prints its output.

The shell itself is just another program — `bash` — that reads your input, interprets it, runs programs, and connects their inputs and outputs together. This piping — connecting programs together — is the core philosophy of Unix: small tools that do one thing well, combined arbitrarily.

---

## 3. Progressive Examples

### Level 1: Navigation and File Operations

```bash
# Where am I?
pwd
# /home/alice

# List files in current directory
ls          # basic list
ls -l       # long format: permissions, size, date, name
ls -lh      # same, but sizes in human-readable form (KB, MB)
ls -la      # include hidden files (names starting with .)
ls -lah     # all three — the combination you'll use constantly

# Move around
cd /var/log         # absolute path — starts from root /
cd Documents        # relative path — relative to current directory
cd ..               # go up one level
cd ~                # go to your home directory (same as cd $HOME)
cd -                # go back to the previous directory

# Create things
mkdir projects                      # create a directory
mkdir -p projects/web/frontend      # create nested directories in one shot

touch notes.txt                     # create an empty file (or update timestamp)
echo "Hello, world" > notes.txt     # create file with content (overwrites)
echo "Second line" >> notes.txt     # append to file (does NOT overwrite)

# Copy, move, delete
cp notes.txt backup.txt             # copy file
cp -r projects/ projects_backup/    # copy directory recursively
mv notes.txt ~/Documents/           # move file to another location
mv old_name.txt new_name.txt        # rename a file (mv to same dir with new name)

rm file.txt                         # delete file (no recycle bin — permanent)
rm -r old_project/                  # delete directory and all contents
rm -i important.txt                 # -i asks for confirmation before deleting

# View file contents
cat server.log                      # print entire file to terminal
less server.log                     # paginate (q to quit, / to search, n for next match)
head -n 20 server.log               # first 20 lines
tail -n 20 server.log               # last 20 lines
tail -f server.log                  # follow — shows new lines as they're written (great for watching logs)
```

### Level 2: Searching and Finding

```bash
# grep: search inside files
grep "ERROR" server.log             # find lines containing "ERROR"
grep -i "error" server.log          # case-insensitive
grep -n "error" server.log          # show line numbers
grep -r "TODO" ./src/               # search recursively in directory
grep -v "DEBUG" server.log          # invert: lines that do NOT match
grep -c "ERROR" server.log          # count of matching lines
grep -E "ERROR|WARN" server.log     # extended regex: match ERROR or WARN
grep -A 3 "FATAL" server.log        # print 3 lines AFTER each match (context)
grep -B 3 "FATAL" server.log        # print 3 lines BEFORE each match

# find: locate files in the filesystem
find . -name "*.py"                 # find Python files in current directory
find /var/log -name "*.log" -type f # files only (not directories)
find . -type d -name "__pycache__"  # find directories named __pycache__
find . -size +10M                   # files larger than 10 megabytes
find . -mtime -7                    # files modified within the last 7 days
find . -name "*.tmp" -delete        # find and delete (no confirmation — careful)
find . -name "*.sh" -exec chmod +x {} \;   # find and execute a command on each result
# {} is placeholder for the found file, \; ends the -exec expression

# which / whereis: find where a program is installed
which python3                       # /usr/bin/python3
whereis nginx                       # shows binary, source, and man page locations
```

### Level 3: Pipes, Redirection, and Text Processing

This is where the real power emerges. Pipes connect programs together: the output of one command becomes the input of the next.

```bash
# | (pipe): send output of left command to input of right command
ps aux | grep nginx              # list processes, then filter for nginx
cat server.log | grep ERROR | wc -l    # count error lines
ls -la | sort -k5 -n             # list files, sort by 5th field (size) numerically
history | tail -20               # last 20 commands you ran

# > and >>: redirect output to a file
ls -la > filelist.txt            # write output to file (overwrites if exists)
echo "new entry" >> filelist.txt # append — does not overwrite
command 2> errors.log            # redirect stderr (error output) to file
command > output.log 2>&1        # redirect both stdout and stderr to same file
command &> output.log            # shorthand for above (bash only)
command > /dev/null 2>&1         # discard all output (silence a command)

# < : redirect a file as input
sort < names.txt                 # sort reads from file instead of keyboard
wc -l < server.log               # count lines using file as stdin

# tee: write to file AND display on screen simultaneously
make 2>&1 | tee build.log        # see build output live AND save it

# --- Text Processing Tools ---

# wc: word count
wc -l server.log                 # line count
wc -w document.txt               # word count
wc -c binary.dat                 # byte count

# sort and uniq
sort names.txt                   # alphabetical sort
sort -r names.txt                # reverse
sort -n numbers.txt              # numeric sort
sort -u names.txt                # sort and remove duplicates (unique)
sort server.log | uniq -c | sort -rn   # count occurrences, sort by frequency

# cut: extract columns
cut -d',' -f1,3 data.csv         # extract columns 1 and 3, comma-delimited
cut -d':' -f1 /etc/passwd        # extract usernames from passwd file

# awk: column-oriented text processing
awk '{print $1, $3}' access.log  # print fields 1 and 3 (whitespace-delimited)
awk -F',' '{print $2}' data.csv  # print column 2 from CSV
awk '$3 > 1000' data.txt         # print rows where column 3 > 1000
awk '/ERROR/ {count++} END {print count}' app.log   # count ERROR lines

# sed: stream editor — find and replace in text
sed 's/old/new/' file.txt        # replace first occurrence per line
sed 's/old/new/g' file.txt       # replace all occurrences (g = global)
sed -i 's/localhost/0.0.0.0/g' config.conf   # edit file in place (-i)
sed -n '10,20p' file.txt         # print only lines 10 through 20
sed '/^#/d' config.conf          # delete lines starting with #
```

### Level 4: Permissions and Process Management

```bash
# --- Permissions ---
# ls -l shows permissions like: -rwxr-xr-x
#
# Position 1: file type (- = file, d = directory, l = symlink)
# Positions 2-4: owner permissions (r=read, w=write, x=execute)
# Positions 5-7: group permissions
# Positions 8-10: others (everyone else) permissions

# chmod: change permissions
chmod +x deploy.sh               # add execute permission for everyone
chmod u+x deploy.sh              # add execute for owner (u) only
chmod g-w shared_file.txt        # remove write from group (g)
chmod o= private.txt             # set others to no permissions at all
chmod 755 deploy.sh              # numeric: rwxr-xr-x (owner full, group/others read+exec)
chmod 644 config.txt             # numeric: rw-r--r-- (owner read/write, others read)
chmod 600 private.key            # numeric: rw------- (owner read/write only)
chmod -R 755 ./public/           # recursive — change entire directory tree

# Numeric permission reference:
# 4 = read, 2 = write, 1 = execute. Add them for each group.
# 7 = 4+2+1 = rwx
# 5 = 4+0+1 = r-x
# 6 = 4+2+0 = rw-

# chown: change ownership
chown alice report.txt                   # change owner to alice
chown alice:developers report.txt        # change owner and group
chown -R www-data:www-data /var/www/     # recursive ownership change for web server

# --- Process Management ---
# ps: snapshot of running processes
ps aux                           # all processes, all users (a=all, u=user format, x=no tty)
ps aux | grep python             # find python processes
ps -ef --forest                  # show process tree (parent/child relationships)

# top / htop: live process viewer
top                              # built-in, press q to quit, k to kill, r to renice
htop                             # better version (may need: sudo apt install htop)

# kill processes
kill 1234                        # send SIGTERM (15) — polite request to stop
kill -9 1234                     # send SIGKILL — immediate force stop (cannot be caught)
killall python3                  # kill all processes named python3
pkill -f "gunicorn"              # kill by matching full command line

# Background jobs
python server.py &               # start in background; shell prints the PID
jobs                             # list background jobs for current shell session
fg %1                            # bring job 1 to foreground
bg %1                            # resume stopped job in background
Ctrl+Z                           # suspend the foreground process (send to background stopped)
nohup python server.py &         # run in background, immune to hangup (survives SSH disconnect)
```

### Level 5: Networking, SSH, and System Diagnostics

```bash
# --- Network diagnostics ---
ip addr show                     # show all network interfaces and IP addresses
ip addr show eth0                # show specific interface
ping -c 4 google.com             # test connectivity, 4 packets
traceroute google.com            # trace the network path to destination
ss -tuln                         # show listening TCP/UDP ports (no dns lookup)
ss -tulnp                        # same, including the process name
lsof -i :8080                    # what process is listening on port 8080?
curl -I https://example.com      # HTTP HEAD request — check headers/status
curl -s https://api.example.com/health | python3 -m json.tool  # pretty-print JSON
wget -q -O - https://example.com/data.csv | head   # download and pipe

# --- SSH ---
ssh user@192.168.1.10                       # connect to server
ssh -p 2222 user@host                       # non-standard port
ssh -i ~/.ssh/deploy_key user@host          # specify private key
ssh -L 8080:localhost:80 user@server        # local port forwarding
# (access server's port 80 via localhost:8080)

# Copy files securely
scp report.pdf user@server:/home/user/      # copy file to remote
scp -r ./project/ user@server:~/            # copy directory recursively
scp user@server:~/backup.tar.gz ./          # download from remote

# rsync: efficient sync (only transfers what changed)
rsync -avz ./src/ user@server:~/src/        # sync local to remote
rsync -avz --delete ./src/ user@server:~/src/   # sync and delete remote files not in source

# --- System health ---
df -h                            # disk space usage, human-readable
du -sh /var/log/                 # disk usage of a specific directory
du -h --max-depth=1 /var/ | sort -hr   # find biggest subdirectories
free -h                          # RAM usage
uptime                           # system uptime and load average
cat /proc/loadavg                # raw load average (1, 5, 15 minute)
vmstat 1 5                       # virtual memory stats, 1-second intervals, 5 times
iostat -x 1 3                    # disk I/O stats (needs sysstat package)
```

### Level 6: Shell Features That Save Hours

```bash
# --- Keyboard shortcuts (non-negotiable muscle memory) ---
# Ctrl+C    — kill current process
# Ctrl+Z    — suspend current process (then use fg/bg)
# Ctrl+D    — send EOF / exit shell
# Ctrl+L    — clear screen (same as `clear`)
# Ctrl+R    — reverse history search (type to search past commands)
# Ctrl+A    — jump to beginning of line
# Ctrl+E    — jump to end of line
# Ctrl+U    — delete everything before cursor
# Ctrl+K    — delete everything after cursor
# Ctrl+W    — delete one word backwards
# Tab       — autocomplete (double-Tab to see options)
# !! runs last command, !ssh runs last command starting with ssh

# --- Variables and Environment ---
NAME="alice"
echo "Hello, $NAME"             # Hello, alice
echo "Hello, ${NAME}!"          # Hello, alice! (braces prevent ambiguity)

export DATABASE_URL="postgresql://localhost/myapp"   # export makes it available to child processes
env | grep DATABASE              # confirm it's set
unset DATABASE_URL               # remove it

# --- Aliases: shortcuts for long commands ---
alias ll='ls -lah'
alias gs='git status'
alias ..='cd ..'
alias grep='grep --color=auto'
# Put these in ~/.bashrc to make them permanent

# --- Brace expansion: powerful shorthand ---
mkdir -p project/{src,tests,docs,scripts}    # create 4 directories at once
cp config.yml{,.backup}                      # copy to config.yml.backup (expands to: cp config.yml config.yml.backup)
echo file{1..5}.txt                          # file1.txt file2.txt file3.txt file4.txt file5.txt

# --- Practical one-liners for real situations ---

# Find the 10 largest files in /var
find /var -type f -printf '%s %p\n' | sort -rn | head -10

# Kill all processes matching a pattern (careful)
ps aux | grep '[g]unicorn' | awk '{print $2}' | xargs kill

# Watch a command every 2 seconds (like a live dashboard)
watch -n 2 'df -h && echo --- && free -h'

# Count unique IP addresses in an access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Recursively find all TODO comments in Python files
grep -rn "TODO" --include="*.py" ./src/

# Replace a string in all Python files in a directory
find . -name "*.py" -exec sed -i 's/old_function/new_function/g' {} \;

# Show disk usage sorted, excluding mount points
du -h --max-depth=1 / --exclude=/proc --exclude=/sys 2>/dev/null | sort -hr | head -15
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: `rm -rf` without understanding what you're deleting**

```bash
# WRONG (can destroy your entire home directory):
cd /home/alice/projects
rm -rf *              # intended to delete files IN projects

# If you accidentally ran: rm -rf / (or rm -rf ~)...
# There is no recycle bin. The files are gone.

# CORRECT practice: always verify first
ls                    # look at what's there
rm -ri *              # -i asks for confirmation per file
# Or move to a temp location first:
mv ~/projects/old_stuff /tmp/   # can delete later
```

**Mistake 2: Misunderstanding `>` vs `>>`**

```bash
# > OVERWRITES the file completely — previous contents are lost
echo "first line" > output.txt
echo "second line" > output.txt   # first line is GONE

# >> APPENDS — adds to existing content
echo "first line" >> output.txt
echo "second line" >> output.txt  # both lines present

# Common disaster: accidentally overwriting a config file
cat > /etc/nginx/nginx.conf       # starts overwriting immediately when you press Enter
```

**Mistake 3: `chmod 777` as a "fix everything" solution**

```bash
# WRONG: giving world write permission is a security vulnerability
chmod 777 /var/www/html/uploads/   # any user on the system can modify these files

# Why this is wrong:
# - On a shared server, other users can overwrite your web files
# - PHP/CGI files become executable by everyone — code injection risk
# - If your web server is compromised, attackers can write anywhere

# CORRECT: give only the permissions actually needed
chown -R www-data:www-data /var/www/html/uploads/   # web server owns the files
chmod -R 750 /var/www/html/         # owner full, group read+exec, others nothing
chmod -R 770 /var/www/html/uploads/ # uploads need write for www-data
```

**Mistake 4: Not understanding that pipes work on stdout, not stderr**

```bash
# WRONG: expecting errors to be filtered by grep
python app.py | grep "ImportError"   # errors go to stderr, not caught by pipe

# The error output goes directly to your terminal, bypassing the pipe

# CORRECT: redirect stderr into stdout first, then pipe
python app.py 2>&1 | grep "ImportError"

# Or save both to a file
python app.py > output.log 2>&1
```

**Mistake 5: Forgetting that `find -exec` is slow for many files**

```bash
# SLOW: spawns a new process for EACH file found
find . -name "*.log" -exec rm {} \;

# FAST: xargs batches files into fewer rm invocations
find . -name "*.log" | xargs rm

# SAFEST: xargs with null terminator (handles filenames with spaces)
find . -name "*.log" -print0 | xargs -0 rm
```

**Mistake 6: `kill -9` as the first choice**

```bash
# WRONG: jumping straight to SIGKILL
kill -9 1234    # force kill — no cleanup possible

# Why this matters: the process can't flush buffers, close DB connections,
# write shutdown logs, or clean up temp files.

# CORRECT: try graceful first, force only if needed
kill 1234           # sends SIGTERM — process has a chance to clean up
sleep 5
kill -0 1234 2>/dev/null && kill -9 1234   # only force-kill if still running
```

---

## 5. The "Why Does This Work" Layer

### Why Pipes Are Powerful: Standard Streams

Every process in Linux has three open file descriptors by default:

```
File Descriptor 0: stdin  — standard input  (keyboard by default)
File Descriptor 1: stdout — standard output (terminal by default)
File Descriptor 2: stderr — standard error  (terminal by default)
```

When you write `command1 | command2`, the shell connects `command1`'s stdout (fd 1) to `command2`'s stdin (fd 0) using an in-kernel pipe buffer. Both processes run concurrently — `command1` writes, `command2` reads as fast as data arrives. No temporary file is created. This is why long pipelines are memory-efficient even with huge inputs.

`2>&1` means "redirect fd 2 (stderr) to wherever fd 1 (stdout) currently points." The order matters: `command > file 2>&1` is correct (redirect stdout to file, then redirect stderr to stdout which now points to file). `command 2>&1 > file` is wrong (redirects stderr to the terminal, then redirects stdout to file).

### How the Shell Finds Commands

When you type `nginx`, the shell doesn't search the entire filesystem. It searches only the directories listed in `$PATH`, in order:

```
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin

# Shell checks:
# /usr/local/bin/nginx — not found
# /usr/bin/nginx       — FOUND — execute this
```

This is why `export PATH="$PATH:/usr/local/go/bin"` is needed after installing Go — you're appending Go's binary directory to the search list. If you run `./script.sh` (with the `./` prefix), you're bypassing PATH entirely and specifying an exact file location.

### File Permissions Under the Hood

Permissions are stored as a 12-bit integer in the file's inode, not in the filename or directory. The `chmod 755` notation encodes three octal digits (3 bits each):

```
7   5   5
│   │   └── others: 4+0+1 = r-x
│   └────── group:  4+0+1 = r-x
└────────── owner:  4+2+1 = rwx

Binary representation of each digit:
7 = 111 = rwx
5 = 101 = r-x
4 = 100 = r--
6 = 110 = rw-
0 = 000 = ---
```

When you run a file, the kernel checks: is the calling process's effective UID the file owner? Check owner bits. Is the calling process's effective GID the file's group? Check group bits. Otherwise, check others bits. If no matching execute bit is set, `execve()` returns `EACCES`.

---

## 6. Quick Reference

### Navigation

| Command | Action |
|---------|--------|
| `pwd` | Print current directory |
| `cd /path` | Change to absolute path |
| `cd ..` | Go up one level |
| `cd -` | Go to previous directory |
| `ls -lah` | List with details, including hidden |

### Files

| Command | Action |
|---------|--------|
| `cp -r src/ dst/` | Copy directory recursively |
| `mv old new` | Move or rename |
| `rm -ri target` | Delete with confirmation |
| `find . -name "*.py"` | Find files by pattern |
| `grep -rn "TODO" ./` | Search file contents recursively |

### Permissions

| Pattern | Numeric | Meaning |
|---------|---------|---------|
| `rwxr-xr-x` | 755 | Scripts, executables |
| `rw-r--r--` | 644 | Config files, documents |
| `rw-------` | 600 | Private keys, secrets |
| `rwxrwx---` | 770 | Shared group directories |

### Processes

| Command | Action |
|---------|--------|
| `ps aux \| grep name` | Find process by name |
| `kill PID` | Graceful stop |
| `kill -9 PID` | Force stop |
| `lsof -i :8080` | What's on this port? |
| `nohup cmd &` | Run detached from terminal |

### Redirection

| Operator | Meaning |
|----------|---------|
| `> file` | Redirect stdout (overwrite) |
| `>> file` | Redirect stdout (append) |
| `2> file` | Redirect stderr |
| `2>&1` | Merge stderr into stdout |
| `cmd1 \| cmd2` | Pipe stdout to stdin |
| `> /dev/null 2>&1` | Silence everything |

### Essential Keyboard Shortcuts

| Key | Effect |
|-----|--------|
| `Ctrl+R` | Search command history |
| `Ctrl+C` | Kill current process |
| `Ctrl+Z` | Suspend (then `fg`/`bg`) |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` / `Ctrl+E` | Jump to line start / end |
| `Ctrl+U` | Delete to start of line |
| `Tab` | Autocomplete |
