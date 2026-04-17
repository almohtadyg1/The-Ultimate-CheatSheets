# Bash Scripting: A Complete Progressive Tutorial

---

## 1. What & Why

Bash is both an interactive command interpreter and a scripting language that ships on virtually every Unix-like system. You use it every time you open a terminal. But its real power emerges when you start writing scripts: files of Bash commands that automate tasks you'd otherwise do manually.

Why script in Bash rather than Python or Ruby? Because Bash is *already there*. On any Linux server, in any Docker container, in any CI/CD runner — Bash is available without installation. It integrates directly with the operating system's process model, making it the natural choice for tasks that involve running other programs, piping data between them, managing files, and coordinating system operations.

Where Bash excels: deployment scripts, log processing, file organization, system administration, CI/CD pipelines, scheduled jobs, bootstrapping environments. Where it does not: complex data structures, floating-point math, anything requiring libraries or network I/O beyond `curl`. For those, reach for Python.

This tutorial teaches you to write Bash scripts that are correct, readable, and robust — not just "works on my machine" shell one-liners.

---

## 2. Mental Model

Think of Bash as a language that orchestrates other programs. Unlike Python or Java, where you write logic directly, Bash primarily runs external tools and connects them.

```
Your script
│
├── Variables: store state between commands
├── Control flow: decide which commands to run
├── Functions: name and reuse command sequences
└── Plumbing: connect commands with pipes, redirections
         │
         ├── grep, awk, sed: text processing
         ├── find, ls, cp, mv: filesystem
         ├── curl, ssh: networking
         └── python, node, psql: anything else
```

Every command produces output (stdout), errors (stderr), and an exit code (0 = success, anything else = failure). Bash's entire flow-control system — if, while, &&, || — is built around exit codes. The output of one command becomes the input of another through pipes. This composition model is Bash's core philosophy.

```bash
# The mental model in action:
find . -name "*.log"       \   # find: produces filenames
  | xargs grep "ERROR"     \   # grep: filters by content
  | awk '{print $1}'       \   # awk: extracts timestamps
  | sort                   \   # sort: orders them
  | uniq -c                \   # uniq: counts duplicates
  | sort -rn               \   # sort: most frequent first
  | head -5                    # head: top 5 results
# No single tool does all of this. Bash composes them.
```

---

## 3. Progressive Examples

### Level 1: Script Structure and Variables

```bash
#!/usr/bin/env bash
# The shebang line: tells the OS what interpreter to use.
# /usr/bin/env bash finds bash in PATH — more portable than /bin/bash.
# Make executable with: chmod +x script.sh
# Run with: ./script.sh  OR  bash script.sh

# Variables — NO spaces around the = sign (this is non-negotiable)
name="Alice"           # correct
# name = "Alice"       # WRONG: Bash treats this as running a command called 'name'

age=30
message="Hello from Bash"

# Reading variables: prefix with $
echo $name             # Alice
echo "$name"           # Alice (quoted — always prefer this)
echo "${name}!"        # Alice! — braces separate variable name from suffix

# Commands and their output
current_date=$(date +%Y-%m-%d)   # $() = command substitution
echo "Today is $current_date"

hostname=$(hostname)
echo "Running on: $hostname"

# Arithmetic — only integers, use $(( ))
x=10
y=3
echo $((x + y))        # 13
echo $((x * y))        # 30
echo $((x / y))        # 3 (integer division — truncates)
echo $((x % y))        # 1 (remainder)
echo $((x ** 2))       # 100 (exponentiation)

# Increment
count=0
((count++))            # increment in-place
echo $count            # 1

# Floating point requires bc or awk
echo "scale=4; 22/7" | bc      # 3.1428 — bc does arbitrary precision
awk 'BEGIN { printf "%.6f\n", 22/7 }'  # 3.142857

# Special variables you'll use constantly
echo "Script name: $0"     # name of this script
echo "First arg:  $1"      # first argument passed to script
echo "Arg count:  $#"      # number of arguments
echo "All args:   $@"      # all arguments (safe, quote-preserving)
echo "Last status: $?"     # exit code of previous command
echo "This PID:   $$"      # PID of current shell process
```

