# Session Conventions for Claude Code Projects

Shared patterns for working with Claude Code across projects. Reference this from your project's CLAUDE.md.

## File Structure

Projects using these conventions maintain:

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project-specific guidance (build commands, architecture, tools) |
| `JOURNAL.md` | Session summaries - what changed, why, test results |
| `BACKLOG.md` | Workarounds, deferred work, ideas for future |
| `docs/research/` | Deep dives, specs, implementation notes |

## Research Sub-Projects

Sub-projects are directories with their own README, journal, and backlog. The primary location is `docs/research/[topic]/` but the pattern applies wherever a sub-project is created.

**Required files:**

- `README.md` -- problem statement, approach, architecture, how to build/run (if applicable), file inventory
- `JOURNAL.md` -- dated session logs (same format as root journal)
- `BACKLOG.md` -- phased roadmap, deferred work, ideas

**Optional (when the sub-project includes runnable code):**

- `src/` -- prototype source files
- Build scripts (`build.bat`, `Makefile`) at the sub-project root

Documentation-only sub-projects (e.g., dataset schema research) can use multiple topic `.md` files instead of or alongside the standard three.

**Integration steps when creating a new sub-project:**

1. Add an entry to the design references table in `CLAUDE.md`
2. Add a section in root `BACKLOG.md` referencing the sub-project's backlog
3. If the sub-project leads to mainline implementation, create a corresponding `docs/design/[topic].md`

## Session Workflow

### Before Starting Work

1. Read JOURNAL.md to understand recent work and context
2. Check BACKLOG.md for pending issues or deferred work relevant to current task
3. Use this context to inform task list creation

### Task Lists

Use task lists for any work involving multiple steps or file modifications.

**Structure:**
1. ... work tasks ...
2. `[Finalize]` Update JOURNAL.md with session summary
3. `[Finalize]` Update BACKLOG.md if workarounds/deferred items added
4. `[Finalize]` Commit all changes

After completing all tasks, halt for user review before proceeding to new work.

### Journal Maintenance

Append entries at the end of JOURNAL.md. Use narrative style with:
- `## YYYY-MM-DD: Topic` header
- `### Subsection` for key decisions or findings
- **Bold labels** for important points
- Code snippets where relevant
- Reasoning behind decisions, not just what was done

When updating a subproject journal, add a brief entry in the root JOURNAL.md referencing the update.

### Backlog Maintenance

Track in BACKLOG.md:
- Workarounds added (with context for why)
- Deferred work (with reason for deferral)
- Ideas for future improvements
- Known issues to revisit

Update when adding workarounds or identifying issues to fix later.

## Autonomy Rules

### Scope of Changes

No limit. Proceed with changes of any size.

### Destructive Actions

- Before overwriting work that hasn't been committed, confirm with user
- No git push - local commits only (unless user explicitly requests)
- **Never run** `git clean -fd` without checking `git status` first

### Uncertainty During Work

- If there's a reasonable path forward that doesn't accumulate error, make a best guess and mark with `TODO:`
- Halt at end of task list for user review before continuing

### Architectural Decisions

Prompt user. Directory structure, file organization, and naming conventions follow direct user instruction.

### Tool Usage

Proceed with using tools to maintain automated flow.

## Error Handling

When tools fail, crash, produce unexpected output, or dependencies are missing:

1. **If error is due to incorrect usage** - Attempt to fix the invocation and retry
2. **If unfixable** - Halt for user review
3. **If other work can proceed** - Log the error and pivot to unblocked tasks

**Document errors in both:**
- JOURNAL.md under `### Errors` section - what happened, what was tried
- BACKLOG.md - for follow-up investigation

## Code Comments

- Mark unclear sections with `TODO:` or `UNKNOWN:`
- Use comments to explain *why*, not *what*

## External References

**Archival approach:**
- Include source URL in comments/docs
- Save excerpts and summaries locally
- Avoid link rot by capturing key information

## CLAUDE.md Principles

Keep the project's CLAUDE.md:
1. **Scannable** - Use headers and bullet points; move large content to linked files
2. **Project-specific** - Don't duplicate these shared conventions; reference them
3. **Actionable** - Build commands, verification steps, architecture constraints
