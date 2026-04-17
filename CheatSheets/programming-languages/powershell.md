# PowerShell: A Complete Progressive Tutorial

---

## 1. What & Why

PowerShell is a cross-platform shell and scripting language built on .NET. Unlike Bash, which passes text between commands, PowerShell passes **objects** — structured data with properties and methods. This fundamental difference makes PowerShell dramatically more powerful for Windows administration and increasingly relevant for cross-platform automation.

Why PowerShell instead of Bash on Windows? Because Windows is object-oriented at its core. Registry entries, processes, services, COM objects, .NET types, WMI classes — all of these are objects. Bash treats everything as text; you'd have to parse output with regex to extract the values you need. PowerShell gives you direct access to structured data, so filtering, sorting, grouping, and acting on that data requires no parsing at all.

PowerShell 7+ runs on Linux and macOS as well, making it relevant beyond Windows contexts. For Windows-heavy environments (Active Directory, Exchange, Azure, SQL Server), PowerShell is the primary automation tool.

---

## 2. Mental Model

PowerShell's pipeline passes objects, not text. Each cmdlet in a pipeline receives the full object from the previous cmdlet, with all its properties and methods intact.

```
# Bash: everything is text — you must parse it
ps aux | grep nginx | awk '{print $1, $11}'   # text manipulation hell

# PowerShell: everything is an object with named properties
Get-Process nginx | Select-Object Id, CPU, WorkingSet

# The process object has dozens of typed properties:
$proc = Get-Process nginx
$proc.Id           # integer
$proc.CPU          # double (seconds)
$proc.StartTime    # DateTime object
$proc.WorkingSet64 # long (bytes)
$proc.Kill()       # method call — terminates the process

Pipeline flow:
  Get-Process  →  [Process objects]
       ↓
  Where-Object {$_.CPU -gt 100}  →  [filtered Process objects]
       ↓
  Select-Object Name, CPU, Id    →  [custom objects with 3 properties]
       ↓
  Sort-Object CPU -Descending    →  [sorted objects]
       ↓
  Format-Table                   →  display as table
```

All cmdlets follow the `Verb-Noun` naming convention: `Get-Process`, `Set-Content`, `Remove-Item`, `Invoke-Command`. Learn the common verbs (Get, Set, New, Remove, Add, Invoke, Start, Stop, Write, Read) and you can guess cmdlet names.

---

## 3. Progressive Examples

### Level 1: Variables, Types, and Basic Cmdlets

```powershell
# Variables always use $ prefix. No declaration keyword needed.
$name = "Alice"
$age = 30
$pi = 3.14159
$active = $true    # booleans: $true, $false (not True/False)
$nothing = $null   # null value

# Strong typing (optional but useful)
[string]$name = "Alice"
[int]$count = 42
[datetime]$today = Get-Date
[string[]]$colors = "red", "green", "blue"  # typed array

# Special variables
$PSVersionTable     # PowerShell version info
$env:PATH           # environment variables accessed via $env:
$HOME               # home directory
$_                  # current pipeline object (inside Where-Object, ForEach-Object, etc.)
$?                  # last command's success/failure ($true = success)
$LASTEXITCODE       # exit code of last native executable
$Error[0]           # most recent error object
$args               # arguments passed to a script/function

# String types
"Hello, $name!"                           # double-quoted: variables interpolated
'Hello, $name!'                           # single-quoted: literal, no interpolation
"The value is: $(2 + 2)"                 # expression in $()
@"
This is a
multi-line string with $name interpolation
"@                                         # here-string (double-quoted, multi-line)

@'
No interpolation $name
'@                                         # here-string (single-quoted, literal)

# Common string operations
"Hello".Length          # 5
"Hello".ToUpper()       # "HELLO" — method call on string object
"hello world".Split(" ")  # ["hello", "world"]
"  spaces  ".Trim()       # "spaces"
"hello" -replace "l", "L"  # "heLLo" — regex replace
"hello" -like "hel*"       # $true — wildcard match
"hello" -match "^h\w+"     # $true — regex match
```

### Level 2: The Object Pipeline

