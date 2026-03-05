# PowerShell Cheat Sheet
### From Zero to Expert

---

## What is PowerShell?

**PowerShell** is a cross-platform shell and scripting language built on .NET. Unlike traditional shells that pass text between commands, PowerShell passes **objects** — structured data with properties and methods. This makes filtering, transforming, and acting on data dramatically more powerful than text-based shells.

```powershell
# Hello, World!
Write-Host "Hello, World!"

# PowerShell's superpower: objects, not text
Get-Process | Where-Object CPU -gt 100 | Select-Object Name, CPU | Sort-Object CPU -Descending
```

**Key traits:** object pipeline · .NET integration · verb-noun cmdlet naming · built-in remoting · cross-platform (Windows, Linux, macOS) · backwards-compatible with cmd.exe

---

## Versions & Running PowerShell

```powershell
# Check your version:
$PSVersionTable.PSVersion

# Windows PowerShell (built-in, ships with Windows):  version 5.1
# PowerShell 7+  (cross-platform, install separately): pwsh

# Run PowerShell:
powershell.exe          # Windows PowerShell 5.1
pwsh                    # PowerShell 7+
pwsh -File script.ps1   # run a script file
pwsh -Command "Get-Date" # run inline command

# Execution policy (controls which scripts can run):
Get-ExecutionPolicy
Set-ExecutionPolicy RemoteSigned   # allow local + signed remote scripts
Set-ExecutionPolicy Bypass -Scope Process  # bypass for this session only
```

---

## Getting Help

PowerShell's help system is comprehensive and built in.

```powershell
Get-Help Get-Process              # full help for a cmdlet
Get-Help Get-Process -Examples    # show usage examples only
Get-Help Get-Process -Online      # open browser to online docs
Get-Help *network*                # search for help topics matching "network"
Update-Help                       # download latest help files

Get-Command                       # list ALL available commands
Get-Command -Verb Get             # all cmdlets starting with Get
Get-Command -Noun Process         # all cmdlets ending with Process
Get-Command *network*             # search by name pattern

Get-Member                        # inspect object properties and methods
Get-Process | Get-Member          # what properties does a process object have?

# Aliases for lazy typing:
Get-Alias gps                     # what does 'gps' expand to?
Get-Alias -Definition Get-Process # what aliases exist for Get-Process?
```

---

## Variables

```powershell
# Declaration — no keyword needed, $ prefix always required:
$name = "Alice"
$age  = 30
$pi   = 3.14159

# Explicit typing (optional but makes intent clear):
[string]$name  = "Alice"
[int]$age      = 30
[bool]$active  = $true
[double]$ratio = 1.618

# Multiple assignment:
$a, $b, $c = 1, 2, 3
$x, $y = $y, $x          # swap values

# Special built-in variables:
$_            # current pipeline object (used in Where-Object, ForEach-Object)
$PSItem       # same as $_ (more readable alias)
$true         # boolean true
$false        # boolean false
$null         # null / nothing
$?            # $true if last command succeeded
$LASTEXITCODE # exit code of last native executable
$Error        # array of recent errors ($Error[0] = most recent)
$PSVersionTable  # version info
$env:PATH     # environment variables (via $env: drive)
$HOME         # user's home directory
$PWD          # current working directory
$MyInvocation # info about the current script/function
$args         # arguments passed to a script (unnamed)
$input        # pipeline input in a function

# Environment variables:
$env:USERNAME
$env:COMPUTERNAME
$env:TEMP
$env:PATH = $env:PATH + ";C:\MyTools"  # append to PATH

# Variable operations:
Remove-Variable name         # delete variable
Clear-Variable name          # set to $null
$name -eq $null              # check if null
[string]::IsNullOrEmpty($s)  # check null or empty string
```

---

## Data Types

```powershell
# Strings:
$s = "Hello, $name!"            # double-quoted: variable interpolation
$s = 'Hello, $name!'            # single-quoted: literal (no interpolation)
$s = "Tab:`t Newline:`n Done"   # escape sequences with backtick `

# Here-string (multi-line):
$text = @"
Line one
Line two: $name
Line three
"@   # closing @" must be at the start of the line

$literal = @'
No $interpolation here
'@

# Numbers:
$i = 42           # Int32
$l = 100GB        # Int64 (KB, MB, GB, TB, PB suffixes supported)
$f = 3.14         # Double
$hex = 0xFF       # 255 (hex literal)
$bin = 0b1010     # 10 (binary, PS 7+)

# Booleans:
$true   $false

# Null:
$null

