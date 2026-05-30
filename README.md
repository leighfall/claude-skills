# claude-skills

Personal Claude Code configuration — rules, commands, and skills that travel with me across projects.

## Structure

```
.claude/
├── CLAUDE.md                          # Index of all rules, commands, and skills
├── commands/                          # Slash commands
│   ├── next-test.md                   # Write the next test for a class or controller
│   ├── heal-tests.md                  # Fix failing, broken, or flaky tests
│   ├── code-review.md                 # Run code review on current diff
│   ├── decompose.md                   # Break scope into epics, stories, tasks + estimates
│   └── estimate.md                    # Estimate a single described task
├── rules/                             # Always-available conventions
│   ├── communication.md               # How Claude communicates + my stack experience levels
│   ├── testing.md                     # Testing philosophy
│   ├── vue.md                         # Vue 3 Composition API conventions
│   ├── dotnet.md                      # .NET Core / C# conventions
│   └── no-overengineering.md          # Simplest solution first
└── skills/                            # Loaded on demand by commands
    ├── xunit-integration.md           # xUnit integration test patterns
    ├── xunit-unit.md                  # xUnit unit test patterns
    ├── estimation-decomposition/      # Breakdown and estimation conventions
    ├── code-review-skill/             # Placeholder — awesome-skills/code-review-skill
    ├── playwright-skill/              # testdino-hq/playwright-skill (full install)
    └── vitest-skill/                  # Placeholder — fastmcp vitest skill
```

## Usage

Drop this repo's `.claude/` directory into any project, or symlink it to `~/.claude/` to apply globally.

Commands are available as slash commands in Claude Code (e.g. `/next-test`, `/decompose`).

## Status

| File | Status |
|------|--------|
| `commands/next-test.md` | Done |
| `skills/xunit-integration.md` | Done |
| `skills/xunit-unit.md` | Done |
| `rules/communication.md` | Done |
| `commands/heal-tests.md` | Placeholder |
| `commands/code-review.md` | Placeholder |
| `commands/decompose.md` | Placeholder |
| `commands/estimate.md` | Placeholder |
| `rules/testing.md` | Placeholder |
| `rules/vue.md` | Placeholder |
| `rules/dotnet.md` | Placeholder |
| `rules/no-overengineering.md` | Placeholder |
| `skills/estimation-decomposition/` | Placeholder |
| `skills/code-review-skill/` | Pending external skill |
| `skills/playwright-skill/` | Done (testdino-hq/playwright-skill) |
| `skills/vitest-skill/` | Pending external skill |
