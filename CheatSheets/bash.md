# Bash Cheat Sheet
### From Zero to Expert

---

## What is Bash?

**Bash** (Bourne Again SHell) is the default shell on most Linux distributions and macOS (pre-Catalina). It is simultaneously an interactive command interpreter and a full scripting language — capable of automating system administration, building CI/CD pipelines, processing text, and orchestrating complex workflows entirely from the command line.

```bash
#!/usr/bin/env bash
echo "Hello, World!"
```

**Key traits:** text-stream based · process-oriented · ubiquitous on Unix/Linux · POSIX-compliant · rich built-in text processing via pipes · direct system call access

---

## Running Bash

```bash
bash script.sh          # run a script
bash -x script.sh       # debug mode — print each command before running
bash -n script.sh       # syntax check only (don't execute)
bash -e script.sh       # exit immediately on any error
bash -c "echo hello"    # run inline command string
chmod +x script.sh && ./script.sh   # make executable and run directly

# Shebang line (first line of script):
#!/bin/bash             # use system bash
#!/usr/bin/env bash     # use bash found in PATH (more portable)
#!/bin/sh               # POSIX sh (avoid bashisms)
```

---

## Variables

```bash
# Assignment — NO spaces around =:
name="Alice"
age=30
pi=3.14159

# Reading a variable — $ prefix:
echo $name
echo "$name"            # always quote variables to prevent word splitting
echo "${name}"          # explicit boundary: ${var}suffix

# Readonly variables:
readonly MAX=100
declare -r CONST="immutable"

# Unset a variable:
unset name

# Check if variable is set:
[ -z "$var" ] && echo "empty or unset"
[ -n "$var" ] && echo "set and non-empty"

# Default values:
echo "${name:-Alice}"        # use "Alice" if name is unset or empty
echo "${name:=Alice}"        # assign "Alice" if name is unset or empty
echo "${name:?Must be set}"  # error if name is unset or empty
echo "${name:+override}"     # use "override" only if name IS set

# Variable types with declare:
declare -i count=0      # integer
declare -a arr          # indexed array
declare -A map          # associative array (bash 4+)
declare -l lower="HI"  # auto-lowercase → "hi"
declare -u upper="hi"  # auto-uppercase → "HI"
declare -x exported     # export to child processes (same as export)

# Environment variables:
export PATH="$PATH:/usr/local/bin"   # add to PATH
export MY_VAR="value"                # available to child processes
env                                  # print all environment variables
printenv HOME                        # print specific env var
```

---

## Special Variables

```bash
$0          # name of the script
$1 $2 ... $9 $10  # positional arguments ($10+ need braces: ${10})
$#          # number of arguments
$@          # all arguments as separate words (quote-aware: "$@")
$*          # all arguments as a single word
$$          # PID of current shell
$!          # PID of last background process
$?          # exit status of last command (0 = success)
$-          # current shell options (e.g., himBH)
$_          # last argument of previous command
$LINENO     # current line number in script
$FUNCNAME   # current function name
$BASH_VERSION   # bash version string
$HOSTNAME   # machine hostname
$PWD        # current working directory
$OLDPWD     # previous working directory
$HOME       # home directory of current user
$USER       # current username
$SHELL      # path to current shell
$PATH       # executable search path
$IFS        # Internal Field Separator (default: space, tab, newline)
$RANDOM     # random integer 0–32767
$SECONDS    # seconds since shell started
$EPOCHSECONDS  # Unix timestamp (bash 5+)
PIPESTATUS  # array of exit codes from last pipeline
```

---

## Quoting

Quoting is one of the most important — and most misunderstood — aspects of Bash.

```bash
# No quotes — subject to word splitting and globbing:
echo $name              # fine for simple cases
echo $name $age         # DANGEROUS if values have spaces

# Double quotes — allows variable and command substitution, prevents splitting:
echo "$name"            # safe — preserves spaces in $name
echo "Hello, $name!"    # variable interpolated
echo "Files: $(ls)"     # command substitution interpolated
echo "Tab:\t newline:\n" # escape sequences NOT processed (use $'...' instead)

# Single quotes — completely literal, nothing is interpreted:
echo 'Hello, $name!'    # prints literally: Hello, $name!
echo 'no $(substitution) here'

# ANSI-C quoting $'...' — processes escape sequences:
echo $'Hello\tWorld\n'  # tab and newline ARE processed
echo $'Line1\nLine2'
printf '%s\n' $'it\'s'  # can include single quotes

# Here-string:
cat <<< "This is a here-string passed to stdin"
grep "pattern" <<< "$variable"

# Here-document:
cat <<EOF
Line 1: $name
Line 2: $(date)
EOF

cat <<'EOF'             # single-quoted heredoc — no interpolation
Literal $variable
No $(substitution)
EOF

cat <<-EOF              # strip leading tabs (not spaces)
	Indented content
	More content
EOF
```

