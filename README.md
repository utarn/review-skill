# review-skill

A code review skill for AI coding agents. Finds real bugs — not style issues.

## Install

```bash
npx skills@latest add utarn/review-skill
```

## What it does

`/review-code` performs systematic bug detection across your entire codebase:

- **Cross-boundary contract mismatches** — function names, parameter names/types, return types
- **Serialization gaps** — casing mismatches, optional vs required fields, encoding layers
- **Logic bugs** — double counting, off-by-one, dead code, shadowed variables
- **Property & method access errors** — null dereferences, wrong types, optional chaining
- **Async & concurrency bugs** — missing awaits, race conditions, resource leaks
- **Placeholder & stub code** — unused results, TODOs, dead imports
- **Language-specific checks** — type system gaps per language

## Usage

In your AI agent (Claude Code, etc.):

```
/review-code
```

Then specify files or directories to review, or let it scan the whole project.

## Skill format

This repo is compatible with [skills.sh](https://github.com/mattpocock/skills).

```
review-skill/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── review-code/
│       ├── SKILL.md              # Main instructions
│       ├── CHECKLIST.md          # Detailed checklist
│       ├── LANGUAGE-SPECIFIC.md  # Per-language checks
│       └── OUTPUT-FORMAT.md      # Reporting format
└── README.md
```

## Skills

| Skill | Description |
|-------|-------------|
| [review-code](skills/review-code/SKILL.md) | Systematic bug detection — finds cross-boundary mismatches, serialization gaps, logic bugs, async bugs, and stub code |
| [work-on-issues](skills/work-on-issues/SKILL.md) | Fetch issues from GitHub or GitLab, implement them, and close completed tickets |

## License

MIT
