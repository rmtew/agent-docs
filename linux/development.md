# Linux Development Patterns

Guide for Linux-native development with Claude Code.

## Environment

Claude Code on Linux uses the system's default shell (usually bash). The environment is straightforward compared to Windows - no path translation or shell quirks to navigate.

## Common Build Tools

### GCC/Clang

```bash
# GCC
gcc -std=c11 -Wall -Wextra -o myapp main.c

# Clang
clang -std=c11 -Wall -Wextra -o myapp main.c

# With debug symbols
gcc -g -O0 -o myapp main.c

# Release build
gcc -O2 -DNDEBUG -o myapp main.c
```

### Make

```bash
# Build with default target
make

# Parallel build (auto-detect cores)
make -j$(nproc)

# Specific target
make debug
make test
make clean

# Pass variables
make CC=clang DEBUG=1
```

### CMake

```bash
# Configure (out-of-source build)
mkdir build && cd build
cmake ..

# Configure with options
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake -DCMAKE_C_COMPILER=clang ..

# Build
cmake --build .

# Or use make after configure
make -j$(nproc)
```

## Package Management

### Debian/Ubuntu (apt)

```bash
# Update package lists
sudo apt update

# Install development tools
sudo apt install build-essential
sudo apt install cmake
sudo apt install libsqlite3-dev

# Search for packages
apt search ncurses

# Show package info
apt show libncurses-dev
```

### Fedora/RHEL (dnf)

```bash
# Install development tools
sudo dnf groupinstall "Development Tools"
sudo dnf install cmake
sudo dnf install sqlite-devel

# Search
dnf search ncurses
```

### Arch (pacman)

```bash
# Install
sudo pacman -S base-devel
sudo pacman -S cmake
sudo pacman -S sqlite

# Search
pacman -Ss ncurses
```

## Debugging

### GDB

```bash
# Start debugging
gdb ./myapp

# Common commands
(gdb) run                    # Start program
(gdb) break main             # Set breakpoint
(gdb) break file.c:42        # Breakpoint at line
(gdb) next                   # Step over
(gdb) step                   # Step into
(gdb) continue               # Continue execution
(gdb) print variable         # Print value
(gdb) backtrace              # Show call stack
(gdb) quit                   # Exit

# Run with arguments
gdb --args ./myapp arg1 arg2
```

### Valgrind

```bash
# Memory leak detection
valgrind --leak-check=full ./myapp

# More detailed
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./myapp

# Memory error detection
valgrind ./myapp
```

### Address Sanitizer

```bash
# Compile with sanitizer
gcc -fsanitize=address -g -o myapp main.c

# Run normally - crashes will show detailed info
./myapp
```

## File System

### Common Paths

| Purpose | Path |
|---------|------|
| User config | `~/.config/appname/` or `$XDG_CONFIG_HOME/appname/` |
| User data | `~/.local/share/appname/` or `$XDG_DATA_HOME/appname/` |
| User cache | `~/.cache/appname/` or `$XDG_CACHE_HOME/appname/` |
| System config | `/etc/appname/` |
| System binaries | `/usr/bin/`, `/usr/local/bin/` |
| Libraries | `/usr/lib/`, `/usr/local/lib/` |
| Headers | `/usr/include/`, `/usr/local/include/` |

### XDG Base Directory

Follow the XDG specification for user files:

```c
const char *config_home = getenv("XDG_CONFIG_HOME");
if (!config_home || !config_home[0]) {
    // Default to ~/.config
    config_home = "~/.config";
}
```

## Process Management

### Running in Background

```bash
# Run in background
./long_running_task &

# Disown from shell (survives shell exit)
./server &
disown

# Or use nohup
nohup ./server > server.log 2>&1 &
```

### Signals

```bash
# Send SIGTERM (graceful shutdown)
kill PID

# Send SIGKILL (force kill)
kill -9 PID

# Send SIGHUP (often triggers config reload)
kill -HUP PID

# Find process
pgrep -f "myapp"
ps aux | grep myapp
```

## Permissions

### File Permissions

```bash
# Make executable
chmod +x script.sh

# Set specific permissions (owner rwx, group rx, other rx)
chmod 755 script.sh

# Recursive
chmod -R 644 config/
```

### Common Permission Patterns

| Mode | Meaning | Use case |
|------|---------|----------|
| 755 | rwxr-xr-x | Executables, directories |
| 644 | rw-r--r-- | Regular files |
| 600 | rw------- | Private files (keys, credentials) |
| 700 | rwx------ | Private directories |

## Environment Variables

### Setting Variables

```bash
# For current session
export MY_VAR=value

# For single command
MY_VAR=value ./myapp

# Persistent (add to ~/.bashrc or ~/.profile)
echo 'export MY_VAR=value' >> ~/.bashrc
source ~/.bashrc
```

### Common Variables

| Variable | Purpose |
|----------|---------|
| `PATH` | Executable search path |
| `LD_LIBRARY_PATH` | Shared library search path |
| `PKG_CONFIG_PATH` | pkg-config search path |
| `CC` | C compiler |
| `CXX` | C++ compiler |
| `CFLAGS` | C compiler flags |
| `LDFLAGS` | Linker flags |

## systemd (Service Management)

### User Services

For development, you can create user-level services:

```bash
# Create service file
mkdir -p ~/.config/systemd/user/
cat > ~/.config/systemd/user/myapp.service << 'EOF'
[Unit]
Description=My Application

[Service]
ExecStart=/home/user/bin/myapp
Restart=on-failure

[Install]
WantedBy=default.target
EOF

# Reload and start
systemctl --user daemon-reload
systemctl --user start myapp
systemctl --user status myapp

# Enable on login
systemctl --user enable myapp
```

### Viewing Logs

```bash
# Service logs
journalctl --user -u myapp

# Follow logs
journalctl --user -u myapp -f

# System service logs
sudo journalctl -u nginx
```