---

## Strings

```bash
str="Hello, Bash World!"

# Length:
echo ${#str}                    # 18

# Substring (offset, length):
echo ${str:7}                   # "Bash World!"
echo ${str:7:4}                 # "Bash"
echo ${str: -6}                 # "World!"  (negative offset needs space)

# Case modification (bash 4+):
echo ${str,,}                   # all lowercase
echo ${str^^}                   # all uppercase
echo ${str,}                    # first char lowercase
echo ${str^}                    # first char uppercase

# Prefix removal:
path="/usr/local/bin/bash"
echo ${path#*/}                 # "usr/local/bin/bash"  (remove shortest prefix)
echo ${path##*/}                # "bash"                (remove longest prefix)

# Suffix removal:
file="document.tar.gz"
echo ${file%.*}                 # "document.tar"        (remove shortest suffix)
echo ${file%%.*}                # "document"            (remove longest suffix)

# Substitution:
echo ${str/Bash/Shell}          # replace first occurrence
echo ${str//o/0}                # replace ALL occurrences
echo ${str/#Hello/Hi}           # replace only at start
echo ${str/%!/...}              # replace only at end

# String contains (no built-in — use case or [[ ]]):
if [[ "$str" == *"Bash"* ]]; then echo "contains Bash"; fi
if [[ "$str" =~ [0-9]+ ]]; then echo "contains digits"; fi

# Concatenation:
greeting="Hello"
greeting+=" World"              # append in-place
result="${greeting}!"

# Split string on delimiter:
IFS=',' read -ra parts <<< "a,b,c,d"
for part in "${parts[@]}"; do echo "$part"; done

# Trim whitespace (no built-in — use parameter expansion):
str="  hello  "
trimmed="${str#"${str%%[![:space:]]*}"}"   # ltrim
trimmed="${trimmed%"${trimmed##*[![:space:]]}"}"  # rtrim
# Or with sed:
trimmed=$(echo "$str" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
```

---

## Numbers & Arithmetic

```bash
# Arithmetic expansion (integer only):
result=$((2 + 3))           # 5
result=$((10 / 3))          # 3 (integer division)
result=$((10 % 3))          # 1
result=$((2 ** 8))          # 256 (exponentiation)
result=$((x++))             # post-increment
result=$((++x))             # pre-increment

# let command:
let "result = 5 * 3"
let "count++"

# Compound arithmetic (no $ needed inside):
((count++))
((result = a + b))
if ((count > 5)); then echo "big"; fi

# Floating point — requires bc or awk (bash is integer-only):
echo "scale=4; 10/3" | bc        # 3.3333
echo "scale=2; sqrt(2)" | bc -l  # 1.41
awk "BEGIN { printf \"%.2f\n\", 10/3 }"  # 3.33

# printf for formatted numbers:
printf "%.2f\n" 3.14159     # 3.14
printf "%05d\n" 42          # 00042
printf "%x\n" 255           # ff (hex)
printf "%o\n" 8             # 10 (octal)

# Arithmetic operators inside (( )):
+  -  *  /  %  **            # basic
++  --                        # increment/decrement
&  |  ^  ~  <<  >>           # bitwise
&&  ||  !                    # logical
```

---

## Arrays

```bash
# Indexed arrays (0-based):
fruits=("apple" "banana" "cherry")
fruits[3]="date"                  # add element
fruits+=("elderberry")            # append element(s)

# Access:
echo "${fruits[0]}"               # "apple"
echo "${fruits[-1]}"              # last element (bash 4.2+)
echo "${fruits[@]}"               # ALL elements (as separate words)
echo "${fruits[*]}"               # all elements (as one word when quoted)
echo "${#fruits[@]}"              # array length (number of elements)
echo "${!fruits[@]}"              # all indices

# Slicing:
echo "${fruits[@]:1:2}"           # elements at index 1 and 2

# Iterate:
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Iterate with index:
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Delete element:
unset fruits[1]                   # remove "banana" (leaves a gap)
fruits=("${fruits[@]}")           # re-index to close gaps

# Delete entire array:
unset fruits

# Array from command output:
files=($(ls *.txt))               # DANGEROUS: word splitting issues
mapfile -t files < <(ls *.txt)    # SAFE: read into array line by line
readarray -t lines < file.txt     # read file into array

# Check if element exists:
if [[ -v fruits[2] ]]; then echo "index 2 exists"; fi

# Associative arrays (bash 4+):
declare -A person
person[name]="Alice"
person[age]=30
person["full name"]="Alice Smith"

echo "${person[name]}"            # "Alice"
echo "${person[@]}"               # all values
echo "${!person[@]}"              # all keys

for key in "${!person[@]}"; do
    echo "$key = ${person[$key]}"
done
```