# Arrays:
$arr = 1, 2, 3, 4, 5           # comma-separated
$arr = @(1, 2, 3)              # explicit array literal
$arr = @()                     # empty array
$arr[0]                        # first element (0-indexed)
$arr[-1]                       # last element
$arr[1..3]                     # slice: elements 1,2,3
$arr.Count                     # number of elements
$arr.Length                    # same as Count
$arr += 6                      # append (creates new array)
$arr -contains 3               # $true
$arr -notcontains 99           # $true

# Strongly-typed array:
[int[]]$nums = 1, 2, 3

# Hash tables (dictionaries):
$ht = @{
    Name = "Alice"
    Age  = 30
    City = "Cairo"
}
$ht["Name"]           # "Alice"
$ht.Name              # "Alice" (dot notation)
$ht["Job"] = "Dev"    # add/update key
$ht.Remove("City")    # remove key
$ht.ContainsKey("Age")# $true
$ht.Keys              # all keys
$ht.Values            # all values
$ht.Count             # number of pairs

# Ordered hash table (preserves insertion order):
$ordered = [ordered]@{ A = 1; B = 2; C = 3 }

# Type conversions:
[int]"42"             # string → int
[string]42            # int → string
[double]"3.14"        # string → double
[char]65              # int → char 'A'
[datetime]"2025-01-01"# string → DateTime object
```

---

## Strings In Depth

```powershell
$s = "Hello, PowerShell!"

# Common string methods (.NET methods via dot notation):
$s.Length                        # 19
$s.ToUpper()                     # "HELLO, POWERSHELL!"
$s.ToLower()                     # "hello, powershell!"
$s.Trim()                        # remove leading/trailing whitespace
$s.TrimStart()  $s.TrimEnd()
$s.Replace("PowerShell", "World")# "Hello, World!"
$s.Contains("Power")             # $true
$s.StartsWith("Hello")           # $true
$s.EndsWith("!")                 # $true
$s.IndexOf("P")                  # 7
$s.Substring(7, 10)              # "PowerShell"
$s.Split(",")                    # @("Hello", " PowerShell!")
$s.Split(" ", 2)                 # split into max 2 parts

# PowerShell operators on strings:
"Hello" + " " + "World"          # concatenation
"Ha" * 3                         # "HaHaHa"
"hello" -eq "HELLO"              # $false (case-sensitive)
"hello" -ieq "HELLO"             # $true  (case-INsensitive)
"hello" -ceq "hello"             # $true  (case-sensitive explicit)
"PowerShell" -like "Power*"      # $true  (wildcard)
"PowerShell" -notlike "Java*"    # $true
"abc123" -match "^\w+\d+$"       # $true  (regex)
"abc123" -notmatch "^\d+$"       # $true
$Matches                         # auto-populated with -match groups

# String formatting:
"Name: {0}, Age: {1}" -f "Alice", 30   # "Name: Alice, Age: 30"
[string]::Format("Pi = {0:F2}", 3.14159) # "Pi = 3.14"

# Subexpression inside double-quoted strings:
"Today is $(Get-Date -Format 'yyyy-MM-dd')"
"Result: $(2 + 2)"               # "Result: 4"
"Array: $($arr -join ', ')"      # force array to expand properly

# Join and Split:
$arr -join ", "                  # "1, 2, 3, 4, 5"
"a,b,c" -split ","               # @("a","b","c")
"a1b2c3" -split "\d"             # split on digit regex
```

---

## Operators

```powershell
# Arithmetic:
+   -   *   /   %              # addition, subtraction, multiply, divide, modulo

# Assignment:
=   +=  -=  *=  /=  %=  ++  --

# Comparison (case-insensitive by default for strings):
-eq   -ne   -lt   -gt   -le   -ge   # equal, not-equal, less, greater, ...
-ieq  -ine  -ilt  -igt  -ile  -ige  # explicit case-insensitive
-ceq  -cne  -clt  -cgt  -cle  -cge  # explicit case-sensitive

# Logical:
-and   -or   -not   -xor
!      # alias for -not

# String / Collection:
-like     -notlike              # wildcard (* and ?)
-match    -notmatch             # regex match (populates $Matches)
-cmatch   -cnotmatch            # case-sensitive regex
-contains -notcontains          # collection contains value
-in       -notin                # value in collection
-replace                        # regex replace
"text" -replace "e","3"         # "t3xt"

# Type operators:
-is    -isnot                   # type check
42 -is [int]                    # $true
"hi" -is [string]               # $true
$obj -isnot [System.IO.FileInfo] # $true

# Bitwise:
-band   -bor   -bxor   -bnot   -shl   -shr