### Level 2: Conditionals, Tests, and Quoting

Understanding tests and quoting correctly is what separates working scripts from scripts that break on edge cases.

```bash
#!/usr/bin/env bash

# The [[ ]] construct — always prefer this over [ ]
# [[ ]] is a Bash built-in, handles quoting more safely, supports regex

age=25

if [[ $age -ge 18 ]]; then
    echo "Adult"
elif [[ $age -ge 13 ]]; then
    echo "Teen"
else
    echo "Child"
fi

# Numeric comparison operators in [[ ]]:
# -eq  equal
# -ne  not equal
# -lt  less than
# -gt  greater than
# -le  less than or equal
# -ge  greater than or equal

# String comparison operators:
name="Alice"
if [[ "$name" == "Alice" ]]; then echo "Hi Alice"; fi
if [[ "$name" != "Bob" ]]; then echo "Not Bob"; fi
if [[ -z "$name" ]]; then echo "empty"; fi      # -z: string is empty
if [[ -n "$name" ]]; then echo "not empty"; fi  # -n: string has content

# File tests:
if [[ -f "/etc/passwd" ]]; then echo "file exists"; fi
if [[ -d "/home" ]]; then echo "is directory"; fi
if [[ -e "/tmp" ]]; then echo "path exists (file or dir)"; fi
if [[ -r "/etc/passwd" ]]; then echo "readable"; fi
if [[ -x "/usr/bin/bash" ]]; then echo "executable"; fi
if [[ -s "file.txt" ]]; then echo "file exists and non-empty"; fi
if [[ -L "/usr/bin/python" ]]; then echo "is symlink"; fi

# Logical operators inside [[ ]]:
if [[ -f "config.yml" && -r "config.yml" ]]; then
    echo "config exists and is readable"
fi

if [[ "$env" == "prod" || "$env" == "staging" ]]; then
    echo "remote environment"
fi

# Regex match with =~
email="user@example.com"
if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi
# Capture groups go into BASH_REMATCH array:
if [[ "2024-01-15" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
    echo "Year:  ${BASH_REMATCH[1]}"   # 2024
    echo "Month: ${BASH_REMATCH[2]}"   # 01
    echo "Day:   ${BASH_REMATCH[3]}"   # 15
fi

# case statement — cleaner than long if/elif chains for pattern matching
extension="${filename##*.}"   # extract extension (more on this below)
case "$extension" in
    txt|md|rst)
        echo "Text document"
        ;;
    jpg|jpeg|png|gif|webp)
        echo "Image file"
        ;;
    sh|bash)
        echo "Shell script"
        ;;
    py)
        echo "Python script"
        ;;
    *)
        echo "Unknown type: $extension"
        ;;
esac

# --- Quoting: the most critical Bash skill ---

filename="my report 2024.csv"   # filename with spaces

# WRONG: without quotes, Bash splits on whitespace
# wc -l $filename   -> treated as: wc -l my report 2024.csv (4 args!)

# CORRECT: always quote variables that may contain spaces or special chars
wc -l "$filename"              # treated as: wc -l "my report 2024.csv"

# Default value patterns:
port="${PORT:-8080}"           # use $PORT if set, otherwise 8080
outdir="${1:?Must provide output dir}"  # error and exit if $1 is unset
```

### Level 3: Loops and Arrays

