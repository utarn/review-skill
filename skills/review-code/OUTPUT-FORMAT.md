# Output Format

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

## What NOT to report

- Style issues, naming conventions, or formatting
- Hypothetical performance improvements
- Missing tests or documentation
- "Consider using X instead of Y" suggestions

Only report things that **will break or produce incorrect results**.
