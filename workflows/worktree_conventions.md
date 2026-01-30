# Worktree Conventions

Conventions for Claude-managed git worktrees. The developer stays in the main repo directory; Claude handles worktree lifecycle, file paths, and builds.

## Layout

Worktrees live inside the main repo, gitignored:

```
project/
  worktrees/
    targets-ui/           # git worktree checkout
      TASK.md             # Worktree-specific task file
      src/                # Full source tree (git worktree)
      ...
    recipe-editor/
      TASK.md
      ...
```

Each worktree is a full git checkout created via `git worktree add`. Add `worktrees/` to the project's `.gitignore`.

## TASK.md Format

Every worktree has a `TASK.md` at its root describing the work:

```markdown
# <description>
Branch: feature/<slug>
Created: <YYYY-MM-DD>
Source: <backlog item, issue, or user request>

## Goal
<what this worktree is for>

## Status
<current state -- what's done, what's next>

## Context
<relevant design docs, key files, decisions made>
```

## Lifecycle

### Create

```
git worktree add -b feature/<slug> worktrees/<slug>
```

Then write `worktrees/<slug>/TASK.md` with the initial goal and context.

### Continue

Read `worktrees/<slug>/TASK.md` to restore context. Summarize current status to the developer. All subsequent file operations target the worktree path.

### List

Enumerate worktrees via `git worktree list`. For each worktree under `worktrees/`, read its `TASK.md` and show slug + status summary.

### Clean

1. Confirm with developer
2. `git worktree remove worktrees/<slug>`
3. `git branch -d feature/<slug>` (use `-D` only if developer confirms force-delete)

## Session Behavior

When Claude is working in a worktree (after create or continue):

- **File operations**: All Read/Write/Edit/Glob/Grep use the `worktrees/<slug>/` path prefix
- **Build commands**: Run builds from the worktree's directory (not the main repo's)
- **TASK.md updates**: Update status section before ending work in the worktree
- **Git operations**: Run from the worktree directory (`worktrees/<slug>/`)

## Build Context

Each worktree has its own complete checkout, including build scripts. Builds run from the worktree directory so that the worktree's own source and build configuration are used.

Projects with external dependencies should use environment variables or absolute paths so dependencies resolve correctly regardless of worktree location. For example, a project with shared deps outside the repo might use an env var like `DEPS_ROOT` to avoid relying on relative paths that break in nested worktree checkouts.

When building from a worktree:
- Run the worktree's build command (e.g., `worktrees/<slug>/build.sh`, `make -C worktrees/<slug>/`)
- Build artifacts go to the worktree's output directories
- Test commands should also run from the worktree

## Branch Naming

Worktree branches use the `feature/<slug>` convention. The slug should be short, lowercase, hyphen-separated (e.g., `targets-ui`, `recipe-editor`, `srs-module`).

## Adopting in a New Project

To add worktree support to a project that uses agent-docs:

1. Add `worktrees/` to `.gitignore`
2. Copy `agent-docs/workflows/worktree_command_template.md` to `.claude/commands/worktree.md`
3. Customize the "Active Worktree Context" section with project-specific build commands
4. If the project has a build slash command, add worktree path patterns to its `allowed-tools`
5. Reference this conventions doc from the project's `CLAUDE.md`