```bash
#!/usr/bin/env bash

# --- Loops ---

# for in loop — most common
for fruit in apple banana cherry; do
    echo "Fruit: $fruit"
done

# Brace expansion — generate ranges
for i in {1..5}; do
    echo "Count: $i"
done

for i in {0..20..5}; do     # 0, 5, 10, 15, 20
    echo "$i"
done

# C-style for loop
for ((i = 0; i < 10; i++)); do
    printf "Item %02d\n" $i
done

# Iterate over files — ALWAYS quote the glob result
for f in /var/log/*.log; do
    [[ -f "$f" ]] || continue   # skip if glob matched nothing
    echo "Processing: $f"
    wc -l "$f"
done

# while loop
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line — the safe pattern
while IFS= read -r line; do
    # IFS= prevents trimming leading/trailing whitespace
    # -r prevents backslash interpretation
    echo "Line: $line"
done < /etc/hosts

# Read from command output
while IFS= read -r filename; do
    echo "Found: $filename"
done < <(find . -name "*.sh" -type f)
# The < <() syntax is process substitution — avoids a subshell (variables persist)

# Loop control
for i in {1..20}; do
    [[ $i -eq 5 ]] && continue    # skip 5
    [[ $i -eq 15 ]] && break      # stop at 15
    echo $i
done

# --- Arrays ---

# Indexed array
servers=("web01" "web02" "web03" "db01")
servers+=("cache01")         # append

echo "${servers[0]}"         # web01 — first element
echo "${servers[-1]}"        # cache01 — last element (Bash 4.2+)
echo "${servers[@]}"         # all elements
echo "${#servers[@]}"        # 5 — array length
echo "${!servers[@]}"        # 0 1 2 3 4 — all indices

# Iterate over array
for server in "${servers[@]}"; do
    echo "Checking $server..."
    ping -c 1 -q "$server" && echo "  UP" || echo "  DOWN"
done

# Iterate with index
for i in "${!servers[@]}"; do
    printf "%d: %s\n" "$i" "${servers[$i]}"
done

# Slicing
echo "${servers[@]:1:2}"     # web02 web03 (from index 1, length 2)

# Read command output into array — safe pattern
mapfile -t log_files < <(find /var/log -name "*.log" -type f)
echo "Found ${#log_files[@]} log files"

# Associative array (dictionary) — requires declare -A
declare -A config
config[host]="localhost"
config[port]="5432"
config[database]="myapp"
config["max_connections"]="100"

echo "${config[host]}"        # localhost
for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done
```

### Level 4: Functions and Error Handling

```bash
#!/usr/bin/env bash
set -euo pipefail   # fail fast — always put this in production scripts
# -e  exit on any command failure
# -u  error on undefined variable
# -o pipefail  pipeline fails if any command in it fails

# Functions
greet() {
    local name="$1"             # local: scoped to this function only
    local greeting="${2:-Hello}"  # optional second arg with default
    echo "$greeting, $name!"
}

greet "Alice"           # Hello, Alice!
greet "Bob" "Hi"        # Hi, Bob!

# Functions return values via stdout, not return statements
# return only sets an exit code (0-255)
get_timestamp() {
    date +%Y%m%d_%H%M%S   # printed to stdout
}
stamp=$(get_timestamp)    # captured with $()
echo "Timestamp: $stamp"

# Exit codes as return values
is_port_open() {
    local host="$1"
    local port="$2"
    timeout 2 bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null
    # returns 0 if connection succeeded, non-zero otherwise
}

if is_port_open "localhost" "5432"; then
    echo "PostgreSQL is up"
else
    echo "PostgreSQL is not reachable"
fi

# Error handling with trap
cleanup() {
    local exit_code=$?
    echo "Cleaning up..."
    rm -rf "$WORK_DIR"
    exit $exit_code   # preserve original exit code
}

WORK_DIR=$(mktemp -d)
trap cleanup EXIT   # runs cleanup on ANY exit (success, error, Ctrl+C)

# Without trap, if the script dies mid-run, $WORK_DIR leaks.
# With trap, it's always cleaned up.

# die function — standard pattern for fatal errors
die() {
    echo "ERROR: $*" >&2   # >&2 sends to stderr, not stdout
    exit 1
}

require_command() {
    command -v "$1" >/dev/null 2>&1 || die "Required command not found: $1"
}

require_command curl
require_command jq
require_command aws

# Conditional execution with && and ||
mkdir -p /tmp/deploy || die "Failed to create deploy directory"
cd /tmp/deploy && echo "Changed to deploy directory"

# Check exit codes explicitly when set -e is too broad
if ! git pull origin main; then
    warn "git pull failed — using local version"
fi
```