# Pipeline operators (PS 7+):
||    # pipeline chain: run right side if left fails
&&    # pipeline chain: run right side if left succeeds
?     # alias for Where-Object in pipelines
```

---

## Control Flow

```powershell
# if / elseif / else:
if ($age -ge 18) {
    Write-Host "Adult"
} elseif ($age -ge 13) {
    Write-Host "Teen"
} else {
    Write-Host "Child"
}

# switch statement (more powerful than most languages):
switch ($day) {
    "Monday"  { Write-Host "Start of week" }
    "Friday"  { Write-Host "End of week" }
    "Saturday" { Write-Host "Weekend"; break }
    "Sunday"   { Write-Host "Weekend"; break }
    default    { Write-Host "Midweek" }
}

# switch with -Wildcard, -Regex, -CaseSensitive, -Exact:
switch -Wildcard ($filename) {
    "*.txt"  { "Text file" }
    "*.log"  { "Log file" }
    "*.ps1"  { "PowerShell script" }
}

switch -Regex ($input) {
    "^\d+$"   { "All digits" }
    "^[a-z]+$"{ "All lowercase" }
}

# switch on array (processes each element):
switch (1, 2, 3) {
    1 { "One" }
    2 { "Two" }
    3 { "Three" }
}

# Ternary operator (PS 7+):
$result = $age -ge 18 ? "Adult" : "Minor"

# Null coalescing (PS 7+):
$value = $possiblyNull ?? "default"
$x ??= "default"    # assign only if $x is null

# for loop:
for ($i = 0; $i -lt 10; $i++) {
    Write-Host $i
}

# foreach loop:
foreach ($item in $collection) {
    Write-Host $item
}

# while loop:
while ($condition) {
    # body
}

# do-while / do-until:
do {
    $input = Read-Host "Enter a number"
} while ($input -notmatch "^\d+$")

do {
    $input = Read-Host "Enter a number"
} until ($input -match "^\d+$")   # runs until condition is TRUE

# Loop control:
break       # exit loop
continue    # skip to next iteration
return      # exit function (or script at top level)

# ForEach-Object (pipeline-friendly):
1..10 | ForEach-Object { $_ * 2 }
1..10 | ForEach-Object -Begin { "Start" } -Process { $_ } -End { "End" }

# Where-Object (filter):
Get-Process | Where-Object { $_.CPU -gt 100 }
Get-Process | Where-Object CPU -gt 100    # simplified syntax (PS 3+)
```

---

## Functions

```powershell
# Basic function:
function Greet {
    param($Name)
    "Hello, $Name!"
}

Greet -Name "Alice"
Greet "Alice"          # positional — works too

# Advanced function (full parameter declaration):
function Get-UserInfo {
    [CmdletBinding()]                    # enables -Verbose, -Debug, -WhatIf, etc.
    param(
        [Parameter(Mandatory, Position=0, HelpMessage="Enter username")]
        [string]$Username,

        [Parameter()]
        [ValidateRange(1, 150)]
        [int]$Age = 25,                  # default value

        [Parameter()]
        [ValidateSet("Admin","User","Guest")]
        [string]$Role = "User",

        [Parameter()]
        [ValidatePattern("^\d{4}-\d{2}-\d{2}$")]
        [string]$DOB,

        [Parameter(ValueFromPipeline)]   # accepts pipeline input
        [string]$PipelineInput,

        [switch]$Verbose2                # switch parameter (flag, no value)
    )

    begin   { Write-Verbose "Starting..." }    # runs once before pipeline
    process { Write-Verbose "Processing $_" }  # runs once per pipeline object
    end     { Write-Verbose "Done" }           # runs once after pipeline
}

# Return values:
function Add-Numbers {
    param([int]$A, [int]$B)
    return $A + $B      # explicit return
    # OR just write the value — PowerShell returns all uncaptured output:
    $A + $B
}

$result = Add-Numbers 3 5    # $result = 8

# Multiple return values (returns array):
function Get-MinMax {
    param([int[]]$Numbers)
    return ($Numbers | Measure-Object -Min -Max | Select-Object Minimum, Maximum)
}

# Validation attributes:
[ValidateNotNull()]
[ValidateNotNullOrEmpty()]
[ValidateLength(3, 20)]
[ValidateRange(0, 100)]
[ValidateSet("A", "B", "C")]
[ValidatePattern("regex")]
[ValidateScript({ $_ -gt 0 })]    # custom script block