---

## Control Flow

```bash
# if / elif / else:
if [ "$age" -ge 18 ]; then
    echo "Adult"
elif [ "$age" -ge 13 ]; then
    echo "Teen"
else
    echo "Child"
fi

# [[ ]] — modern bash test (preferred over [ ]):
if [[ "$name" == "Alice" ]]; then echo "Hi Alice"; fi
if [[ "$str" =~ ^[0-9]+$ ]]; then echo "All digits"; fi
if [[ -f "/etc/passwd" && -r "/etc/passwd" ]]; then echo "Readable file"; fi

# Comparison operators:
# Numeric:  -eq  -ne  -lt  -gt  -le  -ge
# String:   ==   !=   <    >    -z (empty)  -n (non-empty)
# File:     -f (file)  -d (dir)  -e (exists)  -r -w -x (permissions)
#           -s (non-empty file)  -L (symlink)  -N (modified since read)
# Combined: && (and)  || (or)  ! (not)

# case statement:
case "$extension" in
    txt|md)
        echo "Text file"
        ;;
    jpg|jpeg|png|gif)
        echo "Image file"
        ;;
    sh|bash)
        echo "Shell script"
        ;;
    *)
        echo "Unknown"
        ;;
esac

# Ternary-style (short-circuit):
[ "$x" -gt 0 ] && echo "positive" || echo "non-positive"
result=$( [[ "$score" -ge 60 ]] && echo "pass" || echo "fail" )

# for loop:
for i in 1 2 3 4 5; do
    echo $i
done

for i in {1..10}; do echo $i; done       # brace expansion range
for i in {0..20..2}; do echo $i; done    # step of 2
for f in *.txt; do echo "$f"; done        # glob expansion

# C-style for loop:
for ((i=0; i<10; i++)); do
    echo $i
done

# while loop:
count=0
while [ $count -lt 5 ]; do
    echo $count
    ((count++))
done

# Read file line by line (safe pattern):
while IFS= read -r line; do
    echo "$line"
done < input.txt

# Read from command output:
while IFS= read -r line; do
    echo "$line"
done < <(find . -name "*.txt")

# until loop (runs until condition is TRUE):
until [ -f /tmp/done.flag ]; do
    sleep 5
    echo "Waiting..."
done

# Loop control:
break           # exit loop
break 2         # exit 2 levels of nested loops
continue        # skip to next iteration
continue 2      # skip 2 levels up
```

---

## Functions

```bash
# Definition syntax (two equivalent forms):
greet() {
    echo "Hello, $1!"
}

function greet {
    echo "Hello, $1!"
}

# Calling:
greet "Alice"       # Hello, Alice!
greet Alice         # same (quotes optional for single words)

# Local variables (ALWAYS use local inside functions):
calculate() {
    local result=$(( $1 + $2 ))   # local scope — won't leak out
    echo $result                   # "return" value via stdout
}
sum=$(calculate 3 5)               # capture output

# Return codes (0–255; 0 = success):
is_even() {
    (( $1 % 2 == 0 ))   # returns 0 if true, 1 if false
}
if is_even 4; then echo "even"; fi

# Returning values — Bash has no return value, only exit codes.
# Pattern 1: echo the result and capture with $()
# Pattern 2: use a nameref (bash 4.3+):
get_result() {
    local -n _ref=$1        # nameref to caller's variable
    _ref="computed value"
}
get_result myvar
echo "$myvar"               # "computed value"

# Variadic functions:
log() {
    local level="$1"
    shift                   # remove $1 from args; $2 becomes $1
    echo "[$level] $*"
}
log "INFO" "This" "is" "a" "message"

# Default parameter values:
connect() {
    local host="${1:-localhost}"
    local port="${2:-8080}"
    echo "Connecting to $host:$port"
}
connect                     # localhost:8080
connect myserver            # myserver:8080
connect myserver 443        # myserver:443

# Recursive function:
factorial() {
    (( $1 <= 1 )) && echo 1 && return
    echo $(( $1 * $(factorial $(( $1 - 1 ))) ))
}

# Exporting functions to subshells:
export -f greet
```

