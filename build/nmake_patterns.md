# Windows nmake Patterns

Common patterns for Microsoft nmake (Windows).

## Basic Structure

```makefile
# Variables
CC = cl
CFLAGS = /W4 /O2
LDFLAGS = /link

# Default target
all: myapp.exe

# Build rules
myapp.exe: main.obj utils.obj
	$(CC) /Fe:$@ $** $(LDFLAGS)

.c.obj:
	$(CC) $(CFLAGS) /c /Fo:$@ $<

# Clean
clean:
	del /q *.obj *.exe 2>nul
```

## Key Differences from GNU Make

| Feature | GNU Make | nmake |
|---------|----------|-------|
| Variable reference | `$(VAR)` or `${VAR}` | `$(VAR)` only |
| Automatic variables | `$@`, `$<`, `$^` | `$@`, `$<`, `$**` |
| All prerequisites | `$^` | `$**` |
| Pattern rules | `%.o: %.c` | `.c.obj:` (suffix rules) |
| Recipe prefix | Tab | Tab |
| Comments | `#` | `#` |
| Line continuation | `\` | `\` |
| Conditionals | `ifeq/else/endif` | `!IF/!ELSE/!ENDIF` |

## Automatic Variables

| Variable | Meaning |
|----------|---------|
| `$@` | Target name |
| `$<` | First dependency (only in inference rules) |
| `$**` | All dependencies |
| `$?` | Dependencies newer than target |

## Suffix Rules

nmake uses suffix rules instead of pattern rules:

```makefile
# Define suffixes
.SUFFIXES: .c .obj .exe

# Suffix rule: .c -> .obj
.c.obj:
	$(CC) $(CFLAGS) /c /Fo:$@ $<

# With different directories, use explicit rules instead
```

## Conditionals

```makefile
# If defined
!IFDEF DEBUG
CFLAGS = /W4 /Od /Zi /DDEBUG
!ELSE
CFLAGS = /W4 /O2 /DNDEBUG
!ENDIF

# If equal
!IF "$(CONFIG)" == "debug"
CFLAGS = /Od /Zi
!ELSEIF "$(CONFIG)" == "release"
CFLAGS = /O2
!ELSE
CFLAGS = /O1
!ENDIF

# Numeric comparison
!IF $(VERSION) >= 10
CFLAGS = $(CFLAGS) /std:c11
!ENDIF
```

## Directory Handling

```makefile
SRC_DIR = src
BUILD_DIR = build

# Create directory (ignore if exists)
$(BUILD_DIR):
	@if not exist $(BUILD_DIR) mkdir $(BUILD_DIR)

# Build to different directory
{$(SRC_DIR)}.c{$(BUILD_DIR)}.obj:
	$(CC) $(CFLAGS) /c /Fo:$@ $<

# Or use explicit rules
$(BUILD_DIR)\main.obj: $(SRC_DIR)\main.c
	@if not exist $(BUILD_DIR) mkdir $(BUILD_DIR)
	$(CC) $(CFLAGS) /c /Fo:$@ $<
```

## Common MSVC Compiler Flags

```makefile
# Warnings
CFLAGS = /W4          # Warning level 4 (highest practical)
CFLAGS = /WX          # Treat warnings as errors

# Optimization
CFLAGS = /Od          # No optimization (debug)
CFLAGS = /O2          # Optimize for speed
CFLAGS = /Os          # Optimize for size

# Debug info
CFLAGS = /Zi          # Debug info in PDB
CFLAGS = /Z7          # Debug info in .obj

# Runtime library
CFLAGS = /MT          # Static runtime (release)
CFLAGS = /MTd         # Static runtime (debug)
CFLAGS = /MD          # DLL runtime (release)
CFLAGS = /MDd         # DLL runtime (debug)

# Standards
CFLAGS = /std:c11     # C11 standard
CFLAGS = /std:c17     # C17 standard

# Output
/Fe:name.exe          # Output executable name
/Fo:name.obj          # Output object name
/Fd:name.pdb          # Output PDB name
```

## Linker Flags

```makefile
LDFLAGS = /link

# Libraries
LDFLAGS = $(LDFLAGS) user32.lib gdi32.lib

# Subsystem
LDFLAGS = $(LDFLAGS) /SUBSYSTEM:CONSOLE
LDFLAGS = $(LDFLAGS) /SUBSYSTEM:WINDOWS

# Debug
LDFLAGS = $(LDFLAGS) /DEBUG

# Library paths
LDFLAGS = $(LDFLAGS) /LIBPATH:lib
```

## Complete Example

```makefile
# Makefile.win - Windows build with nmake

CC = cl
CFLAGS = /W4 /std:c11 /I"include"
LDFLAGS = /link

!IFDEF DEBUG
CFLAGS = $(CFLAGS) /Od /Zi /DDEBUG /MDd
LDFLAGS = $(LDFLAGS) /DEBUG
!ELSE
CFLAGS = $(CFLAGS) /O2 /DNDEBUG /MD
!ENDIF

SRC_DIR = src
BUILD_DIR = build
BIN_DIR = bin

TARGET = $(BIN_DIR)\myapp.exe

OBJS = \
	$(BUILD_DIR)\main.obj \
	$(BUILD_DIR)\utils.obj \
	$(BUILD_DIR)\parser.obj

LIBS = sqlite3.lib

# Default target
all: dirs $(TARGET)

# Create directories
dirs:
	@if not exist $(BUILD_DIR) mkdir $(BUILD_DIR)
	@if not exist $(BIN_DIR) mkdir $(BIN_DIR)

# Link
$(TARGET): $(OBJS)
	$(CC) /Fe:$@ $** $(LDFLAGS) $(LIBS)

# Compile
{$(SRC_DIR)}.c{$(BUILD_DIR)}.obj:
	$(CC) $(CFLAGS) /c /Fo:$@ $<

# Clean
clean:
	@if exist $(BUILD_DIR) rmdir /s /q $(BUILD_DIR)
	@if exist $(BIN_DIR) rmdir /s /q $(BIN_DIR)

# Run tests
test: all
	$(BIN_DIR)\myapp.exe --test

.PHONY: all dirs clean test
```

## Invoking from Git Bash

When running nmake from Git Bash (Claude Code's shell), escape forward slashes:

```bash
# Use // for flags
nmake //f Makefile.win
nmake //f Makefile.win DEBUG=1
nmake //f Makefile.win clean
```

See [terminal_quirks.md](../windows/terminal_quirks.md) for more details.

## Useful Commands

```makefile
# Delete files (ignore errors)
clean:
	-del /q $(BUILD_DIR)\*.obj 2>nul
	-del /q $(TARGET) 2>nul

# The - prefix ignores errors (like GNU Make)

# Copy files
install:
	copy $(TARGET) "$(INSTALL_DIR)\"

# Echo (use @ to suppress command echo)
build:
	@echo Building $(TARGET)...
	$(CC) ...
```

## Debugging nmake

```cmd
# Show macro values
nmake /P /f Makefile.win

# Show commands without executing
nmake /N /f Makefile.win

# Verbose output
nmake /D /f Makefile.win
```
