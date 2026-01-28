# Windows Terminal Quirks

Differences between Git Bash, PowerShell, and cmd.exe that affect Claude Code and build scripts.

## Git Bash

Claude Code uses Git Bash internally as its shell on Windows. This provides Unix-like commands but introduces some quirks when interacting with Windows tools.

### Forward Slash Escaping

Git Bash interprets `/` as a path prefix and attempts to convert it. To pass literal flags to Windows tools, use `//`:

```bash
# Wrong - Git Bash converts /f to a path like C:/Program Files/Git/f
nmake /f Makefile.win

# Correct - passes literal /f to nmake
nmake //f Makefile.win

# Same applies to other flags
cl //W4 //O2 file.c
findstr //R //C:"pattern" file.txt
```

### Null Device

Git Bash doesn't recognize `nul` as the Windows null device:

```bash
# Wrong - creates a file literally named 'nul'
some_command >nul 2>&1

# Correct in Git Bash
some_command >/dev/null 2>&1
```

**Note:** Inside `.bat` files (which run in cmd.exe), use `>nul`. The `/dev/null` form is only for commands typed directly in Git Bash.

### Path Conversion

Git Bash automatically converts paths in many contexts:

```bash
# These are equivalent in Git Bash:
/c/Users/name/file.txt
C:/Users/name/file.txt

# But Windows tools may need backslashes:
cmd //c 'C:\Users\name\file.txt'
```

### Invoking Batch Files

```bash
# Use cmd //c with the batch file path
cmd //c 'C:\path\to\script.bat'

# With arguments
cmd //c 'C:\path\to\build.bat arg1 arg2'

# If the path has spaces, quotes are essential
cmd //c 'C:\Program Files\tool\run.bat'
```

### Environment Variables

Git Bash uses Unix-style environment variable syntax:

```bash
# Setting variables
export MY_VAR=value

# Reading variables
echo $MY_VAR
echo $PATH

# Windows-style doesn't work in Git Bash
echo %MY_VAR%    # Wrong - prints literal %MY_VAR%
```

## PowerShell

### Escaping Special Characters

PowerShell uses backtick (`) as escape character:

```powershell
# Escaping quotes
Write-Host "He said `"hello`""

# Escaping dollar signs (prevent variable expansion)
Write-Host "Cost: `$100"
```

### Running Batch Files

```powershell
# Direct invocation
.\build.bat

# With cmd explicitly
cmd /c build.bat

# Note: single / works in PowerShell (no Git Bash conversion)
nmake /f Makefile.win
```

### Path Handling

PowerShell handles both forward and back slashes:

```powershell
# Both work
cd C:\Users\name
cd C:/Users/name

# But some Windows tools may require backslashes
```

## cmd.exe

### Escaping Special Characters

cmd.exe uses caret (^) as escape character:

```cmd
echo He said ^"hello^"
echo 100^% complete
```

### Delayed Expansion

For variables inside loops or if blocks, use delayed expansion:

```cmd
setlocal EnableDelayedExpansion

set COUNT=0
for %%f in (*.txt) do (
    set /a COUNT+=1
    echo File !COUNT!: %%f
)
```

Without delayed expansion, `%COUNT%` would be evaluated once at parse time, not during loop execution.

### Batch File Self-Location

Get the directory containing the batch file:

```cmd
REM %~dp0 expands to drive and path of the batch file
cd /d "%~dp0"

REM %~f0 is the full path to the batch file itself
echo Running from: %~f0
```

## Cross-Shell Compatibility

### Writing Portable Scripts

When a script needs to work across shells:

1. **Prefer batch files for Windows operations** - They run consistently in cmd.exe regardless of how they're invoked

2. **Use full paths** - Avoid relying on PATH for critical tools

3. **Quote paths with spaces** - Always quote paths that might contain spaces

4. **Test from Git Bash** - If it works from Git Bash, it will work from Claude Code

### Common Patterns

**Checking if a command exists:**

```bash
# Git Bash / Unix
command -v cl >/dev/null 2>&1 && echo "found"

# Or use 'where' for Windows commands
where cl.exe >/dev/null 2>&1 && echo "found"
```

```cmd
REM cmd.exe / batch
where cl.exe >nul 2>&1
if %ERRORLEVEL% EQU 0 echo found
```

```powershell
# PowerShell
if (Get-Command cl -ErrorAction SilentlyContinue) { "found" }
```