---

## Input & Output

```bash
# Read from stdin:
read name                       # read one line into $name
read -p "Enter name: " name     # with prompt
read -s -p "Password: " pass    # silent (no echo)
read -t 10 -p "Timeout in 10s: " input  # timeout
read -n 1 char                  # read exactly 1 character
read -a arr                     # read into array

# printf — preferred over echo for reliable output:
printf "Hello, %s!\n" "Alice"
printf "%-20s %5d\n" "Item" 42       # left-aligned string, right-aligned int
printf "%04X\n" 255                  # hex: 00FF

# echo options:
echo "text"         # with newline
echo -n "no newline"
echo -e "Tab:\t Newline:\n"   # interpret escapes (-e flag)

# Redirection:
command > file.txt      # stdout to file (overwrite)
command >> file.txt     # stdout to file (append)
command < file.txt      # stdin from file
command 2> err.txt      # stderr to file
command 2>&1            # stderr to same place as stdout
command &> file.txt     # stdout + stderr to file (bash shorthand)
command > file.txt 2>&1 # stdout + stderr to file (POSIX)
command 2>/dev/null     # discard stderr
command > /dev/null 2>&1 # discard all output

# Redirect stdout and stderr to different files:
command > out.txt 2> err.txt

# Process substitution (treat command output as a file):
diff <(sort file1.txt) <(sort file2.txt)
while IFS= read -r line; do ...; done < <(command)

# Named pipes (FIFO):
mkfifo mypipe
command1 > mypipe &
command2 < mypipe

# tee — write to file AND stdout simultaneously:
command | tee output.txt
command | tee -a output.txt   # append mode
command | tee file1 file2     # multiple files

# /dev/fd and /dev/stdin:
echo "text" | cat /dev/stdin  # read from stdin explicitly
```

---

## Pipes & Process Substitution

```bash
# Basic pipe — stdout of left → stdin of right:
ls -la | grep ".txt"
cat file.txt | sort | uniq -c | sort -rn | head -10

# Exit status of a pipeline — only last command's status by default:
set -o pipefail     # make pipeline fail if ANY command fails

# PIPESTATUS array — exit codes of all commands in last pipeline:
ls | grep ".txt" | wc -l
echo "${PIPESTATUS[@]}"    # e.g., "0 0 0"

# xargs — build command from stdin:
find . -name "*.tmp" | xargs rm -f
find . -name "*.txt" | xargs -I{} cp {} /backup/
echo "file1 file2" | xargs -n1 echo   # one argument per line
cat list.txt | xargs -P4 -I{} process_item {}  # 4 parallel processes

# Subshell (runs in child process — variable changes don't propagate):
(cd /tmp && ls)             # cd doesn't affect parent
result=$(expensive_command) # command substitution uses subshell

# Command grouping (runs in current shell):
{ echo "one"; echo "two"; } > combined.txt
{ cd /tmp && ls; }          # cd DOES affect current shell

# Background execution:
long_command &              # run in background
pid=$!                      # capture PID
wait $pid                   # wait for specific background job
wait                        # wait for ALL background jobs
jobs                        # list background jobs
fg %1                       # bring job 1 to foreground
bg %1                       # send job 1 to background
kill %1                     # kill job 1
```

---

## Exit Codes & Error Handling

```bash
# Exit codes: 0 = success, 1-255 = failure
# Common conventions:
# 0   success
# 1   general error
# 2   misuse of shell built-ins
# 126 command found but not executable
# 127 command not found
# 128+N signal N received (e.g., 130 = Ctrl+C)

# Check exit code:
command
if [ $? -eq 0 ]; then echo "success"; fi
# Or more idiomatically:
if command; then echo "success"; fi
command || echo "failed"
command && echo "succeeded"

# set options for robust scripts:
set -e          # exit on error (errexit)
set -u          # treat unset variables as errors (nounset)
set -o pipefail # pipeline fails if any command fails
set -x          # print each command (xtrace — for debugging)
set -E          # ERR trap inherited by functions/subshells

# Best practice — put this at the top of every script:
set -euo pipefail

# Trap — run code on signals or exit:
trap 'echo "Exiting"; cleanup' EXIT         # always runs on exit
trap 'echo "Interrupted"' INT               # Ctrl+C
trap 'echo "Terminated"' TERM
trap 'echo "Error at line $LINENO"' ERR     # any error (with set -e)

# Cleanup pattern:
TMPFILE=$(mktemp)
trap 'rm -f "$TMPFILE"' EXIT    # guaranteed cleanup even on error

cleanup() {
    local exit_code=$?
    rm -f "$TMPFILE"
    exit $exit_code
}
trap cleanup EXIT

# Disable trap:
trap - EXIT

# Exit with a code:
exit 0      # success
exit 1      # general failure
exit $?     # propagate last command's exit code
```

