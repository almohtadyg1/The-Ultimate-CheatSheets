# Linux CLI Cheat Sheet

Complete guide to the Linux command line — from basics to advanced operations.

> **Tip:** Press `Tab` for auto-completion and `Ctrl+R` to search command history!

---

## File & Directory Operations

### Basic File Commands

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `ls` | List files and directories | `-l` long format, `-a` show hidden, `-h` human-readable | `ls -lah` |
| `cd` | Change directory | `-` previous dir, `~` home dir | `cd /var/log` |
| `pwd` | Print working directory | — | `pwd` |
| `mkdir` | Create directory | `-p` create parent dirs | `mkdir -p dir/subdir` |
| `touch` | Create empty file or update timestamp | — | `touch newfile.txt` |

### File Manipulation

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `cp` | Copy files/directories | `-r` recursive, `-i` interactive | `cp -r dir1 dir2` |
| `mv` | Move or rename files | `-i` interactive, `-v` verbose | `mv old.txt new.txt` |
| `rm` | Remove files/directories | `-r` recursive, `-f` force, `-i` interactive | `rm -rf directory/` |
| `ln` | Create links | `-s` symbolic link | `ln -s target link` |

### Viewing Files

```bash
# View entire file
cat file.txt

# View with pagination
less file.txt
more file.txt

# View first/last lines
head -n 20 file.txt   # First 20 lines
tail -n 20 file.txt   # Last 20 lines
tail -f log.txt       # Follow log file in real time
```

### Finding Files

```bash
# Find by name
find /path -name "*.txt"

# Find by type
find /path -type f   # Files only
find /path -type d   # Directories only

# Find by size
find /path -size +100M   # Files larger than 100MB

# Find and execute command on results
find /path -name "*.log" -exec rm {} \;
```

---

## File Permissions

> **Permission format:** `rwxrwxrwx` = Read, Write, Execute for Owner, Group, Others

### Permission Commands

| Command | Description | Example |
|---------|-------------|---------|
| `chmod` | Change file permissions | `chmod 755 file.sh` |
| `chown` | Change file owner | `chown user:group file.txt` |
| `chgrp` | Change group ownership | `chgrp developers file.txt` |
| `umask` | Set default permissions | `umask 022` |

### Common Permission Patterns

```bash
# Symbolic notation
chmod u+x script.sh     # Add execute for user
chmod g-w file.txt      # Remove write for group
chmod o=r file.txt      # Set others to read-only

# Numeric notation
chmod 755 script.sh     # rwxr-xr-x
chmod 644 file.txt      # rw-r--r--
chmod 600 private.key   # rw-------

# Recursive changes
chmod -R 755 directory/
chown -R user:group directory/
```

---

## Process Management

### Process Monitoring

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `ps` | Display processes | `aux` all processes, `-ef` full format | `ps aux \| grep apache` |
| `top` | Real-time process viewer | `-u user` filter by user | `top` |
| `htop` | Interactive process viewer | — | `htop` |
| `pgrep` | Find processes by name | `-u user` by user | `pgrep -u www-data` |

### Process Control

```bash
# Kill processes
kill PID              # Terminate gracefully (SIGTERM)
kill -9 PID           # Force kill (SIGKILL)
killall process_name  # Kill by name
pkill pattern         # Kill by pattern

# Background processes
command &             # Run in background
jobs                  # List background jobs
fg %1                 # Bring job 1 to foreground
bg %1                 # Resume job 1 in background

# Process priority
nice -n 10 command    # Start with lower priority
renice -5 PID         # Change priority of running process
```

### Search with grep

```bash
# Basic search
grep "pattern" file.txt

# Case insensitive
grep -i "error" log.txt

# Recursive search in directory
grep -r "pattern" /path/

# Show line numbers
grep -n "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Invert match (lines that do NOT match)
grep -v "error" log.txt

# Multiple patterns
grep -e "pattern1" -e "pattern2" file.txt
```

---

## Networking

### Network Commands

| Command | Description | Example |
|---------|-------------|---------|
| `ifconfig` | Display network interfaces | `ifconfig eth0` |
| `ip` | Modern network configuration | `ip addr show` |
| `ping` | Test network connectivity | `ping -c 4 google.com` |
| `netstat` | Network statistics | `netstat -tuln` |
| `ss` | Socket statistics (modern netstat) | `ss -tuln` |

### Network Tools

```bash
# SSH — Secure Shell
ssh user@hostname
ssh -p 2222 user@host        # Custom port
ssh -i key.pem user@host     # With private key

# SCP — Secure Copy
scp file.txt user@host:/path/
scp -r directory/ user@host:/path/

# Download files
wget http://example.com/file.zip
curl -O http://example.com/file.zip

# Network diagnostics
traceroute google.com
nslookup google.com
dig google.com
```

---

## System Information

### System Monitoring

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `uname` | System information | `-a` all info, `-r` kernel version | `uname -a` |
| `df` | Disk space usage | `-h` human-readable | `df -h` |
| `du` | Directory size | `-sh` summary, `-h` readable | `du -sh /var/log` |
| `free` | Memory usage | `-h` human-readable | `free -h` |
| `uptime` | System uptime and load | — | `uptime` |
| `whoami` | Current username | — | `whoami` |
| `who` | Logged in users | — | `who` |