# Splatting (pass hashtable as parameters):
$params = @{
    Path    = "C:\Logs"
    Recurse = $true
    Filter  = "*.log"
}
Get-ChildItem @params   # equivalent to: Get-ChildItem -Path "C:\Logs" -Recurse -Filter "*.log"
```

---

## The Pipeline

The pipeline is PowerShell's most distinctive feature. **Objects** flow between commands — not text.

```powershell
# Basic pipeline:
Get-Process | Stop-Process

# Filter → select → sort → format:
Get-Process |
    Where-Object { $_.WorkingSet -gt 100MB } |
    Select-Object Name, CPU, WorkingSet |
    Sort-Object WorkingSet -Descending |
    Format-Table -AutoSize

# ForEach-Object (transform each object):
Get-ChildItem *.txt | ForEach-Object { $_.Name.ToUpper() }
1..5 | ForEach-Object { $_ * $_ }   # squares: 1,4,9,16,25

# Select-Object (choose/rename properties):
Get-Process | Select-Object Name, CPU, @{N="RAM_MB"; E={[math]::Round($_.WorkingSet/1MB,1)}}
Get-Process | Select-Object -First 5    # top 5
Get-Process | Select-Object -Last 5     # bottom 5
Get-Process | Select-Object -Skip 10   # skip first 10
Get-Process | Select-Object -Unique     # deduplicate
Get-Process | Select-Object -ExpandProperty Name  # unwrap property to raw values

# Where-Object (filter objects):
Get-Service | Where-Object Status -eq "Running"
Get-Service | Where-Object { $_.Status -eq "Running" -and $_.Name -like "W*" }

# Sort-Object:
Get-Process | Sort-Object CPU                   # ascending
Get-Process | Sort-Object CPU -Descending
Get-Process | Sort-Object Name, CPU             # multi-key sort
Get-Process | Sort-Object CPU -Unique           # deduplicate

# Group-Object (group by property):
Get-Process | Group-Object -Property Company
Get-Service | Group-Object Status | Select-Object Name, Count

# Measure-Object (statistics):
Get-Process | Measure-Object WorkingSet -Sum -Average -Min -Max
1..100 | Measure-Object -Sum -Average

# Tee-Object (split pipeline to file AND continue):
Get-Process | Tee-Object -FilePath processes.txt | Where-Object CPU -gt 50

# Out-* cmdlets (terminal output):
Out-Host         # default — display to console
Out-Null         # discard output (like > /dev/null)
Out-File         # write to file (text)
Out-String       # convert to string
Out-GridView     # interactive GUI grid (Windows)
Out-Printer      # send to printer

# Export cmdlets:
Get-Process | Export-Csv processes.csv -NoTypeInformation
Get-Process | Export-Clixml processes.xml    # preserves full object fidelity
Get-Process | ConvertTo-Json | Out-File data.json
Get-Process | ConvertTo-Html | Out-File report.html

# Import cmdlets:
Import-Csv data.csv
Import-Clixml data.xml
Get-Content data.json | ConvertFrom-Json
```

---

## Error Handling

```powershell
# Error action preferences:
$ErrorActionPreference = "Stop"       # make all errors terminating (script-wide)
$ErrorActionPreference = "Continue"   # default: show error and continue
$ErrorActionPreference = "SilentlyContinue" # suppress errors
$ErrorActionPreference = "Inquire"    # ask user what to do

# Per-command error action:
Get-Item "missing.txt" -ErrorAction Stop
Get-Item "missing.txt" -ErrorAction SilentlyContinue
Get-Item "missing.txt" -ErrorAction Ignore    # no error variable entry

# try / catch / finally:
try {
    $result = 1 / 0
    Get-Item "C:\doesnotexist" -ErrorAction Stop
}
catch [System.DivideByZeroException] {
    Write-Host "Division by zero!"
}
catch [System.IO.FileNotFoundException] {
    Write-Host "File not found: $($_.Exception.Message)"
}
catch {
    Write-Host "Unexpected error: $($_.Exception.Message)"
    Write-Host "Stack trace: $($_.ScriptStackTrace)"
}
finally {
    Write-Host "Always runs — cleanup here"
}

# $_ in catch block is the ErrorRecord:
catch {
    $_.Exception.Message        # error message
    $_.Exception.GetType()      # exception type
    $_.InvocationInfo.Line      # line that caused the error
    $_.ScriptStackTrace         # full stack trace
    $_.CategoryInfo             # error category
    $_.FullyQualifiedErrorId    # unique error ID
}

# throw custom errors:
throw "Something went wrong"
throw [System.ArgumentException]::new("Invalid argument")

# -ErrorVariable: capture errors into a variable:
Get-Item "missing" -ErrorAction SilentlyContinue -ErrorVariable myErr
if ($myErr) { Write-Host "Error captured: $myErr" }