---

## Text Processing

```bash
# grep — search for patterns:
grep "pattern" file.txt
grep -i "pattern" file.txt    # case-insensitive
grep -r "pattern" ./dir       # recursive
grep -n "pattern" file.txt    # show line numbers
grep -v "pattern" file.txt    # invert (lines that DON'T match)
grep -c "pattern" file.txt    # count matching lines
grep -l "pattern" *.txt       # only list matching filenames
grep -w "word" file.txt       # whole word match
grep -A3 "pattern" file.txt   # 3 lines after match
grep -B3 "pattern" file.txt   # 3 lines before match
grep -C3 "pattern" file.txt   # 3 lines of context
grep -E "pattern1|pattern2"   # extended regex (ERE)
grep -P "\d+"                 # Perl-compatible regex (PCRE)

# sed — stream editor:
sed 's/old/new/' file         # replace first occurrence per line
sed 's/old/new/g' file        # replace ALL occurrences
sed 's/old/new/gi' file       # case-insensitive replace all
sed -i 's/old/new/g' file     # in-place edit (modifies file)
sed -i.bak 's/old/new/g' file # in-place with backup
sed '3d' file                 # delete line 3
sed '2,5d' file               # delete lines 2–5
sed '/pattern/d' file         # delete lines matching pattern
sed -n '5,10p' file           # print only lines 5–10
sed -n '/start/,/end/p' file  # print between patterns
sed '5a\new line' file        # append line after line 5
sed '5i\new line' file        # insert line before line 5
sed -n 's/pattern/&/p'        # print only matching lines (&=matched text)

# awk — field processing:
awk '{print $1}' file         # print first field (whitespace delimited)
awk '{print $NF}' file        # print last field
awk -F: '{print $1}' /etc/passwd  # colon-delimited
awk 'NR==5' file              # print line 5
awk 'NR>=2 && NR<=5' file     # lines 2–5
awk '/pattern/' file          # print lines matching pattern
awk '!/pattern/' file         # print lines NOT matching
awk '{sum += $3} END {print sum}' file    # sum column 3
awk 'BEGIN {FS=":"; OFS="\t"} {print $1,$3}' /etc/passwd
awk '{if ($2 > 100) print $1, $2}' file
awk 'length > 80' file        # lines longer than 80 chars
awk '{gsub(/old/, "new"); print}' file    # global substitute

# cut — extract columns:
cut -d: -f1 /etc/passwd       # first field, colon delimiter
cut -d, -f2,4 file.csv        # fields 2 and 4
cut -c1-10 file               # characters 1–10

# sort:
sort file                     # alphabetical sort
sort -n file                  # numeric sort
sort -r file                  # reverse sort
sort -u file                  # unique (remove duplicates)
sort -k2 file                 # sort by field 2
sort -k2 -n file              # sort by field 2 numerically
sort -t: -k3 -n /etc/passwd   # sort passwd by UID

# uniq:
sort file | uniq              # remove adjacent duplicates (sort first!)
sort file | uniq -c           # count occurrences
sort file | uniq -d           # only show duplicates
sort file | uniq -u           # only show unique lines

# tr — translate/delete characters:
echo "hello" | tr 'a-z' 'A-Z'    # uppercase
echo "hello world" | tr -d ' '   # delete spaces
echo "hello   world" | tr -s ' ' # squeeze multiple spaces to one
echo "abc" | tr 'abc' '123'       # character mapping

# wc — word/line/byte count:
wc -l file        # line count
wc -w file        # word count
wc -c file        # byte count
wc -m file        # character count

# head / tail:
head -20 file     # first 20 lines (default: 10)
tail -20 file     # last 20 lines
tail -f file      # follow (watch file grow — like log tailing)
tail -F file      # follow by name (handles log rotation)
tail -n +5 file   # from line 5 onwards

# Other text tools:
column -t file            # align into neat columns
column -t -s, file.csv    # CSV → table
paste file1 file2         # merge files side by side
join file1 file2          # join on common field (like SQL JOIN)
diff file1 file2          # line-by-line differences
comm file1 file2          # compare two sorted files
tac file                  # reverse order of lines
rev file                  # reverse each line
fold -w 80 file           # wrap long lines at 80 chars
```

