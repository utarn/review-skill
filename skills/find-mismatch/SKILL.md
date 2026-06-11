---
name: find-mismatch
description: >
  Systematic code review focused on finding real bugs — cross-boundary contract mismatches,
  logic errors, async bugs, and runtime failures. Use when reviewing AI-generated code,
  reviewing a PR, auditing a codebase for bugs, or when user says "review this code",
  "find bugs", "check for issues", or "find-mismatch".
disable-model-invocation: true
---

# Find Mismatch

Systematic bug detection for AI-generated or any codebase. Finds things that **will break at runtime** — not style issues, not hypotheticals.

## Scope

Walk through **every file**. Check every function call, type annotation, property access, and cross-boundary interaction. Do not skip any file.

## Process

1. Read the full codebase (or specified files/directories)
2. **Fallow Audit (JS/TS only)** — see [Fallow Integration](#fallow-integration) below
3. Apply each checklist category from [CHECKLIST.md](CHECKLIST.md)
4. For language-specific checks, consult [LANGUAGE-SPECIFIC.md](LANGUAGE-SPECIFIC.md)
5. Report findings using the format in [OUTPUT-FORMAT.md](OUTPUT-FORMAT.md)

## Checklist Categories

Apply these in order — the first two are the most critical:

| # | Category | Why it matters |
|---|----------|---------------|
| 1 | Cross-Boundary Contracts | Silent runtime failures — see [CHECKLIST.md](CHECKLIST.md#1-cross-boundary-contract-mismatches) |
| 2 | Serialization Gaps | Data corruption between layers — see [CHECKLIST.md](CHECKLIST.md#2-serialization--deserialization-gaps) |
| 3 | Logic Bugs | Wrong results — see [CHECKLIST.md](CHECKLIST.md#3-logic-bugs) |
| 4 | Property & Method Access | Null dereferences — see [CHECKLIST.md](CHECKLIST.md#4-property--method-access-errors) |
| 5 | Async & Concurrency | Race conditions, leaks — see [CHECKLIST.md](CHECKLIST.md#5-async--concurrency-bugs) |
| 6 | CSS & Styling Mismatches | Silent transparent/invisible rendering — see [CHECKLIST.md](CHECKLIST.md#6-css--styling-mismatches) |
| 7 | Placeholder & Stub Code | Incomplete implementations — see [CHECKLIST.md](CHECKLIST.md#7-placeholder--stub-code) |
| 8 | Language-Specific Gaps | Type system holes + AI-specific errors — see [LANGUAGE-SPECIFIC.md](LANGUAGE-SPECIFIC.md) |
| 9 | Fallow Static Analysis | Unused exports, dead code, circular deps, complexity, duplication — see [FALLOW-MAPPING.md](FALLOW-MAPPING.md) |

## Rules

- Only report things that **will break or produce incorrect results**
- Do NOT report: style, naming, formatting, performance suggestions, missing tests/docs
- Do NOT report: "consider using X instead of Y"
- If uncertain whether something is a real bug, investigate before reporting

## Fallow Integration

For JS/TS projects, run the `fallow` static analysis tool before manual review. This catches mechanical issues deterministically so the manual review can focus on semantic bugs.

### When to run

Run the fallow pre-check when the project contains a `package.json`, `tsconfig.json`, or `.ts`/`.js` files (JS/TS project detection). This is mandatory for all TypeScript and JavaScript projects.

If `npx fallow --version` fails, install fallow first: `npm install -D fallow`, then proceed with the audit.

### How to run

1. **Run the audit**: `npx fallow audit --format json --quiet`
2. **Get staged files**: `git diff --staged --name-only`
3. **Apply auto-fixes to staged files only** — see [FALLOW-MAPPING.md](FALLOW-MAPPING.md) for fix types and the staged/unstaged filtering logic
4. **Re-run audit** to verify fixes: `npx fallow audit --format json --quiet`
5. **Collect non-auto-fixable findings** (and unstaged auto-fixable findings) for the manual review report

### Edge cases

- **Audit JSON parse error**: Log a warning, skip fallow, proceed with manual review only
- **No staged files**: Treat all findings as report-only (no modifications)
- **Fallow not installed**: Install via `npm install -D fallow`, then run the audit