# Trap (legacy — prefer try/catch):
trap { Write-Host "Error: $_"; continue }
```

---

## Working with Files & the Filesystem

```powershell
# Navigation:
Set-Location C:\Users        # cd
Push-Location C:\Temp        # cd but remember previous location
Pop-Location                 # return to previous location
Get-Location                 # pwd — current directory

# List files:
Get-ChildItem                # ls / dir
Get-ChildItem -Recurse       # recursive
Get-ChildItem -Filter "*.log"
Get-ChildItem -File          # files only
Get-ChildItem -Directory     # directories only
Get-ChildItem -Hidden        # include hidden items
Get-ChildItem -Depth 2       # limit recursion depth

# File operations:
New-Item file.txt -ItemType File
New-Item MyDir  -ItemType Directory
Copy-Item source.txt dest.txt
Copy-Item C:\Src -Destination C:\Dst -Recurse
Move-Item old.txt new.txt
Rename-Item file.txt renamed.txt
Remove-Item file.txt
Remove-Item C:\Dir -Recurse -Force
Test-Path C:\file.txt        # $true / $false

# Read / write files:
Get-Content file.txt                    # read as array of lines
Get-Content file.txt -Raw              # read as single string
Get-Content large.log -Tail 20         # last 20 lines
Get-Content growing.log -Wait          # tail -f equivalent
Set-Content file.txt "Hello"           # overwrite
Add-Content file.txt "New line"        # append
Out-File file.txt                      # pipeline → file (text)
[System.IO.File]::WriteAllText("f.txt","data")  # .NET direct write

# File info:
$f = Get-Item "file.txt"
$f.Name          $f.FullName      $f.Extension
$f.Length        $f.LastWriteTime $f.CreationTime
$f.Attributes                     # Hidden, ReadOnly, etc.

# Paths:
Join-Path "C:\Users" "Alice" "Documents"   # safe path joining
Split-Path "C:\Foo\bar.txt" -Leaf          # "bar.txt"
Split-Path "C:\Foo\bar.txt" -Parent        # "C:\Foo"
[System.IO.Path]::GetExtension("file.txt") # ".txt"
[System.IO.Path]::GetTempPath()            # temp folder
Resolve-Path ".\relative\path"             # absolute path

# Drives:
Get-PSDrive                   # all drives (filesystem, registry, cert, env...)
New-PSDrive -Name X -PSProvider FileSystem -Root \\server\share
```

---

## Processes & Services

```powershell
# Processes:
Get-Process
Get-Process -Name "chrome"
Get-Process -Id 1234
Start-Process notepad
Start-Process "cmd.exe" -ArgumentList "/c dir" -Wait -NoNewWindow
Stop-Process -Name "notepad"
Stop-Process -Id 1234 -Force
Wait-Process -Name "setup" -Timeout 60
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Services:
Get-Service
Get-Service -Name "wuauserv"
Start-Service   "wuauserv"
Stop-Service    "wuauserv"
Restart-Service "wuauserv"
Suspend-Service "wuauserv"
Resume-Service  "wuauserv"
Set-Service "wuauserv" -StartupType Automatic   # Automatic, Manual, Disabled

# Scheduled tasks (Windows):
Get-ScheduledTask
Get-ScheduledTask -TaskName "MyTask"
Start-ScheduledTask -TaskName "MyTask"
```

---

## Registry (Windows)

```powershell
# Registry is a PSDrive — navigate like filesystem:
Set-Location HKLM:\SOFTWARE\Microsoft

Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion -Name ProgramFilesDir
Set-ItemProperty HKLM:\...\Run -Name "MyApp" -Value "C:\MyApp.exe"
New-ItemProperty HKLM:\...\Run -Name "MyApp" -Value "C:\app.exe" -PropertyType String
Remove-ItemProperty HKLM:\...\Run -Name "MyApp"
New-Item HKLM:\SOFTWARE\MyCompany
Remove-Item HKLM:\SOFTWARE\MyCompany -Recurse
Test-Path HKLM:\SOFTWARE\MyCompany
```

---

## Remote Management

```powershell
# One-to-one interactive remoting:
Enter-PSSession -ComputerName Server01
Enter-PSSession -ComputerName Server01 -Credential (Get-Credential)
Exit-PSSession

# One-to-many (fan-out):
Invoke-Command -ComputerName Server01, Server02 -ScriptBlock {
    Get-Service -Name "wuauserv"
}

# Run script file remotely:
Invoke-Command -ComputerName Server01 -FilePath C:\Scripts\Deploy.ps1