```powershell
# Get-Member: inspect any object's properties and methods
Get-Process | Get-Member
# Displays: NoteProperty, Property, Method, Event — everything available

# Select-Object: choose which properties to display or keep
Get-Process | Select-Object Name, Id, CPU, WorkingSet -First 10

# Add computed properties
Get-Process | Select-Object Name, @{
    Name = "MemoryMB"
    Expression = { [math]::Round($_.WorkingSet / 1MB, 2) }
} | Sort-Object MemoryMB -Descending

# Where-Object: filter objects based on conditions
Get-Process | Where-Object { $_.CPU -gt 5 }
Get-Process | Where-Object CPU -gt 5   # simplified syntax for single comparisons
Get-Service | Where-Object Status -eq "Running"
Get-ChildItem | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) }

# Sort-Object: sort by one or more properties
Get-Process | Sort-Object CPU -Descending
Get-ChildItem | Sort-Object LastWriteTime, Name

# Group-Object: group objects by a property value
Get-Service | Group-Object Status
# Name      Count  Group
# -------   -----  -----
# Running   189    {Appinfo, AudioEndpointBuilder, ...}
# Stopped   47     {AJRouter, ALG, ...}

# Measure-Object: compute statistics
Get-Process | Measure-Object WorkingSet -Sum -Average -Maximum -Minimum
Get-ChildItem -Recurse | Measure-Object Length -Sum

# ForEach-Object: run code for each pipeline object
Get-Service | Where-Object Status -eq "Stopped" | ForEach-Object {
    Write-Host "Starting: $($_.Name)"
    $_.Start()
}

# A complete real-world pipeline: find top 5 memory-consuming processes
Get-Process |
    Where-Object WorkingSet -gt 50MB |
    Select-Object Name, Id, @{N="MemoryMB"; E={[math]::Round($_.WorkingSet/1MB,1)}} |
    Sort-Object MemoryMB -Descending |
    Select-Object -First 5 |
    Format-Table -AutoSize
```

### Level 3: Files, Registry, and System Administration

```powershell
# === File Operations ===
# PowerShell treats the filesystem as a provider — same cmdlets work for files, registry, certs

# Navigation
Set-Location C:\Users        # cd
Set-Location ~               # home directory
Get-Location                 # pwd

# Listing
Get-ChildItem                # ls
Get-ChildItem -Recurse       # recursive
Get-ChildItem *.log -Recurse # filter by pattern
Get-ChildItem | Where-Object { -not $_.PSIsContainer }   # files only
Get-ChildItem | Where-Object PSIsContainer               # directories only

# File operations
Copy-Item source.txt dest.txt
Copy-Item src_folder -Destination dest_folder -Recurse
Move-Item old_name.txt new_name.txt
Remove-Item file.txt
Remove-Item folder -Recurse -Force    # force: no confirmation, Recurse: delete contents
New-Item -ItemType Directory -Path C:\NewFolder
New-Item -ItemType File -Path C:\file.txt

# Reading and writing files
$content = Get-Content C:\file.txt               # string array (one element per line)
$content = Get-Content C:\file.txt -Raw          # single string
Set-Content -Path C:\output.txt -Value "Hello"   # overwrite
Add-Content -Path C:\log.txt -Value "Entry"      # append

# Working with CSV, JSON, XML
$users = Import-Csv C:\users.csv
$users | Export-Csv C:\output.csv -NoTypeInformation

$config = Get-Content config.json | ConvertFrom-Json
$config.server.port = 9090
$config | ConvertTo-Json -Depth 10 | Set-Content config.json

[xml]$doc = Get-Content settings.xml
$doc.configuration.appSettings.add | Where-Object key -eq "timeout"

# === Registry ===
# HKCU: = HKEY_CURRENT_USER, HKLM: = HKEY_LOCAL_MACHINE
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
Set-ItemProperty -Path "HKCU:\Software\MyApp" -Name "Theme" -Value "Dark"
New-Item -Path "HKCU:\Software\MyApp"
Remove-Item -Path "HKCU:\Software\MyApp" -Recurse

# === Services ===
Get-Service | Where-Object Status -eq "Running" | Sort-Object DisplayName
Start-Service "wuauserv"       # Windows Update
Stop-Service "wuauserv"
Restart-Service "Spooler"      # Print Spooler
Set-Service "Spooler" -StartupType Automatic

# === Processes ===
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Get-Process -Name "chrome" | Stop-Process -Force
Start-Process "notepad.exe" -Wait   # launch and wait for exit
$proc = Start-Process "ping" -ArgumentList "google.com", "-n", "4" -PassThru
$proc.WaitForExit()
$proc.ExitCode
```