---

## File System Operations

```bash
# Navigation:
cd /path/to/dir
cd ~            # home directory
cd -            # previous directory
cd ..           # parent directory
pwd             # print working directory

# Listing:
ls              # basic list
ls -la          # long format + hidden files
ls -lh          # human-readable sizes
ls -lt          # sort by modification time (newest first)
ls -lS          # sort by size (largest first)
ls -R           # recursive
ls -1           # one file per line

# Finding files:
find . -name "*.txt"
find . -name "*.txt" -type f         # files only
find . -type d -name "logs"          # directories named "logs"
find . -mtime -7                     # modified in last 7 days
find . -mtime +30                    # modified more than 30 days ago
find . -size +10M                    # larger than 10 MB
find . -perm 644                     # specific permissions
find . -user alice                   # owned by alice
find . -name "*.log" -exec rm {} \;  # delete all .log files
find . -name "*.txt" -exec grep -l "pattern" {} +  # efficient batch exec
find . -name "*.txt" | xargs grep "pattern"         # alternative with xargs

# File operations:
cp source dest              # copy file
cp -r source/ dest/         # copy directory recursively
cp -p file dest             # preserve timestamps/permissions
mv old new                  # move / rename
rm file                     # delete file
rm -rf dir/                 # delete directory recursively (DANGEROUS)
mkdir newdir
mkdir -p a/b/c              # create parent directories as needed
rmdir emptydir              # delete empty directory
ln -s target link           # create symbolic link
ln target hardlink          # create hard link
touch file.txt              # create empty file / update timestamp
stat file.txt               # detailed file metadata
file document               # detect file type
du -sh dir/                 # disk usage of directory
du -sh *                    # disk usage of all items in current dir
df -h                       # disk free space (all mounts)

# Permissions:
chmod 755 script.sh         # rwxr-xr-x
chmod +x script.sh          # add execute permission
chmod -w file.txt           # remove write permission
chmod u+x,g-w file          # user +execute, group -write
chown user:group file       # change owner and group
chown -R alice: dir/        # recursive ownership change
umask 022                   # default permission mask

# Archives:
tar -czf archive.tar.gz dir/     # create gzipped tar
tar -xzf archive.tar.gz          # extract gzipped tar
tar -czf - dir/ | ssh host "cat > /path/archive.tar.gz"  # stream over SSH
tar -tf archive.tar.gz           # list contents without extracting
zip -r archive.zip dir/
unzip archive.zip
gzip file.txt                    # compress (replaces file with file.txt.gz)
gunzip file.txt.gz               # decompress
```

---

## Process Management

```bash
# View processes:
ps aux                      # all processes, detailed
ps aux | grep "nginx"
top                         # interactive process monitor
htop                        # enhanced top (if installed)
pgrep nginx                 # find PIDs by name
pgrep -a nginx              # show full command lines

# Signals:
kill PID                    # SIGTERM (graceful shutdown)
kill -9 PID                 # SIGKILL (force kill, unblockable)
kill -HUP PID               # SIGHUP (reload config)
killall nginx               # kill by name (all instances)
pkill -f "python script.py" # kill by command pattern

# Job control:
command &                   # run in background
jobs                        # list jobs for current shell
fg                          # bring last background job to foreground
fg %2                       # bring job 2 to foreground
bg %2                       # resume job 2 in background
Ctrl+Z                      # suspend current foreground job
Ctrl+C                      # send SIGINT to foreground job
disown %1                   # detach job from shell (survives logout)
nohup command &             # immune to hangup, output to nohup.out

# Priority:
nice -n 10 command          # run with lower priority (niceness 10)
renice -n 5 -p PID          # change priority of running process

# Timing:
time command                # measure execution time
timeout 30 command          # kill command after 30 seconds

# Subshells and process substitution:
(command1; command2)        # run in subshell
{ command1; command2; }     # run in current shell (note: trailing semicolon)
diff <(sort a.txt) <(sort b.txt)
```

---

## Networking