### Level 5: String Manipulation and Text Processing

```bash
#!/usr/bin/env bash

# --- Parameter Expansion: built-in string operations ---
# These run in the shell without spawning any subprocess. Prefer them
# over calling sed/awk for simple string operations.

path="/home/alice/projects/app/main.py"

# Length
echo "${#path}"           # 37

# Substring (offset, length)
echo "${path:6}"          # alice/projects/app/main.py (from index 6)
echo "${path:6:5}"        # alice

# Remove prefix — # removes shortest, ## removes longest
echo "${path#*/}"         # home/alice/projects/app/main.py
echo "${path##*/}"        # main.py (basename)
echo "${path##*.}"        # py (file extension)

# Remove suffix — % removes shortest, %% removes longest
echo "${path%.py}"        # /home/alice/projects/app/main (strip extension)
echo "${path%/*}"         # /home/alice/projects/app (dirname)

# Substitution
echo "${path/alice/bob}"  # /home/bob/projects/app/main.py (first only)
echo "${path//\//|}"      # replace ALL slashes with pipes

# Case modification (Bash 4+)
name="hello world"
echo "${name^}"           # Hello world (capitalize first char)
echo "${name^^}"          # HELLO WORLD (all uppercase)
name="HELLO WORLD"
echo "${name,,}"          # hello world (all lowercase)

# --- Text processing pipelines ---

# Process a CSV file: find users with score > 80, print formatted
process_scores() {
    local file="$1"
    awk -F',' '
        NR > 1 && $3 > 80 {
            printf "%-20s %-20s %3d\n", $1, $2, $3
        }
    ' "$file" | sort -k3 -rn
}

# Count occurrences of each HTTP status code in a log
analyze_access_log() {
    local logfile="$1"
    echo "HTTP Status Code Distribution:"
    awk '{print $9}' "$logfile" |   # field 9 = status code in nginx/apache logs
        sort |
        uniq -c |
        sort -rn |
        awk '{printf "  %5d  %s\n", $1, $2}'
}

# Parse INI-style config file
parse_config() {
    local file="$1"
    local key="$2"
    grep "^${key}\s*=" "$file" |
        sed "s/^${key}\s*=\s*//" |
        tr -d '"' |
        head -1
}

db_host=$(parse_config /etc/app/config.ini "database_host")

# Split a string on a delimiter — safe pattern
IFS=',' read -ra fields <<< "alice,30,engineer,cairo"
echo "Name: ${fields[0]}"    # alice
echo "Age:  ${fields[1]}"    # 30
echo "Role: ${fields[2]}"    # engineer
echo "City: ${fields[3]}"    # cairo
```

### Level 6: Production Script Template