### Level 4: Functions, Modules, and Error Handling

```powershell
# Functions with full parameter support
function Get-DiskUsage {
    [CmdletBinding()]    # enables -Verbose, -Debug, -ErrorAction etc.
    param(
        [Parameter(Mandatory = $true, Position = 0)]
        [string]$Path,

        [Parameter()]
        [ValidateSet("MB", "GB")]
        [string]$Unit = "MB",

        [Parameter()]
        [switch]$Recurse   # switch: present = $true, absent = $false
    )

    $items = if ($Recurse) {
        Get-ChildItem $Path -Recurse -File
    } else {
        Get-ChildItem $Path -File
    }

    $totalBytes = ($items | Measure-Object -Property Length -Sum).Sum
    $divisor = if ($Unit -eq "MB") { 1MB } else { 1GB }

    [PSCustomObject]@{
        Path  = $Path
        Files = $items.Count
        Size  = [math]::Round($totalBytes / $divisor, 2)
        Unit  = $Unit
    }
}

# Usage:
Get-DiskUsage -Path C:\Users -Unit GB -Recurse
Get-DiskUsage C:\Temp   # positional parameter

# Pipeline-aware functions
function Format-FileSize {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline = $true)]
        [System.IO.FileInfo]$File
    )

    process {   # 'process' block runs for each pipeline object
        [PSCustomObject]@{
            Name    = $File.Name
            SizeMB  = [math]::Round($File.Length / 1MB, 2)
            LastMod = $File.LastWriteTime
        }
    }
}

Get-ChildItem C:\Temp *.log | Format-FileSize | Sort-Object SizeMB -Descending

# Error handling
function Invoke-SafeOperation {
    param([string]$Path)

    try {
        $content = Get-Content $Path -ErrorAction Stop   # throw on error
        return $content
    }
    catch [System.IO.FileNotFoundException] {
        Write-Warning "File not found: $Path"
        return $null
    }
    catch {
        Write-Error "Unexpected error: $($_.Exception.Message)"
        throw   # re-throw to caller
    }
    finally {
        Write-Verbose "Cleanup in finally block"
    }
}

# $ErrorActionPreference controls default behavior:
# Continue (default): show error, keep going
# Stop: throw exception (like -ErrorAction Stop on every cmdlet)
# SilentlyContinue: suppress error, keep going
# Inquire: ask user what to do

$ErrorActionPreference = "Stop"   # make all errors throw by default in scripts
```

### Level 5: Remoting, Jobs, and Scheduled Tasks

```powershell
# === PowerShell Remoting (PSRemoting) ===
# Run commands on remote machines

# Enable remoting (run as administrator on target machine)
Enable-PSRemoting -Force

# One-off remote command
Invoke-Command -ComputerName server01 -ScriptBlock {
    Get-Service | Where-Object Status -eq "Stopped"
}

# Pass variables to remote session
$threshold = 80
Invoke-Command -ComputerName server01 -ScriptBlock {
    param($t)
    Get-Process | Where-Object CPU -gt $t
} -ArgumentList $threshold

# Persistent session
$session = New-PSSession -ComputerName server01
Invoke-Command -Session $session -ScriptBlock { Get-Date }
Enter-PSSession $session   # interactive shell
Exit-PSSession
Remove-PSSession $session

# Multiple machines in parallel
$servers = "server01", "server02", "server03"
Invoke-Command -ComputerName $servers -ScriptBlock {
    [PSCustomObject]@{
        Server = $env:COMPUTERNAME
        CPU    = (Get-Process | Measure-Object CPU -Sum).Sum
    }
}

# === Background Jobs ===
# Start a long-running command in the background
$job = Start-Job -ScriptBlock {
    Start-Sleep 30
    Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
}

Get-Job                           # list all jobs
Wait-Job $job                     # block until job completes
Receive-Job $job                  # get the output
Remove-Job $job

# Parallel execution (PowerShell 7+)
$servers | ForEach-Object -Parallel {
    Test-Connection $_ -Count 1 -Quiet
} -ThrottleLimit 10   # max 10 concurrent threads

# === Scheduled Tasks ===
# Create a task that runs a script every day at 6 AM
$action = New-ScheduledTaskAction -Execute "pwsh.exe" `
    -Argument "-NonInteractive -File C:\Scripts\backup.ps1"
