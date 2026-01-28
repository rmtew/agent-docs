# GNU Make Patterns

Common patterns and conventions for GNU Make (Linux/Unix).

## Basic Structure

```makefile
# Variables
CC = gcc
CFLAGS = -std=c11 -Wall -Wextra
LDFLAGS = -lm

# Default target (first target is default)
all: myapp

# Build rules
myapp: main.o utils.o
	$(CC) $(LDFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

# Phony targets
.PHONY: all clean test

clean:
	rm -f *.o myapp

test: myapp
	./test_runner
```

## Automatic Variables

| Variable | Meaning |
|----------|---------|
| `$@` | Target name |
| `$<` | First prerequisite |
| `$^` | All prerequisites (deduplicated) |
| `$+` | All prerequisites (with duplicates) |
| `$*` | Stem (matched by `%`) |
| `$(@D)` | Directory part of target |
| `$(@F)` | File part of target |

## Pattern Rules

```makefile
# Compile .c to .o
%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

# Compile .c to .o in different directory
build/%.o: src/%.c
	@mkdir -p $(@D)
	$(CC) $(CFLAGS) -c -o $@ $<
```

## Parallel Builds

```makefile
# In the Makefile, declare independent targets properly
# Make will parallelize automatically with -j

# At invocation:
make -j$(nproc)      # Use all cores
make -j4             # Use 4 cores
make -j              # Unlimited (not recommended)

# Self-detecting parallelism in Makefile
NPROC := $(shell nproc 2>/dev/null || echo 4)
MAKEFLAGS += -j$(NPROC)
```

## Dependency Generation

Auto-generate header dependencies:

```makefile
SRCS = main.c utils.c parser.c
OBJS = $(SRCS:.c=.o)
DEPS = $(SRCS:.c=.d)

CFLAGS += -MMD -MP

-include $(DEPS)

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<
```

`-MMD` generates `.d` files with dependencies. `-MP` adds phony targets for headers (prevents errors if headers are deleted).

## Conditional Logic

```makefile
# Based on variable
DEBUG ?= 0
ifeq ($(DEBUG), 1)
    CFLAGS += -g -O0 -DDEBUG
else
    CFLAGS += -O2 -DNDEBUG
endif

# Based on environment
ifdef VERBOSE
    Q =
else
    Q = @
endif

%.o: %.c
	$(Q)$(CC) $(CFLAGS) -c -o $@ $<

# Based on command availability
HAS_CLANG := $(shell command -v clang 2>/dev/null)
ifdef HAS_CLANG
    CC = clang
endif
```

## Common Targets

```makefile
.PHONY: all clean install uninstall test debug release

# Default
all: $(TARGET)

# Debug build
debug: CFLAGS += -g -O0 -DDEBUG
debug: $(TARGET)

# Release build
release: CFLAGS += -O2 -DNDEBUG
release: clean $(TARGET)

# Run tests
test: $(TARGET)
	./run_tests.sh

# Install to system
PREFIX ?= /usr/local
install: $(TARGET)
	install -d $(DESTDIR)$(PREFIX)/bin
	install -m 755 $(TARGET) $(DESTDIR)$(PREFIX)/bin/

uninstall:
	rm -f $(DESTDIR)$(PREFIX)/bin/$(TARGET)

# Clean build artifacts
clean:
	rm -f $(OBJS) $(DEPS) $(TARGET)

# Clean everything including generated files
distclean: clean
	rm -f config.h
```

## Out-of-Source Builds

Keep build artifacts separate from source:

```makefile
SRC_DIR = src
BUILD_DIR = build

SRCS = $(wildcard $(SRC_DIR)/*.c)
OBJS = $(SRCS:$(SRC_DIR)/%.c=$(BUILD_DIR)/%.o)

$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c | $(BUILD_DIR)
	$(CC) $(CFLAGS) -c -o $@ $<

$(BUILD_DIR):
	mkdir -p $@

$(TARGET): $(OBJS)
	$(CC) $(LDFLAGS) -o $@ $^
```

The `| $(BUILD_DIR)` is an order-only prerequisite - it ensures the directory exists but doesn't trigger rebuilds when the directory's timestamp changes.

## Recursive Make

For multi-directory projects:

```makefile
SUBDIRS = lib app tests

.PHONY: all clean $(SUBDIRS)

all: $(SUBDIRS)

$(SUBDIRS):
	$(MAKE) -C $@

# Declare dependencies between subdirs
app: lib
tests: lib app

clean:
	for dir in $(SUBDIRS); do $(MAKE) -C $$dir clean; done
```

**Note:** Recursive make has known issues with parallelism and dependency tracking. For large projects, consider non-recursive approaches or CMake.

## Useful Functions

```makefile
# Wildcard - find files
SRCS = $(wildcard src/*.c)

# Substitution
OBJS = $(SRCS:.c=.o)
OBJS = $(patsubst %.c,%.o,$(SRCS))
OBJS = $(SRCS:src/%.c=build/%.o)

# Filter
C_SRCS = $(filter %.c,$(SRCS))
NOT_TEST = $(filter-out %_test.c,$(SRCS))

# Shell command
GIT_VERSION = $(shell git describe --tags --always)
CFLAGS += -DVERSION=\"$(GIT_VERSION)\"

# Foreach
LIBS = sqlite3 ncurses pthread
LDFLAGS += $(foreach lib,$(LIBS),-l$(lib))

# Word functions
FIRST = $(firstword $(SRCS))
REST = $(wordlist 2,$(words $(SRCS)),$(SRCS))
```

## Debugging Makefiles

```makefile
# Print variable value
$(info CFLAGS is $(CFLAGS))

# Print when rule runs
%.o: %.c
	@echo "Compiling $<"
	$(CC) $(CFLAGS) -c -o $@ $<

# Debug mode: print commands without executing
make -n

# Print database of rules
make -p

# Print why target is being rebuilt
make --debug=b
```

## Portable Considerations

```makefile
# Detect OS
UNAME := $(shell uname)
ifeq ($(UNAME), Linux)
    LDFLAGS += -lrt
endif
ifeq ($(UNAME), Darwin)
    LDFLAGS += -framework CoreFoundation
endif

# Use portable commands
RM = rm -f
MKDIR = mkdir -p
INSTALL = install

# Avoid bash-isms in recipes (use /bin/sh compatible syntax)
SHELL = /bin/sh
```