```bash
# Connectivity:
ping -c 4 google.com        # ping 4 times
ping -i 0.5 host            # ping every 0.5 seconds
curl https://example.com                       # fetch URL
curl -o file.html https://example.com          # save to file
curl -L https://example.com                    # follow redirects
curl -I https://example.com                    # headers only (HEAD)
curl -X POST -d '{"key":"val"}' -H "Content-Type: application/json" URL
curl -u user:pass https://api.example.com      # basic auth
curl -b cookies.txt -c cookies.txt URL         # cookie handling
wget https://example.com/file.zip              # download file
wget -r -np https://example.com/              # recursive download

# DNS:
nslookup google.com
dig google.com
dig google.com A            # A record only
dig +short google.com       # just the IP
host google.com

# Network info:
ip addr show                # network interfaces and IPs
ip route show               # routing table
netstat -tuln               # listening ports (deprecated but still common)
ss -tuln                    # modern replacement for netstat
ss -tulnp                   # include process names
lsof -i :8080               # what's using port 8080
nmap -p 1-1000 host         # port scan

# SSH:
ssh user@host
ssh -p 2222 user@host       # non-standard port
ssh -i key.pem user@host    # specific key file
ssh -L 8080:localhost:80 user@host  # local port forwarding
ssh -R 8080:localhost:80 user@host  # remote port forwarding
ssh -D 1080 user@host       # dynamic (SOCKS proxy)
ssh -N user@host            # no command — tunnel only

# SCP / RSYNC:
scp file.txt user@host:/path/
scp user@host:/path/file.txt local/
rsync -avz source/ user@host:/dest/    # sync directory
rsync -avz --delete source/ dest/      # mirror (delete extraneous)
rsync -avz --exclude "*.tmp" src/ dst/ # exclude pattern

# Firewall (ufw / iptables):
ufw status
ufw allow 22/tcp
ufw allow from 192.168.1.0/24
iptables -L -n              # list rules
```

---

## Advanced Techniques

### Brace Expansion

```bash
echo {a,b,c}                # a b c
echo {1..5}                 # 1 2 3 4 5
echo {01..05}               # 01 02 03 04 05
echo {a..z}                 # a b c ... z
echo {A..Z}                 # A B C ... Z
echo file{1,2,3}.txt        # file1.txt file2.txt file3.txt
mkdir -p project/{src,lib,test,docs}   # create multiple directories
cp file.conf{,.bak}         # backup: cp file.conf file.conf.bak

# Nested:
echo {a,b}{1,2}             # a1 a2 b1 b2
```

### Parameter Expansion (Advanced)

```bash
${!prefix*}       # variable names starting with prefix
${!arr[@]}        # indices of array arr
${var@Q}          # quote value for reuse as input (bash 4.4+)
${var@U}          # uppercase (bash 4.4+)
${var@L}          # lowercase (bash 4.4+)
${var@a}          # attributes of variable

# Indirect reference:
varname="greeting"
greeting="hello"
echo "${!varname}"  # "hello" — dereference $varname as variable name
```

### Process Coprocesses

```bash
# coproc — run command as background coprocess with bidirectional pipes:
coproc myproc { bc -l; }
echo "scale=4; sqrt(2)" >&"${myproc[1]}"
read result <&"${myproc[0]}"
echo "$result"        # 1.4142
```

### Regular Expressions in Bash

```bash
# =~ operator in [[ ]] uses POSIX extended regex:
if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# Capture groups go into BASH_REMATCH:
if [[ "2025-03-06" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
    echo "Year:  ${BASH_REMATCH[1]}"   # 2025
    echo "Month: ${BASH_REMATCH[2]}"   # 03
    echo "Day:   ${BASH_REMATCH[3]}"   # 06
fi
```

### Debugging

```bash
bash -x script.sh           # trace execution
bash -v script.sh           # verbose (print lines as read)
bash -n script.sh           # syntax check only

# In-script debugging:
set -x                      # enable trace from this point
set +x                      # disable trace

# Debug specific section:
set -x
critical_function
set +x

# PS4 — customize the trace prefix:
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'

# Check for undefined variables:
set -u                      # error on unset variable reference

# Print stack trace:
print_trace() {
    local i=0
    while caller $i; do ((i++)); done
}
trap 'print_trace' ERR
```

### Here-Document Tricks

```bash
# Feed SQL to a database:
mysql -u root -p mydb <<EOF
SELECT * FROM users WHERE active=1;
UPDATE users SET last_seen=NOW() WHERE id=42;
EOF

# Configure a remote server:
ssh user@host <<'REMOTE'
sudo apt update
sudo apt install -y nginx
sudo systemctl enable nginx
REMOTE

# Create a script dynamically:
cat > /tmp/worker.sh <<'EOF'
#!/bin/bash
echo "Running on $(hostname)"
EOF
chmod +x /tmp/worker.sh
```

---

## Scripting Patterns & Best Practices