### Hardware Information

```bash
# CPU information
lscpu
cat /proc/cpuinfo

# Memory information
cat /proc/meminfo
free -h

# PCI devices
lspci
lspci -v    # Verbose

# USB devices
lsusb
lsusb -v    # Verbose

# Block devices / disks
lsblk
blkid
```

---

## User Management

### User Commands

| Command | Description | Example |
|---------|-------------|---------|
| `useradd` | Create new user (low-level) | `sudo useradd username` |
| `adduser` | Create user interactively | `sudo adduser username` |
| `userdel` | Delete user | `sudo userdel -r username` |
| `passwd` | Change password | `passwd username` |
| `usermod` | Modify user account | `sudo usermod -aG group user` |
| `su` | Switch user | `su - username` |
| `sudo` | Execute as superuser | `sudo command` |

### User Information

```bash
# View user info
id username
finger username

# Login history
last
last username

# Currently logged in
who
w

# Lock / unlock user account
sudo passwd -l username   # Lock
sudo passwd -u username   # Unlock
```

---

## File Compression & Archiving

### Archive Commands

| Command | Description | Example |
|---------|-------------|---------|
| `tar` | Create/extract tar archives | `tar -czvf archive.tar.gz dir/` |
| `gzip` | Compress files | `gzip file.txt` |
| `gunzip` | Decompress gzip files | `gunzip file.txt.gz` |
| `zip` | Create zip archives | `zip -r archive.zip dir/` |
| `unzip` | Extract zip archives | `unzip archive.zip` |

### Common tar Operations

```bash
# Create compressed archive
tar -czvf archive.tar.gz  directory/   # gzip
tar -cjvf archive.tar.bz2 directory/   # bzip2

# Extract archive
tar -xzvf archive.tar.gz
tar -xjvf archive.tar.bz2

# List archive contents without extracting
tar -tzvf archive.tar.gz

# Extract to a specific directory
tar -xzvf archive.tar.gz -C /path/to/directory
```

---

## IO Redirection & Pipes

### Redirection Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `>` | Redirect stdout (overwrite) | `command > file.txt` |
| `>>` | Redirect stdout (append) | `command >> file.txt` |
| `<` | Redirect stdin | `command < input.txt` |
| `2>` | Redirect stderr | `command 2> error.log` |
| `&>` | Redirect both stdout and stderr | `command &> output.log` |
| `2>&1` | Redirect stderr to stdout | `command > file 2>&1` |
| `\|` | Pipe output to another command | `ls -l \| grep "txt"` |

### Practical Examples

```bash
# Save output to file
ls -la > listing.txt

# Append to file
echo "New line" >> file.txt

# Discard all output (silence a command)
command > /dev/null 2>&1

# Chain multiple commands with pipes
cat file.txt | grep "error" | wc -l

# Tee: write to file AND display on terminal
command | tee output.txt
```

---

## Environment Variables

### Variable Commands

| Command | Description | Example |
|---------|-------------|---------|
| `export` | Set environment variable | `export VAR=value` |
| `echo` | Display variable value | `echo $VAR` |
| `env` | List all environment variables | `env` |
| `unset` | Remove variable | `unset VAR` |
| `printenv` | Print specific variable | `printenv PATH` |

### Common Variables & Usage

```bash
# View important variables
echo $PATH     # Executable search paths
echo $HOME     # Home directory
echo $USER     # Current username
echo $SHELL    # Current shell

# Add permanently (append to ~/.bashrc or ~/.profile)
export PATH="$PATH:/new/path"
export MY_VAR="value"
```

---

## Quick Reference — Useful One-Liners

```bash
# Find large files (>100MB)
find / -type f -size +100M

# Disk usage sorted by size
du -h --max-depth=1 /path | sort -hr

# Follow a log file in real time
tail -f /var/log/syslog

# Count files in current directory
ls -1 | wc -l

# Find and replace text inside a file
sed -i 's/old/new/g' file.txt

# Show directory tree (2 levels deep)
tree -L 2 /path

# List open ports
ss -tuln

# Check which process is using a port
lsof -i :8080

# Show last 10 commands run
history | tail -n 10
```

---

## Keyboard Shortcuts

| Shortcut | Description |
|----------|-------------|
| `Ctrl + C` | Kill current process |
| `Ctrl + Z` | Suspend current process |
| `Ctrl + D` | Exit current shell / EOF |
| `Ctrl + L` | Clear screen |
| `Ctrl + R` | Search command history (reverse) |
| `Ctrl + A` | Move cursor to beginning of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + U` | Delete from cursor to start of line |
| `Ctrl + K` | Delete from cursor to end of line |
| `Ctrl + W` | Delete previous word |
| `Tab` | Auto-complete command or path |
| `↑ / ↓` | Navigate command history |

> **Pro tip:** Master these shortcuts and you'll be flying through the terminal. 🚀
