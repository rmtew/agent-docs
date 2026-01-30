# Agent Development Documentation

Shared documentation for AI-assisted development with Claude Code. Contains:
- Session conventions and workflow patterns
- Platform-specific notes from actual problems encountered

## Contents

### Workflows
| Document | Description |
|----------|-------------|
| [workflows/session_conventions.md](workflows/session_conventions.md) | Session workflow, journal/backlog maintenance, autonomy rules |
| [workflows/worktree_conventions.md](workflows/worktree_conventions.md) | Git worktree lifecycle, TASK.md format, build context |
| [workflows/worktree_command_template.md](workflows/worktree_command_template.md) | Template slash command -- copy to `.claude/commands/worktree.md` and customize |

### Build
| Document | Description |
|----------|-------------|
| [build/shared_deps.md](build/shared_deps.md) | Shared external dependencies (DEPS_ROOT, deps/ layout) |

### Windows Development
| Document | Description |
|----------|-------------|
| [windows/msvc_claude_code.md](windows/msvc_claude_code.md) | MSVC toolchain with Claude Code - auto-detection, Git Bash quirks |
| [windows/terminal_quirks.md](windows/terminal_quirks.md) | Git Bash, PowerShell, cmd differences |

## Usage

Reference these docs from your project's CLAUDE.md:

```markdown
## Session Conventions

Follow [agent-docs/workflows/session_conventions.md](agent-docs/workflows/session_conventions.md).
```