```bash
#!/usr/bin/env bash
# Script template — use this as a starting point:

set -euo pipefail           # exit on error, unset vars, pipe failures
IFS=$'\n\t'                 # safer IFS (only split on newline and tab)

# Constants (UPPER_CASE by convention):
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOGFILE="/var/log/${SCRIPT_NAME%.sh}.log"

# Colors (if terminal supports it):
RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'; NC='\033[0m'
[[ -t 1 ]] || { RED=''; GREEN=''; YELLOW=''; NC=''; }  # no color if not TTY

# Logging:
log()   { echo -e "${GREEN}[INFO]${NC}  $(date '+%F %T') $*" | tee -a "$LOGFILE"; }
warn()  { echo -e "${YELLOW}[WARN]${NC}  $(date '+%F %T') $*" | tee -a "$LOGFILE" >&2; }
error() { echo -e "${RED}[ERROR]${NC} $(date '+%F %T') $*" | tee -a "$LOGFILE" >&2; }
die()   { error "$*"; exit 1; }

# Argument parsing:
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <input_file>

Options:
  -o, --output FILE   Output file (default: output.txt)
  -v, --verbose       Enable verbose output
  -h, --help          Show this help message
EOF
}

VERBOSE=false
OUTPUT="output.txt"

while [[ $# -gt 0 ]]; do
    case "$1" in
        -o|--output)  OUTPUT="$2"; shift 2 ;;
        -v|--verbose) VERBOSE=true; shift ;;
        -h|--help)    usage; exit 0 ;;
        --)           shift; break ;;
        -*)           die "Unknown option: $1" ;;
        *)            break ;;
    esac
done

[[ $# -lt 1 ]] && die "Missing required argument: input_file"
INPUT="$1"

# Temp files with guaranteed cleanup:
TMPDIR_WORK=$(mktemp -d)
trap 'rm -rf "$TMPDIR_WORK"' EXIT

# Dependency check:
require() {
    command -v "$1" >/dev/null 2>&1 || die "Required command not found: $1"
}
require curl; require jq; require awk

# Locking (prevent concurrent runs):
LOCKFILE="/tmp/${SCRIPT_NAME}.lock"
exec 9>"$LOCKFILE"
flock -n 9 || die "Another instance is already running"

# Run as root check:
[[ $EUID -eq 0 ]] || die "This script must be run as root"

# Main logic:
main() {
    log "Starting $SCRIPT_NAME"
    $VERBOSE && set -x
    # ... your logic here ...
    log "Done"
}

main "$@"
```

---

## Quick Reference Card

```
VARIABLES
  $var / ${var}         read variable
  ${var:-default}       use default if unset
  ${var:=default}       assign default if unset
  ${#var}               string length
  ${var:offset:len}     substring
  ${var/old/new}        replace first
  ${var//old/new}       replace all
  ${var#prefix}         remove shortest prefix
  ${var##prefix}        remove longest prefix
  ${var%suffix}         remove shortest suffix
  ${var%%suffix}        remove longest suffix
  ${var,,}  ${var^^}    lowercase / uppercase

SPECIAL VARIABLES
  $0  script name     $#  arg count
  $@  all args        $?  last exit code
  $$  current PID     $!  last background PID
  $_  last argument   $LINENO  current line

TESTS  [ ]  or  [[ ]]
  -f  is file        -d  is directory    -e  exists
  -r -w -x           readable writable executable
  -z  empty string   -n  non-empty string
  -eq -ne -lt -gt -le -ge   numeric comparison
  ==  !=  <  >             string comparison
  =~                       regex match ([[ only]])

LOOPS
  for i in {1..10}; do ... done
  for i in "${arr[@]}"; do ... done
  for ((i=0;i<n;i++)); do ... done
  while condition; do ... done
  until condition; do ... done
  while IFS= read -r line; do ... done < file

REDIRECTION
  > file   overwrite     >> file   append
  < file   stdin         2> file   stderr
  &> file  stdout+stderr  2>&1     stderr→stdout
  | pipe   2>/dev/null   discard stderr

ARRAYS
  arr=(a b c)         declare
  "${arr[@]}"         all elements (safe)
  "${#arr[@]}"        length
  "${!arr[@]}"        all indices
  arr+=(d)            append

BEST PRACTICES
  set -euo pipefail     always at top of scripts
  Quote "$variables"    prevent word splitting
  Use local in functions
  Use [[ ]] over [ ]
  Use $() over backticks
  Use printf over echo
  Always handle cleanup with trap EXIT
```
