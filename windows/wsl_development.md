# WSL Development Patterns

Guide for developing in WSL (Windows Subsystem for Linux) with Claude Code.

## Overview

WSL2 provides a full Linux kernel running alongside Windows. This enables native Linux development while maintaining access to Windows tools and files.

## Setup

### WSL2 Installation

```powershell
# From elevated PowerShell
wsl --install

# Or install specific distribution
wsl --install -d Ubuntu-22.04

# Check WSL version
wsl --list --verbose
```

### Recommended Configuration

Create or edit `~/.wslconfig` in Windows (`C:\Users\<name>\.wslconfig`):

```ini
[wsl2]
memory=8GB          # Limit memory usage
processors=4        # Limit CPU cores
localhostForwarding=true
```

Create or edit `/etc/wsl.conf` inside WSL:

```ini
[automount]
enabled = true
root = /mnt/
options = "metadata,umask=22,fmask=11"

[interop]
enabled = true
appendWindowsPath = false   # Cleaner PATH, add Windows tools explicitly
```

## File System Interop

### Path Translation

| Windows Path | WSL Path |
|--------------|----------|
| `C:\Users\name\project` | `/mnt/c/Users/name/project` |
| `D:\code` | `/mnt/d/code` |

### Performance Considerations

**Store projects in the Linux filesystem for best performance:**

```bash
# Good - native Linux filesystem (fast)
~/projects/myapp

# Slow - accessing Windows filesystem from WSL
/mnt/c/Users/name/projects/myapp
```

The Windows filesystem (`/mnt/c/...`) has significant overhead due to the 9P protocol translation. For I/O-heavy operations (builds, git), keep files in the Linux filesystem.

### Accessing WSL Files from Windows

WSL filesystems are accessible from Windows via the `\\wsl$` network path:

```
\\wsl$\Ubuntu-22.04\home\username\projects
```

Or in Windows Explorer, type `\\wsl$` in the address bar.

## Running Claude Code in WSL

### Option 1: Native WSL Session

Launch a WSL terminal and run Claude Code directly:

```bash
# From Windows Terminal, open Ubuntu tab, then:
cd ~/projects/myapp
claude
```

This gives you a pure Linux environment. Claude Code will use bash (not Git Bash).

### Option 2: From Windows Accessing WSL

You can invoke WSL commands from Windows:

```powershell
# Run a command in WSL
wsl ls -la /home/user/projects

# Run Claude Code in WSL from Windows
wsl -d Ubuntu-22.04 --cd ~/projects/myapp -- claude
```

## Cross-Environment Workflows

### Building in WSL, Editing in Windows

A common pattern: use VS Code on Windows with the WSL extension, while builds run natively in WSL.

```bash
# From WSL, open VS Code connected to WSL
code .

# VS Code opens on Windows but operates in WSL context
```

### Sharing Git Credentials

Configure Git credential helper to share with Windows:

```bash
# In WSL
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

Or use SSH keys stored in WSL:

```bash
# Generate key in WSL
ssh-keygen -t ed25519 -C "your@email.com"

# Add to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Clipboard Integration

WSL2 can access the Windows clipboard:

```bash
# Copy to Windows clipboard
echo "text" | clip.exe

# Paste from Windows clipboard (requires PowerShell)
powershell.exe Get-Clipboard
```

## Common Issues

### "Permission denied" on Windows Files

Windows files mounted in WSL may have incorrect permissions:

```bash
# Check actual permissions
ls -la /mnt/c/Users/name/file.txt

# If needed, remount with correct options
sudo mount -t drvfs C: /mnt/c -o metadata,uid=1000,gid=1000
```

Or set defaults in `/etc/wsl.conf` (see Setup section).

### Slow Git Operations on /mnt/c

Git is slow on the Windows filesystem due to many small file operations:

**Solution:** Clone repos to the Linux filesystem:

```bash
# Instead of /mnt/c/projects
cd ~
mkdir projects
cd projects
git clone git@github.com:user/repo.git
```

### Line Ending Issues

Windows uses CRLF, Linux uses LF. Configure Git appropriately:

```bash
# In WSL (keep LF in repo, LF in working directory)
git config --global core.autocrlf input

# Create .gitattributes in repo
echo "* text=auto" > .gitattributes
```

### Network/DNS Issues

If DNS resolution fails in WSL:

```bash
# Check current DNS
cat /etc/resolv.conf

# If using auto-generated resolv.conf, it may point to Windows DNS
# which can be slow or fail. To fix:

# 1. Disable auto-generation in /etc/wsl.conf:
[network]
generateResolvConf = false

# 2. Create static /etc/resolv.conf:
nameserver 8.8.8.8
nameserver 8.8.4.4

# 3. Restart WSL
wsl --shutdown
```

### WSL Doesn't Start or Hangs

```powershell
# Restart WSL entirely
wsl --shutdown

# Check status
wsl --list --verbose

# If persistent issues, restart the WSL service
# (Run as Administrator)
net stop LxssManager
net start LxssManager
```

## WSL vs Git Bash

| Aspect | WSL | Git Bash |
|--------|-----|----------|
| Kernel | Real Linux kernel | Windows kernel with POSIX emulation |
| Performance | Native Linux speed | Some overhead |
| Compatibility | Full Linux compatibility | Most Unix tools, some gaps |
| File access | Best on Linux FS, slow on Windows FS | Native Windows FS |
| Windows integration | Via /mnt/c, interop | Direct |
| Use case | Linux-native development | Quick Windows scripting, Claude Code default |

**Choose WSL when:**
- Building Linux-native applications
- Using Linux-specific tools (systemd services, Linux containers)
- Performance-critical builds

**Choose Git Bash when:**
- Quick Windows operations
- Building Windows-native applications
- Using MSVC toolchain