```bash
#!/usr/bin/env bash
# deploy.sh — example production deployment script
# Demonstrates all the patterns that matter in real scripts

set -euo pipefail
IFS=$'\n\t'   # split only on newlines and tabs (not spaces)

# Script location — resolve even through symlinks
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/deploy.log"

# Colors — disable if not a TTY (pipes, cron, etc.)
if [[ -t 1 ]]; then
    RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'; BLUE='\033[0;34m'; NC='\033[0m'
else
    RED=''; GREEN=''; YELLOW=''; BLUE=''; NC=''
fi

# Logging functions — always write to log, optionally to screen
log()   { echo -e "${GREEN}[INFO]${NC}  $(date '+%F %T') $*" | tee -a "$LOG_FILE"; }
warn()  { echo -e "${YELLOW}[WARN]${NC}  $(date '+%F %T') $*" | tee -a "$LOG_FILE" >&2; }
error() { echo -e "${RED}[ERROR]${NC} $(date '+%F %T') $*" | tee -a "$LOG_FILE" >&2; }
die()   { error "$*"; exit 1; }

# Usage documentation
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <environment>

Deploy the application to the specified environment.

Arguments:
  environment   Target environment: dev, staging, prod

Options:
  -b, --branch BRANCH   Git branch to deploy (default: main)
  -t, --tag TAG         Git tag to deploy (overrides branch)
  -n, --dry-run         Show what would be done without doing it
  -v, --verbose         Enable verbose output
  -h, --help            Show this help message

Examples:
  $SCRIPT_NAME staging
  $SCRIPT_NAME prod --branch release/v2.1
  $SCRIPT_NAME dev --dry-run
EOF
}

# Argument parsing
BRANCH="main"
TAG=""
DRY_RUN=false
VERBOSE=false

while [[ $# -gt 0 ]]; do
    case "$1" in
        -b|--branch)   BRANCH="$2"; shift 2 ;;
        -t|--tag)      TAG="$2"; shift 2 ;;
        -n|--dry-run)  DRY_RUN=true; shift ;;
        -v|--verbose)  VERBOSE=true; shift ;;
        -h|--help)     usage; exit 0 ;;
        --)            shift; break ;;
        -*)            die "Unknown option: $1" ;;
        *)             break ;;
    esac
done

[[ $# -lt 1 ]] && { usage; die "Missing required argument: environment"; }
ENV="$1"

# Validate environment
case "$ENV" in
    dev|staging|prod) ;;
    *) die "Invalid environment: $ENV. Must be dev, staging, or prod" ;;
esac

# Dependency checks — fail early with clear messages
require() { command -v "$1" >/dev/null 2>&1 || die "Required command not found: $1 (install it first)"; }
require git
require docker
require aws

# Global cleanup — runs on any exit
WORK_DIR=""
cleanup() {
    local code=$?
    [[ -n "$WORK_DIR" ]] && rm -rf "$WORK_DIR"
    if [[ $code -ne 0 ]]; then
        error "Deploy failed with exit code $code"
    else
        log "Deploy completed successfully"
    fi
    exit $code
}
trap cleanup EXIT

# Prevent concurrent deploys with a lock
LOCK_FILE="/tmp/deploy-${ENV}.lock"
exec 9>"$LOCK_FILE"
flock -n 9 || die "Another deploy to $ENV is in progress. Try again in a minute."

# Main logic
main() {
    log "Starting deploy to $ENV"
    $VERBOSE && set -x

    WORK_DIR=$(mktemp -d)
    log "Working directory: $WORK_DIR"

    # Determine what to deploy
    local ref="${TAG:-$BRANCH}"
    log "Deploying ref: $ref"

    if $DRY_RUN; then
        log "[DRY RUN] Would deploy $ref to $ENV"
        log "[DRY RUN] Would run: docker build and docker push"
        return 0
    fi

    # Build
    log "Building Docker image..."
    local image_tag="myapp:${ref//\//-}-$(date +%Y%m%d%H%M%S)"
    docker build -t "$image_tag" . || die "Docker build failed"

    # Push
    log "Pushing image to registry..."
    docker push "$image_tag" || die "Docker push failed"

    # Deploy per environment
    case "$ENV" in
        dev)
            log "Deploying to dev..."
            # kubectl, helm, or whatever your deploy mechanism is
            ;;
        staging)
            log "Deploying to staging..."
            ;;
        prod)
            # Extra confirmation for production
            read -rp "Confirm production deploy of $image_tag? [yes/N] " confirm
            [[ "$confirm" == "yes" ]] || die "Aborted by user"
            log "Deploying to production..."
            ;;
    esac

    log "Deploy to $ENV complete: $image_tag"
}

main "$@"
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Spaces around the assignment operator**

```bash
# WRONG: Bash treats "name" as a command with = and "Alice" as arguments
name = "Alice"   # error: command not found: name