$trigger = New-ScheduledTaskTrigger -Daily -At "06:00"
$settings = New-ScheduledTaskSettingsSet -RunOnlyIfNetworkAvailable
Register-ScheduledTask -TaskName "DailyBackup" `
    -Action $action -Trigger $trigger -Settings $settings
```

### Level 6: Production Script Template

```powershell
#Requires -Version 7.0
#Requires -RunAsAdministrator

<#
.SYNOPSIS
    Deploys an application to a list of servers.

.DESCRIPTION
    Copies binaries, updates configuration, restarts services.

.PARAMETER Servers
    Comma-separated list of server hostnames.

.PARAMETER Environment
    Target environment: dev, staging, prod.

.PARAMETER DryRun
    Show what would happen without doing it.

.EXAMPLE
    .\deploy.ps1 -Servers "web01,web02" -Environment prod
#>
[CmdletBinding(SupportsShouldProcess)]   # adds -WhatIf and -Confirm support
param(
    [Parameter(Mandatory)]
    [string[]]$Servers,

    [Parameter(Mandatory)]
    [ValidateSet("dev", "staging", "prod")]
    [string]$Environment,

    [switch]$DryRun
)

Set-StrictMode -Version Latest          # error on uninitialized variables
$ErrorActionPreference = "Stop"         # throw on errors

# Logging
$LogFile = "C:\Logs\deploy-$(Get-Date -Format 'yyyyMMdd-HHmmss').log"

function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $entry = "[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] [$Level] $Message"
    Add-Content -Path $LogFile -Value $entry
    switch ($Level) {
        "ERROR"   { Write-Error $entry }
        "WARNING" { Write-Warning $entry }
        default   { Write-Host $entry -ForegroundColor Cyan }
    }
}

function Deploy-ToServer {
    param([string]$Server)

    Write-Log "Starting deploy to $Server"

    if ($DryRun) {
        Write-Log "[DRY RUN] Would deploy to $Server" "WARNING"
        return
    }

    try {
        $session = New-PSSession -ComputerName $Server

        Invoke-Command -Session $session -ScriptBlock {
            param($env)
            Stop-Service "MyApp" -ErrorAction SilentlyContinue
            Write-Host "Service stopped"
        } -ArgumentList $Environment

        # Copy files
        Copy-Item -Path ".\dist\*" -Destination "\\$Server\c$\apps\myapp\" `
            -Recurse -Force

        Invoke-Command -Session $session -ScriptBlock {
            Start-Service "MyApp"
            Start-Sleep 5
            $svc = Get-Service "MyApp"
            if ($svc.Status -ne "Running") {
                throw "Service failed to start"
            }
        }

        Write-Log "Deploy to $Server completed successfully"
    }
    catch {
        Write-Log "Deploy to $Server FAILED: $($_.Exception.Message)" "ERROR"
        throw
    }
    finally {
        Remove-PSSession $session -ErrorAction SilentlyContinue
    }
}

# Main execution
Write-Log "Starting deployment to $Environment environment"
Write-Log "Target servers: $($Servers -join ', ')"

$results = @()
foreach ($server in $Servers) {
    try {
        Deploy-ToServer -Server $server
        $results += [PSCustomObject]@{ Server = $server; Status = "Success" }
    }
    catch {
        $results += [PSCustomObject]@{ Server = $server; Status = "Failed" }
    }
}

$results | Format-Table -AutoSize
$failed = $results | Where-Object Status -eq "Failed"
if ($failed) {
    Write-Log "DEPLOYMENT FAILED on: $($failed.Server -join ', ')" "ERROR"
    exit 1
}
Write-Log "All servers deployed successfully"
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Comparing objects with `==` instead of `-eq`**

```powershell
# WRONG: = is assignment, == doesn't exist in PowerShell
if ($status == "Running") { ... }   # SyntaxError

# CORRECT: use comparison operators
if ($status -eq "Running") { ... }    # -eq: equal (case-insensitive for strings)
if ($count -gt 5) { ... }            # -gt: greater than
if ($name -like "Al*") { ... }        # -like: wildcard match
if ($text -match "^\d+$") { ... }     # -match: regex match
```

**Mistake 2: Forgetting that PowerShell strings are case-insensitive by default**

