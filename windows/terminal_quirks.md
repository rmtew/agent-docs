# Windows Terminal Quirks

Differences between Git Bash, PowerShell, and cmd.exe that affect Claude Code and build scripts.

## Git Bash

Claude Code uses Git Bash internally as its shell on Windows. This provides Unix-like commands but introduces some quirks when interacting with Windows tools.

### MSYS Path Conversion on `/`-Prefixed Arguments

Git Bash (MSYS) converts arguments starting with `/` into Windows paths. This mangles Windows-style flags like `/f`, `/W4`, `/nologo` when calling Windows executables directly.

**Conversion rules** (tested):

| Argument | Converted to | Why |
|----------|-------------|-----|
| `/f`, `/c`, `/R` | `F:/`, `C:/`, `R:/` | Single letter → drive letter path |
| `/W4`, `/Od`, `/MP` | `C:/Program Files/Git/...` | Multi-char → Git install path |
| `/nologo`, `/link`, `/DEBUG` | `C:/Program Files/Git/...` | Same — word after `/` |
| `/D_CRT_SECURE_NO_WARNINGS` | `C:/Program Files/Git/...` | Same — any length |
| `/I.`, `/Isrc` | `C:/Program Files/Git/...` | Same |
| `/?` | `/?` | **Not converted** — special exception |
| `/std:c11`, `/Fe:bin/out.exe` | `/std:c11`, `/Fe:bin/out.exe` | **Not converted** — colon breaks the path pattern |

**Fixes** — use either:

```bash
# Fix 1: double slash — strips one /, passes the rest literally
nmake //f Makefile.win
cl //W4 //O2 file.c
findstr //R //C:"pattern" file.txt

# Fix 2: MSYS_NO_PATHCONV=1 — disables conversion for the command
MSYS_NO_PATHCONV=1 nmake /f Makefile.win
```

**Why `.bat` wrappers avoid this problem entirely**: `.bat` files run inside `cmd.exe`, where MSYS conversion does not apply. A build script like `tools/build.bat debug` passes through Git Bash without mangling because Git Bash delegates the `.bat` to `cmd.exe`, and all the `/W4`, `/Od`, `/link` flags exist inside the `.bat` file where `cmd.exe` handles them natively. This is why the project uses `.bat` wrappers for MSVC builds rather than calling `cl` or `nmake` directly.

### Null Device

Git Bash doesn't recognize `nul` as the Windows null device:

```bash
# Wrong - creates a file literally named 'nul'
some_command >nul 2>&1

# Correct in Git Bash
some_command >/dev/null 2>&1
```

**Note:** Inside `.bat` files (which run in cmd.exe), use `>nul`. The `/dev/null` form is only for commands typed directly in Git Bash.

### Paths and Backslashes

Git Bash converts Unix-style paths automatically:

```bash
# These are equivalent in Git Bash:
/c/Users/name/file.txt
C:/Users/name/file.txt
```

**Always use forward slashes** for paths in Git Bash. Backslashes in unquoted strings are escape characters (`\b` → backspace, `\t` → tab, `\n` → newline). If a Windows tool requires backslashes, pass them inside quotes: `"src\main.c"`.

### Invoking Batch Files

Git Bash automatically delegates `.bat` files to `cmd.exe`. Run them directly — do **not** wrap in `cmd /c` or `powershell`.

```bash
# Correct — direct invocation with forward slashes
tools/build.bat
tools/build.bat debug

# Also works — absolute paths (both styles)
/c/path/to/build.bat
"C:/path/to/build.bat"

# WRONG — backslash is an escape character in bash
tools\build.bat           # bash sees "toolsbuild.bat"

# WRONG — cmd /c loses output capture
cmd /c tools/build.bat    # runs but output not captured by Git Bash
cmd //c tools/build.bat   # fails: 'tools' is not recognized

# WRONG — unnecessary escalation
powershell -Command "& cmd.exe /c ..."   # never do this
```

Plain word arguments (`debug`, `test`, `clean`) pass through without issues. Slash-prefixed arguments are subject to the same MSYS path conversion described above — use `MSYS_NO_PATHCONV=1` if needed.

### stderr from Child Processes in Batch Files

When Git Bash runs a `.bat` file, stderr from child processes (compilers, tools) inside the batch file may not be captured by Git Bash's `2>&1` redirection. This makes error messages invisible when piping or capturing output.

**Symptom:** A build fails with `[target] FAILED` but no compiler error messages appear, even with `2>&1` on the Git Bash command line.

**Fix:** Redirect stderr to stdout on the child process invocations **inside the `.bat` file**:

```cmd
REM Wrong — stderr from jai.exe may not reach Git Bash
%JAI% src/main.jai

REM Correct — merge stderr at the cmd.exe level
%JAI% src/main.jai 2>&1
```

The `2>&1` inside the `.bat` file merges stderr within `cmd.exe`'s context, so Git Bash receives everything on stdout. Adding `2>&1` on the Git Bash command line only redirects the outer `cmd.exe` process's stderr, not the stderr of child processes launched inside the batch file.

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