# CORRECT: no spaces
name="Alice"
```

**Mistake 2: Not quoting variables — the word-splitting trap**

```bash
filename="my report 2024.csv"

# WRONG: Bash splits on whitespace — wc sees 4 arguments
wc -l $filename        # error: can't open 'my', 'report', etc.

# WRONG: glob expansion on the variable value
files="*.txt"
rm $files              # expands *.txt in the current directory

# CORRECT: always quote
wc -l "$filename"
rm "$files"

# The universal rule: quote all variable references unless you
# explicitly want word splitting or glob expansion (rare).
```

**Mistake 3: Using `[ ]` instead of `[[ ]]`**

```bash
# WRONG: [ ] requires careful quoting and doesn't support all operators
value=""
if [ $value == "" ]; then   # error if $value is empty — becomes: if [ == "" ]
    echo "empty"
fi

# WRONG: and/or in [ ] use -a and -o, which are deprecated
if [ -f "file" -a -r "file" ]; then ...

# CORRECT: [[ ]] handles empty strings safely, supports &&, ||, =~
if [[ "$value" == "" ]]; then echo "empty"; fi
if [[ -f "file" && -r "file" ]]; then echo "readable file"; fi
```

**Mistake 4: Not using `set -euo pipefail`**

```bash
# Without error settings:
rm -rf /nonexistent/path    # fails silently
cp source.txt dest.txt      # if source doesn't exist, script keeps running
cat file.txt | grep "ERROR" | wc -l   # if cat fails, pipeline continues

# With set -euo pipefail:
# Any command failure causes immediate script exit
# Any unset variable reference is an error
# Any stage of a pipe failing causes the pipeline to fail

# Put this at the top of EVERY non-trivial script:
set -euo pipefail
```

**Mistake 5: Reading files with `for` instead of `while read`**

```bash
# WRONG: for with command substitution — breaks on filenames with spaces
for line in $(cat /etc/hosts); do   # splits on ANY whitespace
    echo "$line"
done
# /etc/hosts lines with multiple spaces are split into multiple "words"

# CORRECT: while read handles whitespace correctly
while IFS= read -r line; do
    echo "$line"
done < /etc/hosts
```

**Mistake 6: Forgetting `local` in functions**

```bash
# WRONG: without local, variables leak into the global scope
process() {
    count=0         # sets the GLOBAL count, overwriting any caller's count
    for i in "$@"; do
        ((count++))
    done
    echo $count
}

count=42
process a b c
echo $count    # 3 — not 42! function modified the global

# CORRECT: always declare function variables with local
process() {
    local count=0
    for i in "$@"; do
        ((count++))
    done
    echo $count
}

count=42
process a b c
echo $count    # 42 — unchanged
```

---

## 5. The "Why Does This Work" Layer

### How Bash Parses a Command Line

Understanding Bash's parsing order prevents most quoting bugs. Bash processes each line through these expansions in sequence:

```
1. Brace expansion:        {a,b,c}  →  a b c
2. Tilde expansion:        ~/dir    →  /home/alice/dir
3. Parameter expansion:    $var     →  value of var
4. Command substitution:   $(cmd)   →  output of cmd
5. Arithmetic expansion:   $((2+2)) →  4
6. Word splitting:         split on $IFS (space, tab, newline by default)
7. Pathname expansion:     *.txt    →  file1.txt file2.txt
8. Quote removal:          "hello"  →  hello
```

The consequence: quoting `"$var"` prevents steps 6 and 7 from operating on the variable's *expanded value*. Without quotes, if `$var` contains spaces or glob characters, Bash splits it into multiple words and expands globs. This is almost never what you want.

### How Exit Codes Drive Everything

In Bash, every command — whether it's a system program, a shell function, or a test — returns a number from 0 to 255. Zero means success; anything else means failure. The shell stores the last exit code in `$?`.

`if`, `while`, `until`, `&&`, and `||` are all built around this. `if command` doesn't check "the output of command" — it checks whether `command` exited with code 0.

```bash
if grep -q "ERROR" logfile.txt; then   # grep exits 0 if found, 1 if not found
    echo "Errors detected"
