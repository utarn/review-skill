# Language-Specific Checks

Apply the section matching the language(s) in the codebase. Each section covers both **core language pitfalls** and **AI-specific error patterns** common to that ecosystem.

## Python

Python's heavy reliance on third-party libraries means AI errors often revolve around the ecosystem rather than syntax.

**Ecosystem / AI-specific:**
- **Hallucinated methods**: Invented functions for libraries like pandas, numpy, or requests. Code looks perfectly Pythonic and logical, but the specific method simply doesn't exist in the actual library. Verify every library call against actual docs.
- **Version confusion**: Mixing older library syntax with newer APIs, or struggling with the transition between synchronous code and modern `asyncio` patterns (e.g., using `async for` on a non-async iterator).
- **Silent indentation drops**: In long scripts, a block of code may be misaligned, accidentally pulling a statement out of a `for` loop or `if` block. Verify indentation matches the intended scope.

**Core language:**
- Mixing `str` and `bytes` (especially in Python 2→3 migration code)
- Mutable default arguments: `def foo(items=[])` — the list is shared across calls
- Missing `self` or wrong decorator order (`@staticmethod` vs `@classmethod` vs instance method)

## JavaScript / TypeScript

The flexibility of JavaScript and the strictness of TypeScript create two different sets of problems.

**Ecosystem / AI-specific:**
- **Environment confusion**: Mixing browser APIs with Node.js APIs. Using `fs` (file system) in a React frontend component, or accessing `window`/`document` in a backend Express server. Check every API call against the actual runtime environment.
- **TypeScript `any` abuse**: When struggling to resolve a complex type definition, AI may lazily fall back to `any`, defeating the purpose of TypeScript. Also watch for hallucinated properties on well-typed objects — properties that look correct but don't exist on the declared type.

**Core language:**
- `any` type that bypasses checking
- Type assertions (`as X`) that lie about the actual shape
- Interface field mismatch between what's declared and what's used at runtime

## C++

C++ is incredibly complex, and AIs often struggle with memory management and historical baggage.

**Ecosystem / AI-specific:**
- **Modern vs. legacy mix-ups**: C++ has evolved massively (C++11, 14, 17, 20). Mixing legacy C-style pointers with modern smart pointers (`std::unique_ptr`), creating inconsistent codebases. Check that pointer management style is uniform.
- **Memory leaks**: Forgetting to cleanly manage manual allocations (`new` and `delete`), or misinterpreting when a destructor will be called. Every `new` must have a matching `delete` or be wrapped in a smart pointer.
- **Header/include errors**: Forgetting to `#include` the standard library headers required for the functions just written. Code compiles in one TU but fails in another.

**Core language:**
- Use-after-free and dangling references
- Undefined behavior from uninitialized variables
- Buffer overflows in C-style arrays and pointer arithmetic

## Rust

Rust's strict compiler means AI-generated code often fails to compile — the errors are caught, but the code is still wrong.

**Ecosystem / AI-specific:**
- **Borrow checker violations**: The #1 AI error in Rust. Logically sound code that fails to compile because it violates Rust's strict ownership, borrowing, or lifetime rules. Check that every reference's lifetime is valid for its use.
- **Overcomplicating lifetimes**: When trying to fix a perceived scope issue, AI may generate chaotic, overly complex lifetime annotations (`<'a, 'b>`) that confuse the compiler further. Look for lifetime annotations that can be simplified or removed.

**Core language:**
- `unwrap()` on `Option`/`Result` that can be `None`/`Err` at runtime
- Holding a lock across an `.await` point (deadlock risk)
- Integer overflow in release mode (`debug_assert!` only runs in debug)

## Java & C# (.NET)

Both are heavily structured, object-oriented languages with massive enterprise frameworks.

**Ecosystem / AI-specific:**
- **Framework hallucinations**: In Java (Spring Boot) or C# (.NET), AI may invent annotations or decorators that sound plausible but don't exist in the framework. Verify every annotation/attribute against the actual framework version.
- **Contextual bloat**: AI tends to write overly verbose boilerplate — sometimes generating entirely new classes or interfaces when a simple built-in method would suffice. This isn't a style issue; the generated classes may have bugs themselves.
- **C# LINQ errors**: Non-existent LINQ extension methods, or deeply nested LINQ queries that perform poorly at scale (N+1 queries, full-table scans).

**Core language:**
- Null dereference without check (NPE)
- Using `==` instead of `.equals()` for string/value comparison (Java)
- Integer division when float was intended
- Generic type erasure causing runtime ClassCastException

**C# Entity Framework — Missing migrations:**
- **Entity changes without migration**: When entity classes (properties added/removed/renamed, new entities, relationship changes) are modified but no corresponding EF Core migration has been created. Check that for every change to files under `Entities/`, `Models/`, or `Domain/` there is a matching migration file under `Migrations/`. Run `dotnet ef migrations list` and compare the latest migration snapshot against the current entity definitions. A missing migration will cause runtime `InvalidOperationException` or schema mismatch errors at startup or during database operations.
- **Orphaned migrations**: Migrations that have been created but never applied to the database (`dotnet ef migrations list` shows pending migrations). These accumulate and can cause conflicts when new migrations are added on top.
- **Model snapshot drift**: The `Migrations/{ContextName}ModelSnapshot.cs` file should match the sum of all applied migrations. If it's out of sync (e.g., manually edited), `dotnet ef migrations add` will generate incorrect migrations.

## Go (Golang)

Go is minimalist, but its specific approach to concurrency and errors trips up AI models.

**Ecosystem / AI-specific:**
- **Concurrency deadlocks**: Mishandling goroutines and channels. A channel that never closes, or a `sync.WaitGroup` that never calls `wg.Done()`, causing the program to hang. Verify every goroutine has a termination path.
- **Ignoring idioms**: Go relies on explicit error handling (`if err != nil`). AI may bypass this with "clever" one-liners or use `panic` improperly, which goes against standard Go practices and can crash the program unexpectedly.

**Core language:**
- Unexported struct fields silently ignored by JSON marshal/unmarshal
- Goroutine leaks: goroutines that never return (missing context cancellation)
- Map concurrent access without `sync.Map` or mutex
- Defer in loop: `defer file.Close()` inside a loop holds files open until function returns

## PHP

- Loose type comparisons: `==` vs `===` (`"0" == false` is true)
- Array vs object access confusion (`$arr->prop` vs `$arr['prop']`)
- Undefined array key access without null coalescing (`??`)