# Persistent sessions:
$session = New-PSSession -ComputerName Server01
Invoke-Command -Session $session -ScriptBlock { $env:COMPUTERNAME }
Enter-PSSession -Session $session
Remove-PSSession $session

# Copy files via PS remoting:
Copy-Item C:\local\file.txt -Destination C:\remote\ -ToSession $session
Copy-Item C:\remote\file.txt -Destination C:\local\ -FromSession $session

# SSH remoting (PS 7+, cross-platform):
Enter-PSSession -HostName ubuntu-server -UserName alice -SSHTransport
Invoke-Command -HostName ubuntu-server -UserName alice -ScriptBlock { uname -a }

# WinRM configuration:
Enable-PSRemoting -Force      # enable on target machine
Test-WSMan -ComputerName Server01
```

---

## Jobs & Background Tasks

```powershell
# Background jobs (run in separate process):
$job = Start-Job -ScriptBlock { Get-Process | Measure-Object }
$job = Start-Job -FilePath C:\script.ps1
Get-Job                        # list all jobs
Get-Job -Id 3                  # specific job
Wait-Job -Id 3                 # block until job finishes
Wait-Job -Id 3 -Timeout 30     # wait max 30 seconds
Receive-Job -Id 3              # get output (clears buffer)
Receive-Job -Id 3 -Keep        # get output (keeps in buffer)
Stop-Job -Id 3
Remove-Job -Id 3
Remove-Job *                   # clean up all jobs

# Thread jobs (faster — same process, uses runspaces):
$job = Start-ThreadJob -ScriptBlock { Start-Sleep 5; "done" }

# Parallel ForEach (PS 7+, most efficient):
1..10 | ForEach-Object -Parallel {
    Start-Sleep 1
    "Processed: $_"
} -ThrottleLimit 5             # max 5 threads at once

# Workflow-style parallel (PS 5):
foreach -Parallel ($item in $items) { ... }  # requires Workflow
```

---

## Modules

```powershell
# Find and install modules:
Find-Module -Name "Pester"
Install-Module -Name "Pester" -Scope CurrentUser
Update-Module -Name "Pester"
Uninstall-Module -Name "Pester"

# Use modules:
Import-Module ActiveDirectory
Import-Module .\MyModule.psm1        # from local file
Get-Module                           # list loaded modules
Get-Module -ListAvailable            # all installed modules
Remove-Module MyModule               # unload

# Module structure:
# MyModule/
# ├── MyModule.psd1   (manifest — metadata, version, dependencies)
# └── MyModule.psm1   (script module — functions, variables)

# MyModule.psm1:
function Get-Something { ... }
function Set-Something { ... }
Export-ModuleMember -Function Get-Something, Set-Something
# -Variable, -Alias also available

# MyModule.psd1 (manifest):
New-ModuleManifest -Path MyModule.psd1 `
    -RootModule MyModule.psm1 `
    -ModuleVersion "1.0.0" `
    -Author "Alice" `
    -Description "My module"

# Module auto-loading:
# Place module folder in $env:PSModulePath → auto-imported on first use
$env:PSModulePath   # see search paths
```

---

## Classes (PS 5+)

```powershell
class Animal {
    # Properties:
    [string]$Name
    [string]$Sound
    [int]hidden $Age       # hidden = not shown by default

    # Constructor:
    Animal([string]$name, [string]$sound) {
        $this.Name  = $name
        $this.Sound = $sound
    }

    # Method:
    [string] Speak() {
        return "$($this.Name) says $($this.Sound)"
    }

    # Static method:
    static [string] Kingdom() {
        return "Animalia"
    }
}

# Inheritance:
class Dog : Animal {
    [string]$Breed

    Dog([string]$name, [string]$breed) : base($name, "Woof") {
        $this.Breed = $breed
    }

    [string] Fetch([string]$item) {
        return "$($this.Name) fetches the $item"
    }
}

# Usage:
$cat = [Animal]::new("Whiskers", "Meow")
$cat.Speak()                    # "Whiskers says Meow"
[Animal]::Kingdom()             # "Animalia" (static)

$dog = [Dog]::new("Rex", "Labrador")
$dog.Speak()                    # "Rex says Woof" (inherited)
$dog.Fetch("ball")              # "Rex fetches the ball"
$dog -is [Animal]               # $true (inheritance check)

# Interfaces (PS 5+):
class MyWriter : System.IO.TextWriter {
    [System.Text.Encoding]$Encoding = [System.Text.Encoding]::UTF8
    [void] Write([char]$value) { Write-Host $value -NoNewline }
}