fi
# grep's exit code, not its output, drives the if branch.

command1 && command2    # run command2 ONLY if command1 succeeded (exit 0)
command1 || command2    # run command2 ONLY if command1 failed (non-zero exit)
```

`set -e` (errexit) makes the shell exit immediately whenever a command exits with a non-zero status. This catches errors that would otherwise be silently ignored — which is why every production script should use it.

### Why `IFS= read -r` Is the Correct File Reading Pattern

`IFS= read -r line` has three components, each solving a specific problem:

- `IFS=`: temporarily sets the Internal Field Separator to empty string. Without this, `read` strips leading and trailing whitespace from each line (because default IFS includes space and tab). `IFS=` preserves exact whitespace.
- `-r`: disables backslash interpretation. Without `-r`, a line ending with `\` would be joined with the next line, and `\n` would be interpreted as a newline.
- `< filename`: redirects the file as stdin without spawning a subshell. Using `while ... done < <(command)` with process substitution similarly avoids subshells, which means variable assignments inside the loop are visible after the loop exits.

---

## 6. Quick Reference

### Variables

| Syntax | Meaning |
|--------|---------|
| `name="value"` | Assignment (no spaces around =) |
| `"$name"` | Variable reference (always quote) |
| `"${name}suffix"` | Reference with suffix |
| `"${name:-default}"` | Use default if unset/empty |
| `"${name:=default}"` | Assign default if unset/empty |
| `"${name:?error}"` | Error if unset/empty |
| `"${#name}"` | String length |
| `"${name:2:5}"` | Substring (offset 2, length 5) |
| `"${name##*/}"` | Remove longest matching prefix |
| `"${name%.*}"` | Remove shortest matching suffix |
| `"${name/old/new}"` | Replace first match |
| `"${name//old/new}"` | Replace all matches |
| `"${name^^}"` | Uppercase |
| `"${name,,}"` | Lowercase |

### Tests `[[ ]]`

| Test | Meaning |
|------|---------|
| `-f file` | File exists and is a regular file |
| `-d dir` | Is a directory |
| `-e path` | Path exists |
| `-r / -w / -x` | Readable / writable / executable |
| `-z "$str"` | String is empty |
| `-n "$str"` | String is non-empty |
| `-eq / -ne / -lt / -gt` | Numeric comparisons |
| `==` / `!=` | String equality |
| `=~` | Regex match |

### Script Header Template

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

log()  { echo "[INFO]  $(date '+%F %T') $*"; }
die()  { echo "[ERROR] $*" >&2; exit 1; }

# Cleanup temp files on exit
WORK_DIR=$(mktemp -d)
trap 'rm -rf "$WORK_DIR"' EXIT

# Argument validation
[[ $# -lt 1 ]] && die "Usage: $(basename $0) <arg>"

main() {
    # script logic here
}

main "$@"
```

### Common One-Liners

```bash
# Run a command on each line of a file
while IFS= read -r line; do echo "$line"; done < file.txt

# Check if command exists
command -v docker >/dev/null 2>&1 || die "docker not found"

# Create temp dir with guaranteed cleanup
TMPDIR=$(mktemp -d); trap 'rm -rf "$TMPDIR"' EXIT

# Retry a command up to N times
retry() { local n=0; until [[ $n -ge $1 ]]; do "$@" && return; ((n++)); sleep 2; done; return 1; }

# Background multiple jobs and wait for all
for item in "${list[@]}"; do process "$item" & done; wait

# Read stdin if no file arg
input_file="${1:--}"   # - means stdin; use as: command "$input_file"
```
