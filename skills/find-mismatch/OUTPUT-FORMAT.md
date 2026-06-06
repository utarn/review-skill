# Output Format

## Report Header

When the fallow pre-check ran, include a summary header before individual findings:

```
## Review Summary
- **Manual review bugs found**: N
- **Fallow auto-fixed**: N findings (staged files only)
- **Fallow non-auto-fixable**: N findings (see below)
- **Fallow verdict**: pass / fail
```

When fallow was skipped (not a JS/TS project, or fallow not installed), omit the fallow header entirely.

## Individual Findings

For each bug found, report:

1. **File and line number** (or approximate location)
2. **Category** (from the checklist)
3. **What's wrong** (one sentence)
4. **What happens at runtime** (error message, wrong data, silent failure, etc.)
5. **Suggested fix** (the specific code change)

## Example

```
## Bug #1
- **File**: src/api/handlers.ts:47
- **Category**: Cross-Boundary Contract Mismatch — parameter name
- **What's wrong**: Caller sends `{ userId: 123 }` but `getUser()` expects `{ user_id: number }`
- **Runtime effect**: `user_id` is undefined in the handler, query returns no results
- **Fix**: Change caller to send `{ user_id: 123 }` or rename the parameter in `getUser()`
```

### Fallow-sourced findings

Non-auto-fixable fallow findings (and auto-fixable findings on unstaged files) appear in the report with a `[fallow]` tag:

```
## Bug #2  [fallow]
- **File**: src/utils.ts:12
- **Category**: Placeholder & Stub Code — unused export
- **What's wrong**: `unusedHelper` is exported but never imported by any file
- **Runtime effect**: No runtime break, but dead code that increases bundle size and maintenance burden
- **Auto-fixed**: no (or: `yes (unstaged — not applied)`)
- **Fix**: Remove the export and the function, or add a consumer
```

Auto-fixed findings (staged files) are **not listed individually**. They appear only as a count in the Review Summary header.

## What NOT to report

- Style issues, naming conventions, or formatting
- Hypothetical performance improvements
- Missing tests or documentation
- "Consider using X instead of Y" suggestions

Only report things that **will break or produce incorrect results**.
