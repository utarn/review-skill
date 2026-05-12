# Language-Specific Checks

Apply the section matching the language(s) in the codebase.

## JavaScript / TypeScript

- `any` type that bypasses checking
- Type assertions (`as X`) that lie about the actual shape
- Interface field mismatch between what's declared and what's used at runtime

## Python

- Mixing `str` and `bytes` (especially in Python 2→3 migration code)
- Mutable default arguments: `def foo(items=[])` — the list is shared across calls
- Missing `self` or wrong decorator order (`@staticmethod` vs `@classmethod` vs instance method)

## Java / Kotlin / C# (.NET)

- Null dereference without check (NPE)
- Using `==` instead of `.equals()` for string/value comparison
- Integer division when float was intended
- Generic type erasure causing runtime ClassCastException

## PHP

- Loose type comparisons: `==` vs `===` (`"0" == false` is true)
- Array vs object access confusion (`$arr->prop` vs `$arr['prop']`)
- Undefined array key access without null coalescing (`??`)

## Rust

- `unwrap()` on `Option`/`Result` that can be `None`/`Err` at runtime
- Holding a lock across an `.await` point (deadlock risk)
- Integer overflow in release mode (`debug_assert!` only runs in debug)

## Go

- Unexported struct fields silently ignored by JSON marshal/unmarshal
- Goroutine leaks: goroutines that never return (missing context cancellation)
- Map concurrent access without sync.Map or mutex
- Defer in loop: `defer file.Close()` inside a loop holds files open until function returns
