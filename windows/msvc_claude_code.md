# Windows MSVC Development with Claude Code

Generic guide for developing C/C++ projects on Windows using Claude Code with MSVC tools. This document covers environment setup patterns that work across projects.

## Environment Setup

### Option 1: Self-Configuring Build Scripts (Recommended)

Build scripts can automatically detect and configure the MSVC environment, allowing them to run from any terminal:

- Windows Terminal (PowerShell or cmd)
- Git Bash
- VS Code integrated terminal

No need to manually open the x64 Native Tools Command Prompt.

### Option 2: Launch from Developer Command Prompt

For tools like `nmake` that don't have wrapper scripts, launch Claude Code from an **x64 Native Tools Command Prompt for VS 2022**:

1. Open Start Menu
2. Find "x64 Native Tools Command Prompt for VS 2022"
3. Navigate to the project directory
4. Launch Claude Code: `claude`

### Why the Developer Prompt Works

Claude Code uses Git Bash internally as its shell, even when launched from cmd.exe. Git Bash inherits environment variables from its parent process. When launched from the x64 Developer Command Prompt:

- `cl.exe`, `nmake.exe`, `link.exe` are in PATH
- `INCLUDE` and `LIB` paths are set for MSVC headers and libraries
- These tools become accessible from Git Bash

This gives you both:
- **Git Bash**: Standard Unix-like commands (grep, sed, find, etc.) that Claude Code expects
- **MSVC Tools**: Native Windows compiler toolchain for builds

## Self-Configuring Build Script Pattern

Add this block to the start of any `.bat` file to make it work from any terminal and any directory:

```batch
@echo off
setlocal EnableDelayedExpansion
cd /d "%~dp0"

REM Auto-setup MSVC environment if not already configured
where cl.exe >nul 2>&1
if %ERRORLEVEL% NEQ 0 (
    echo Setting up MSVC environment...
    set "VSWHERE=%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\vswhere.exe"
    if not exist "!VSWHERE!" (
        echo ERROR: vswhere.exe not found. Install Visual Studio with C++ workload.
        exit /b 1
    )
    for /f "usebackq tokens=*" %%i in (`"!VSWHERE!" -latest -property installationPath`) do set "VSINSTALL=%%i"
    if not defined VSINSTALL (
        echo ERROR: Could not find Visual Studio installation
        exit /b 1
    )
    call "!VSINSTALL!\VC\Auxiliary\Build\vcvarsall.bat" x64 >nul 2>&1
    where cl.exe >nul 2>&1
    if !ERRORLEVEL! NEQ 0 (
        echo ERROR: Failed to initialize MSVC environment
        exit /b 1
    )
)

REM Your build commands here...
cl /W4 /O2 myfile.c /Fe:myfile.exe
```

**How it works:**

1. `cd /d "%~dp0"` - Changes to the script's own directory so it can be invoked from anywhere
2. `where cl.exe` - Checks if MSVC is already configured (skip setup if so)
3. `vswhere.exe` - Finds the latest Visual Studio installation path
4. `vcvarsall.bat x64` - Configures the x64 native compiler environment
5. Second `where cl.exe` - Verifies setup succeeded (more reliable than vcvarsall's exit code)

**Key points:**
- `EnableDelayedExpansion` is required for `!VARIABLE!` syntax inside `if` blocks
- `>nul 2>&1` suppresses vcvarsall's verbose output
- The verification step catches cases where vcvarsall runs but fails silently

## Invoking Build Scripts from Git Bash

Run `.bat` files directly with forward slashes. Git Bash automatically delegates them to `cmd.exe` — do **not** add `cmd /c` or `powershell` wrappers, which break output capture.

```bash
# Correct — direct invocation
tools/build.bat
tools/build.bat debug

# WRONG — all of these break output capture or fail entirely
cmd /c tools/build.bat          # output lost
cmd //c tools/build.bat         # 'tools' not recognized
powershell -Command "& ..."     # unnecessary layers
tools\build.bat                 # backslash eaten by bash
```

See [terminal_quirks.md](terminal_quirks.md) for details on path conversion and slash-prefixed arguments.

## Common MSVC Warnings

### C4267: size_t to int conversion

MSVC warns when assigning `size_t` (from `strlen()`, etc.) to `int`:

```c
// Warning C4267
int len = strlen(str);

// Fixed - explicit cast
int len = (int)strlen(str);
```

### "undefined; assuming extern returning int"

A function is being called without a visible declaration. Ensure the header with the function prototype is included before use.

### C4245: signed/unsigned mismatch

Assigning a signed value to an unsigned type (like `DWORD`):

```c
// Warning C4245
DWORD flags = -1;

// Fixed - use unsigned literal or explicit cast
DWORD flags = (DWORD)-1;
DWORD flags = 0xFFFFFFFF;
```

## Troubleshooting

### "nmake: command not found" or "cl: command not found"

**Cause:** MSVC environment not configured.

**Solutions:**

For self-configuring build scripts:
- Run the `.bat` file directly - it will auto-detect MSVC
- If auto-detection fails, ensure Visual Studio is installed with the "Desktop development with C++" workload

For direct nmake/cl usage:
1. Close Claude Code
2. Open x64 Native Tools Command Prompt for VS 2022
3. Navigate to project directory
4. Relaunch Claude Code

### Build script works in cmd but fails from Git Bash

Check for:
- **Backslashes in the path**: Use `tools/build.bat`, not `tools\build.bat` — backslash is an escape character in bash
- **Using `cmd /c` wrapper**: Remove it — run the `.bat` file directly. `cmd /c` breaks output capture from Git Bash
- **Paths with spaces not properly quoted**: Use double quotes around the path
- **`>nul` in Git Bash commands**: Use `>/dev/null` instead (inside `.bat` files, `>nul` is correct since they run in cmd.exe)

### vcvarsall.bat returns error but environment seems configured

vcvarsall.bat can return non-zero even when successful (it calls vswhere.exe internally which may warn). The pattern above handles this by checking if `cl.exe` is available after the call rather than trusting the exit code.
