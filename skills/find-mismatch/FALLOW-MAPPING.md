# Fallow Mapping

Bridge between [fallow](https://github.com/fallow-rs/fallow) ([docs](https://docs.fallow.tools)) static analysis output and the find-mismatch checklist categories. Fallow is invoked via `fallow audit` — installed globally with `npm install -g fallow` — to surface dead code, circular dependencies, complexity hotspots, duplication, and architecture boundary violations. Used automatically when the reviewed project is JS/TS.

## Finding-to-Category Mapping

| Fallow Finding Type | Checklist Category | Notes |
|---------------------|--------------------|-------|
| `unused_exports` | [Placeholder & Stub Code](CHECKLIST.md#7-placeholder--stub-code) | Exported symbols never imported elsewhere |
| `unused_files` | [Placeholder & Stub Code](CHECKLIST.md#7-placeholder--stub-code) | Entire files with no importers |
| `unused_dependencies` | [Placeholder & Stub Code](CHECKLIST.md#7-placeholder--stub-code) | package.json entries never imported |
| `circular_dependencies` | [Cross-Boundary Contracts](CHECKLIST.md#1-cross-boundary-contract-mismatches) | Circular import chains cause initialization-order bugs |
| `complexity` | [Logic Bugs](CHECKLIST.md#3-logic-bugs) | High cyclomatic complexity correlates with hidden logic errors |
| `duplication` | [Logic Bugs](CHECKLIST.md#3-logic-bugs) | Copy-pasted code drifts — fixes applied to one copy miss the other |

## JSON Parsing Guide

Fallow outputs a JSON object with this top-level structure:

```json
{
  "verdict": "pass" | "fail",
  "dead_code": {
    "unused_exports": [...],
    "unused_files": [...],
    "unused_dependencies": [...]
  },
  "complexity": {
    "files": [...]
  },
  "duplication": {
    "duplicates": [...]
  }
}
```

### Walking the structure

1. Check `verdict` — if `"pass"`, no findings to process.
2. **Dead code**: Iterate `dead_code.unused_exports`, `dead_code.unused_files`, `dead_code.unused_dependencies`. Each entry has `file`, `name`, `line`, and optionally `auto_fixable` with a `fix` object.
3. **Complexity**: Iterate `complexity.files`. Each has `file`, `score`, `threshold`.
4. **Duplication**: Iterate `duplication.duplicates`. Each has `files`, `lines`, `similarity`.

### Auto-fix detection

Entries where `auto_fixable === true` contain a `fix` object:

```json
{
  "file": "src/utils.ts",
  "name": "unusedHelper",
  "line": 12,
  "auto_fixable": true,
  "fix": {
    "type": "remove-export",
    "description": "Remove unused export 'unusedHelper'"
  }
}
```

Fix types:
- `remove-export` — Remove the `export` keyword (or entire declaration if unused locally too)
- `remove-dependency` — Remove the entry from `package.json`
- `remove-file` — Delete the unused file

## Staged File Filtering

After collecting findings, split them based on git staging status:

```bash
# Get list of staged files
git diff --staged --name-only
```

**STAGED findings** (file appears in staged list):
- Apply auto-fixes automatically
- Re-run `fallow audit --format json --quiet` to verify the fix resolved the finding
- If the re-run still reports the same issue, escalate to manual review

**UNSTAGED findings** (file does NOT appear in staged list):
- Do NOT modify the file
- Include in the report as a `[fallow]` tagged finding for manual review
- Mark with `Auto-fixable: yes (unstaged — not applied)` or `Auto-fixable: no` as appropriate

**No staged files at all**:
- Treat all findings as unstaged — report only, no modifications

## Fix Application Guide

### remove-export

1. Read the file containing the unused export
2. If the symbol is used locally (non-exported usages exist): remove only the `export` keyword
3. If the symbol is entirely unused: remove the entire declaration (function, class, const, type, interface)
4. If removing the symbol leaves an empty file with no remaining exports or side effects, flag for `remove-file` instead

### remove-dependency

1. Read `package.json`
2. Remove the dependency entry from the appropriate section (`dependencies`, `devDependencies`, etc.)
3. Do NOT run `npm install` / `yarn` / `pnpm install` — only modify `package.json`

### remove-file

1. Verify the file has no remaining imports from other files (re-run audit or grep)
2. Delete the file
3. Check for any barrel file (`index.ts`) that re-exports from the deleted file and clean up the re-export line

### Verification

After applying fixes to staged files:

```bash
fallow audit --format json --quiet
```

- If `verdict` is now `"pass"`, all fixes succeeded
- If findings remain, check whether they are the same findings (fix failed) or new findings (fix exposed deeper issues). Escalate both to manual review.
