# Agent Development Documentation

Shared documentation for AI-assisted development with Claude Code and similar tools. These patterns are generic and reusable across projects.

## Contents

### Windows Development
| Document | Description |
|----------|-------------|
| [windows/msvc_claude_code.md](windows/msvc_claude_code.md) | MSVC toolchain with Claude Code - auto-detection, Git Bash quirks |
| [windows/wsl_development.md](windows/wsl_development.md) | WSL2 development patterns, file system interop |
| [windows/terminal_quirks.md](windows/terminal_quirks.md) | Git Bash, PowerShell, cmd differences |

### Linux Development
| Document | Description |
|----------|-------------|
| [linux/development.md](linux/development.md) | Linux-native development patterns |

### Build Systems
| Document | Description |
|----------|-------------|
| [build/make_patterns.md](build/make_patterns.md) | GNU Make conventions and patterns |
| [build/nmake_patterns.md](build/nmake_patterns.md) | Windows nmake patterns |

### Workflows
| Document | Description |
|----------|-------------|
| [workflows/git_conventions.md](workflows/git_conventions.md) | Git commit messages, branching, PR workflows |

## Usage

Reference these docs from your project's CLAUDE.md:

```markdown
## References

For Windows MSVC setup, see [agent-docs/windows/msvc_claude_code.md](../agent-docs/windows/msvc_claude_code.md)
```

Or copy relevant sections into your project if you prefer self-contained documentation.

## Contributing

When adding new patterns:
1. Keep content generic - no project-specific references
2. Include concrete examples
3. Document both "what" and "why"
4. Add entry to this README