# Enum:
enum Direction { North; South; East; West }
[Direction]::North
$d = [Direction]"East"
```

---

## Working with .NET

PowerShell sits on .NET — you can use any .NET class directly.

```powershell
# Access .NET types with [TypeName]:
[System.DateTime]::Now
[System.DateTime]::UtcNow
[System.DateTime]::Parse("2025-01-01")
[System.Environment]::MachineName
[System.Environment]::GetEnvironmentVariable("PATH")
[System.Math]::Round(3.14159, 2)
[System.Math]::Pow(2, 10)
[System.Net.Dns]::GetHostEntry("google.com")

# Create .NET objects:
$sb   = [System.Text.StringBuilder]::new()
$sb.Append("Hello")
$sb.Append(", World")
$sb.ToString()

$web  = [System.Net.WebClient]::new()
$html = $web.DownloadString("https://example.com")

$list = [System.Collections.Generic.List[string]]::new()
$list.Add("Alice")
$list.Add("Bob")
$list.Contains("Alice")   # $true
$list.Sort()

$dict = [System.Collections.Generic.Dictionary[string,int]]::new()
$dict["Alice"] = 30
$dict["Bob"]   = 25

# Load assemblies:
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName PresentationFramework   # WPF

# Compile inline C#:
Add-Type -TypeDefinition @"
public class Greeter {
    public static string Hello(string name) {
        return "Hello, " + name + "!";
    }
}
"@
[Greeter]::Hello("PowerShell")   # "Hello, PowerShell!"
```

---

## Regular Expressions

```powershell
# -match operator (case-insensitive by default):
"hello123" -match "\d+"         # $true; $Matches[0] = "123"
"hello123" -match "(\w+?)(\d+)" # $Matches[1]="hello" $Matches[2]="123"

# -replace (regex replace):
"hello world" -replace "\s+", "_"     # "hello_world"
"2025-03-06"  -replace "(\d+)-(\d+)-(\d+)", '$3/$2/$1'  # "06/03/2025"

# -split (regex split):
"a1b2c3" -split "\d"           # @("a","b","c","")
"CSV,data, with spaces" -split ",\s*"  # @("CSV","data","with spaces")

# [regex] class for advanced use:
$rx = [regex]"(\d{4})-(\d{2})-(\d{2})"
$m  = $rx.Match("Date: 2025-03-06")
$m.Success                     # $true
$m.Groups[1].Value             # "2025"

# All matches:
$rx.Matches("2025-01-01 and 2025-12-31") | ForEach-Object { $_.Value }

# Replace with delegate:
$rx.Replace("hello world", { $args[0].Value.ToUpper() })

# Named groups:
"2025-03-06" -match "(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})"
$Matches.year    # "2025"
$Matches.month   # "03"
```

---

## JSON, XML & CSV

```powershell
# JSON:
$json = '{"name":"Alice","age":30,"skills":["PS","C#"]}'
$obj  = $json | ConvertFrom-Json
$obj.name            # "Alice"
$obj.skills[0]       # "PS"

$obj | ConvertTo-Json                    # back to JSON string
$obj | ConvertTo-Json -Depth 10          # handle deep nesting
$obj | ConvertTo-Json -Compress          # minified

# File-based:
Get-Content data.json | ConvertFrom-Json
$data | ConvertTo-Json | Set-Content output.json

# REST API:
$response = Invoke-RestMethod -Uri "https://api.example.com/users" -Method GET
$response.users | Select-Object name, email

Invoke-RestMethod -Uri "https://api.example.com/users" `
    -Method POST `
    -Body (@{name="Alice"; email="a@b.com"} | ConvertTo-Json) `
    -ContentType "application/json" `
    -Headers @{Authorization = "Bearer $token"}

# XML:
[xml]$xml = Get-Content config.xml
$xml.configuration.appSettings.add | Where-Object key -eq "timeout"
$xml.configuration.appSettings.add[0].value = "60"
$xml.Save("config.xml")

# CSV:
Import-Csv employees.csv | Where-Object Department -eq "IT"
Get-Process | Export-Csv processes.csv -NoTypeInformation -Delimiter ";"
$data = @(
    [PSCustomObject]@{Name="Alice"; Age=30}
    [PSCustomObject]@{Name="Bob";   Age=25}
)
$data | Export-Csv output.csv -NoTypeInformation
```

---

## Useful One-Liners & Patterns

```powershell
# System info:
Get-ComputerInfo | Select-Object CsName, WindowsVersion, TotalPhysicalMemory
[System.Environment]::OSVersion

# Disk usage:
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free

# Find large files:
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue |
    Sort-Object Length -Descending | Select-Object -First 10 FullName, Length

