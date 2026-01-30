# Shared External Dependencies

How to manage external dependencies (SDKs, toolchains, models, libraries) across multiple repos and git worktrees using a single shared directory.

## Problem

External dependencies -- prebuilt libraries, toolchains, large model files -- are expensive to duplicate. When using git worktrees or multiple repos that share the same dependencies, each copy wastes disk space and creates version drift when one gets updated but others don't.

## Pattern

Keep all external deps in a single directory **outside any repo**, and reference it via the `DEPS_ROOT` environment variable.

```
parent-dir/
  deps/                    # Shared deps (DEPS_ROOT points here)
    library-a/1.2.3/       # Version-pinned subdirectory
    library-b/4.5.6/
    toolchain/3.1.0/
    models/
      category-a/          # Large model files, grouped by use
      category-b/
    _archives/              # Original downloads (zips, tarballs)
  project/                  # Main worktree
  project-feature-x/       # Another worktree -- same deps
  other-repo/               # Different repo -- same deps
```

Every build script, makefile, and tool resolves dependencies through `DEPS_ROOT`. No repo contains hardcoded absolute paths or relative path fallbacks.

## Directory Conventions

### Version-pinned subdirectories

Each dependency gets a directory named `{package}/{version}/`:

```
deps/
  sqlite/3510200/          # SQLite 3.51.2
  cmake/3.31.4/            # Portable cmake
  sherpa-onnx/v1.12.23/    # SDK release
```

This allows multiple versions to coexist during migration. Build scripts select the version they need (often via a `VERSION` variable in the makefile).

### Source-tree dependencies

Some deps are full source trees (git clones with build artifacts). These don't need version directories since the git history tracks versions:

```
deps/
  whisper.cpp/             # git clone + cmake build output
    include/
    build/
```

### Models directory

Large model files are grouped by category, not by the tool that uses them:

```
deps/
  models/
    whisper/               # Speech-to-text models
    llm/                   # Language models
    tts/                   # Text-to-speech models
    embeddings/            # Embedding models
```

This avoids duplicating models when multiple tools use the same one.

### Archives directory

Keep original downloads in `_archives/` so dependencies can be rebuilt without re-downloading:

```
deps/
  _archives/
    sqlite-amalgamation-3510200.zip
    sqlite-dll-win-x64-3510200.zip
    cmake-3.31.4-windows-x86_64.zip
```

## Build Script Integration

### Requiring DEPS_ROOT

Every script that uses shared deps must check `DEPS_ROOT` at startup and fail with a clear message if it's unset. Do not fall back to relative paths -- that silently breaks in worktrees.

**Batch file pattern:**
```cmd
if not defined DEPS_ROOT (
    echo ERROR: DEPS_ROOT environment variable is not set.
    echo Set it to the shared deps directory, e.g.:
    echo   setx DEPS_ROOT "C:\path\to\deps"
    exit /b 1
)
```

**Makefile pattern:**
```makefile
!IF "$(DEPS_ROOT)" == ""
!ERROR DEPS_ROOT environment variable is not set. Set it to the shared deps directory.
!ENDIF
```

**Shell script pattern:**
```bash
if [ -z "$DEPS_ROOT" ]; then
    echo "ERROR: DEPS_ROOT is not set." >&2
    echo "Set it to the shared deps directory." >&2
    exit 1
fi
```

### Resolving paths

Build scripts compose paths from `DEPS_ROOT` plus the package name and version:

```makefile
SQLITE_VER = 3510200
SQLITE_DIR = $(DEPS_ROOT)\sqlite\$(SQLITE_VER)
CFLAGS = /I"$(SQLITE_DIR)"
LDFLAGS = /LIBPATH:"$(SQLITE_DIR)"
```

The version variable makes upgrades a single-line change in the build file.

## Setup

### Setting DEPS_ROOT permanently (Windows)

```cmd
setx DEPS_ROOT "C:\path\to\deps"
```

This writes to the user's environment. New terminal windows will have it set. Existing windows need `set DEPS_ROOT=...` for the current session.

### Setting DEPS_ROOT per-session

```cmd
set DEPS_ROOT=C:\path\to\deps
```

```bash
export DEPS_ROOT=/path/to/deps
```

### Verifying

After setting, confirm the variable and check that expected contents exist:

```cmd
echo %DEPS_ROOT%
dir %DEPS_ROOT%
```

## Discovering Available Dependencies

List the contents of `DEPS_ROOT` to see what's available:

```cmd
dir %DEPS_ROOT%
```

```bash
ls "$DEPS_ROOT"
```

Each subdirectory follows the conventions above (version-pinned dirs, `models/` by category, `_archives/` for originals). Project-specific docs should describe which dependencies they need and how to set them up.
