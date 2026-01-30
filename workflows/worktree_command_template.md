---
description: Manage git worktrees for parallel feature development
argument-hint: <create|continue|list|clean> [slug] ["description"]
---

Manage git worktrees for parallel development. See `agent-docs/workflows/worktree_conventions.md` for full conventions.

Parse `$ARGUMENTS` to determine the subcommand:

## Subcommands

### `create <slug> "<description>"`

1. Create the worktree and branch:
   ```
   git worktree add -b feature/<slug> worktrees/<slug>
   ```
2. Write `worktrees/<slug>/TASK.md` with:
   - Description from the argument
   - Branch name: `feature/<slug>`
   - Created date: today
   - Goal section from the description
   - Empty Status and Context sections
3. Report success and set active worktree context for this session.

### `continue <slug>`

1. Verify `worktrees/<slug>/` exists
2. Read `worktrees/<slug>/TASK.md`
3. Summarize the current status to the developer
4. Set active worktree context for this session

### `list`

1. Run `git worktree list`
2. For each worktree under `worktrees/`, read its `TASK.md`
3. Display a summary: slug, branch, status (first line of Status section)

### `clean <slug>`

1. Confirm with the developer before proceeding
2. Remove the worktree: `git worktree remove worktrees/<slug>`
3. Delete the branch: `git branch -d feature/<slug>`
4. Report success

## Active Worktree Context

After `create` or `continue`, inform the developer that the worktree is now active and explain the session behavior:

- All file operations (Read, Write, Edit, Glob, Grep) target `worktrees/<slug>/` paths
- Build commands run from the worktree's directory (not the main repo's)
- TASK.md will be updated before ending work in the worktree
- Git operations run from within the worktree directory

<!-- PROJECT-SPECIFIC: Customize the build instructions below for your project.
     Examples:
       - C/Make project: "Build with `make -C worktrees/<slug>/`"
       - Node project: "Run `npm --prefix worktrees/<slug>/ run build`"
       - Rust project: "Run `cargo build --manifest-path worktrees/<slug>/Cargo.toml`"
     Also update `allowed-tools` in the frontmatter if your build slash command
     needs permission for worktree paths (e.g., `Bash(worktrees/*/build.sh:*)`).
-->