# Kill process by name:
Get-Process chrome | Stop-Process -Force

# Check if port is open:
Test-NetConnection -ComputerName google.com -Port 443

# Get public IP:
(Invoke-RestMethod "https://api.ipify.org?format=json").ip

# Download a file:
Invoke-WebRequest -Uri "https://example.com/file.zip" -OutFile "file.zip"

# Base64 encode/decode:
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("Hello"))
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("SGVsbG8="))

# Generate random password:
-join ((65..90) + (97..122) + (48..57) | Get-Random -Count 16 | ForEach-Object {[char]$_})

# Find text in files (like grep):
Select-String -Path "*.log" -Pattern "ERROR" -CaseSensitive

# Zip / Unzip:
Compress-Archive -Path C:\Folder -DestinationPath archive.zip
Expand-Archive -Path archive.zip -DestinationPath C:\Output

# Get event logs:
Get-EventLog -LogName System -Newest 50 -EntryType Error
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 20

# Measure script execution time:
Measure-Command { Get-ChildItem C:\ -Recurse }

# Hash a file:
Get-FileHash file.txt -Algorithm SHA256
Get-FileHash file.txt -Algorithm MD5

# WMI / CIM queries:
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores
Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, Size, FreeSpace
```

---

## Scripting Best Practices

```powershell
# Script header — always include:
#Requires -Version 7.0
#Requires -Modules ActiveDirectory
#Requires -RunAsAdministrator

<#
.SYNOPSIS
    Brief one-line description.
.DESCRIPTION
    Longer description.
.PARAMETER Name
    Description of the Name parameter.
.EXAMPLE
    .\MyScript.ps1 -Name "Alice"
.NOTES
    Author: Alice
    Date:   2025-03-06
#>

[CmdletBinding(SupportsShouldProcess)]   # adds -WhatIf and -Confirm support
param(
    [Parameter(Mandatory)]
    [string]$Name
)

# Use -WhatIf in destructive operations:
if ($PSCmdlet.ShouldProcess($Name, "Delete user")) {
    Remove-ADUser -Identity $Name
}

# Verbose and debug output:
Write-Verbose "Processing $Name"    # shown with -Verbose
Write-Debug "Value is $x"          # shown with -Debug
Write-Warning "This might fail"
Write-Error "Something is wrong"
Write-Information "Info message" -InformationAction Continue

# Logging pattern:
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "$timestamp [$Level] $Message" | Add-Content "app.log"
}

# Always check $PSBoundParameters for optional params:
if ($PSBoundParameters.ContainsKey("Verbose2")) { ... }

# Use [PSCustomObject] for structured output:
[PSCustomObject]@{
    Name      = $Name
    Timestamp = Get-Date
    Success   = $true
}
```

---

## Quick Reference Card

```
SIGILS & SYNTAX
  $variable     variable          @()   array literal
  $_  / $PSItem pipeline object   @{}   hashtable literal
  $?            last success      $null null value
  $env:VAR      environment var   `     escape character / line continuation

COMPARISON OPERATORS
  -eq -ne -lt -gt -le -ge        (strings: case-insensitive)
  -ceq -cne ...                  (case-sensitive)
  -like -notlike                 (wildcard: * and ?)
  -match -notmatch               (regex; populates $Matches)
  -contains -notcontains         (collection contains value)
  -in -notin                     (value in collection)
  -is -isnot                     (type check)

PIPELINE ESSENTIALS
  Where-Object  { $_.Prop -gt X }  filter objects
  ForEach-Object { do-something }  transform each object
  Select-Object Name, CPU          choose properties
  Sort-Object CPU -Descending      sort
  Group-Object Status              group by property
  Measure-Object -Sum -Average     statistics
  Tee-Object -FilePath out.txt     split to file + continue

ERROR HANDLING
  try { } catch [ExType] { } finally { }
  $ErrorActionPreference = "Stop"
  -ErrorAction Stop / SilentlyContinue / Ignore
  $_.Exception.Message            in catch block

STRING TRICKS
  "Hello $name"                   interpolation
  "val: $(expr)"                  expression in string
  @"..."@                         here-string (with interpolation)
  @'...'@                         literal here-string
  -f operator                     "Hello {0}" -f "World"
  -join / -split                  array↔string

USEFUL CMDLETS
  Get-Help     Get-Command    Get-Member
  Get-Process  Get-Service    Get-ChildItem
  Invoke-Command              Test-NetConnection
  Invoke-RestMethod           Measure-Command
  Select-String               Get-FileHash
  Export-Csv   Import-Csv     ConvertTo-Json  ConvertFrom-Json
```