```powershell
"Hello" -eq "hello"    # $true — case-insensitive by default
"Hello" -ceq "hello"   # $false — c prefix = case-sensitive (-ceq, -clike, -cmatch)
"RUNNING" -eq "Running"  # $true — this affects your service status checks
```

**Mistake 3: Mishandling arrays with one element**

```powershell
$items = Get-ChildItem -Filter "*.log" | Select-Object -First 1
# If only one file found, $items might be a FileInfo object, not an array!
$items.Count  # may not work as expected

# Force single results to be arrays with @()
$items = @(Get-ChildItem -Filter "*.log" | Select-Object -First 1)
$items.Count  # always an integer (0 or 1)
```

**Mistake 4: Not using `-ErrorAction Stop` in scripts**

```powershell
# WRONG: without -ErrorAction Stop, errors are displayed but execution continues
Copy-Item "C:\missing_file.txt" "C:\dest\"   # error shown, script continues!
# Next line runs even though the copy failed

# CORRECT: use -ErrorAction Stop or set $ErrorActionPreference
$ErrorActionPreference = "Stop"   # at top of script
# OR per-cmdlet:
Copy-Item "C:\missing_file.txt" "C:\dest\" -ErrorAction Stop
```

---

## 5. The "Why Does This Work" Layer

### Why Objects Beat Text

In Bash, `ps aux` returns text. To get just the CPU usage of nginx, you write: `ps aux | grep nginx | awk '{print $3}'`. You're betting that nginx appears in the output, that the CPU column is always column 3, and that no other process name contains "nginx." These assumptions can silently break.

In PowerShell, `Get-Process nginx` returns process objects. You access `$proc.CPU` — a typed double property. It's always the right value regardless of formatting, always the right type, and raises a clear error if the process doesn't exist. The data is self-describing.

This becomes more powerful at scale: when you retrieve Active Directory users, Exchange mailboxes, or Azure resources as objects, you can filter, sort, group, and transform them with the same pipeline operators — no format-specific parsing required.

### How the Pipeline Passes Objects

When you write `Get-Process | Sort-Object CPU`, PowerShell doesn't wait for `Get-Process` to finish before starting `Sort-Object`. It runs them simultaneously using a producer-consumer model: `Get-Process` emits objects one at a time into the pipeline buffer, and `Sort-Object` processes them as they arrive. For most cmdlets, this streaming behavior means the pipeline is memory-efficient even for large data sets.

The exception is any cmdlet that must see all objects before producing output — like `Sort-Object` (can't sort without seeing everything) and `Group-Object`. These buffer all input before emitting output.

---

## 6. Quick Reference

### Comparison Operators

| Operator | Meaning | Case-Sensitive |
|----------|---------|----------------|
| `-eq` | Equal | `-ceq` |
| `-ne` | Not equal | `-cne` |
| `-gt` / `-lt` | Greater / less than | `-cgt` / `-clt` |
| `-ge` / `-le` | Greater or equal / less or equal | `-cge` / `-cle` |
| `-like` | Wildcard match (`*`, `?`) | `-clike` |
| `-notlike` | Wildcard no-match | `-cnotlike` |
| `-match` | Regex match | `-cmatch` |
| `-contains` | Array contains value | |
| `-in` | Value is in array | |

### Essential Pipeline Cmdlets

```powershell
Where-Object { $_.Property -eq "value" }  # filter
Select-Object Name, CPU, @{N="x";E={...}} # choose/compute properties
Sort-Object Property -Descending          # sort
Group-Object Property                     # group
Measure-Object -Property Size -Sum        # aggregate statistics
ForEach-Object { ... }                    # run code per object
Format-Table -AutoSize                    # display as table
Format-List                               # display as property list
Out-GridView                              # interactive GUI grid (Windows)
Export-Csv -NoTypeInformation             # save to CSV
ConvertTo-Json                            # serialize to JSON
```

### Common Verbs

| Verb | Purpose |
|------|---------|
| `Get` | Retrieve data |
| `Set` | Modify existing |
| `New` | Create |
| `Remove` | Delete |
| `Add` | Append to collection |
| `Start` / `Stop` | Begin / end a process or service |
| `Invoke` | Execute a command or script block |
| `Test` | Return $true/$false about a condition |
| `Write` | Output to host/stream |
| `Read` | Read input |
