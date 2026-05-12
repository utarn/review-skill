# Review Checklist

## 1. Cross-Boundary Contract Mismatches

The most critical category — these silently fail at runtime.

- **Command/function name mismatches**: Caller invokes `foo()` but callee is registered as `bar()`. Check every `invoke()`, `fetch()`, RPC call, API endpoint, event name, message queue subject against what the receiver actually exposes.
- **Parameter name mismatches**: Caller sends `{ userId: 123 }` but receiver expects `user_id`. Check snake_case vs camelCase conversion at every boundary (HTTP APIs, IPC, serialization, database queries).
- **Parameter order**: Caller passes `(path, options)` but callee expects `(options, path)`.
- **Parameter type mismatches**: Caller sends a `string` but receiver expects `number`, or caller sends a flat value but receiver expects a wrapped object like `{ value: x }`.
- **Return type mismatches**: Receiver returns a plain `string` but caller treats it as an object `{ data_path: string }`. Check every `.field` access on a return value against what the function actually returns.
- **Missing or extra parameters**: Caller passes fewer or more arguments than the callee accepts.

## 2. Serialization & Deserialization Gaps

- **Field name casing**: Does `snake_case` in Rust/Python become `camelCase` in JSON? Verify `#[serde(rename_all)]` / `@SerializedName` / `json:"fieldName"` annotations match what the other side expects.
- **Optional vs required fields**: One side has `Option<T>` / `Optional<T>` / `field?: type` but the other side assumes the field is always present.
- **Enum/tagged union serialization**: Check that discriminant field names (`"type"`, `"kind"`) and variant names match between producer and consumer.
- **Base64 encoding layers**: Check whether data is already base64-encoded (e.g., Gmail API `raw` field) before being re-encoded.

## 3. Logic Bugs

- **Double counting**: Counter incremented both inside a loop/branch AND outside it. Look for `count += 1` or `i++` in both handler and post-loop block.
- **Off-by-one errors**: Loop bounds, array indices, pagination tokens (`page <= total` vs `page < total`).
- **Conditions always true/false**: Stale booleans, redundant checks after break/return, `if x != null` when x was already dereferenced above.
- **Unreachable code**: Code after `return`, `break`, `throw`, `continue`.
- **Dead flags / no-op patterns**: Lock/semaphore acquired in sequential code (no effect). Retry loop that never retries. Flag set but never checked.
- **Shadowed variables**: Inner scope declares variable with same name as outer scope, causing outer one to be used unintentionally.

## 4. Property & Method Access Errors

- **Accessing property on wrong type**: `result.data_path` when `result` is a `string`, not an object. `response.data.id` when response wraps data differently.
- **Accessing array index that may not exist**: `items[0].name` without checking `items.length`.
- **Calling method on possibly-null value**: `list.sort(...)` when `list` might be `null`/`None` rather than `[]`.
- **Wrong `this` binding**: Callback passed as `obj.method` loses `this` context. Arrow function vs regular function in classes.
- **Optional chaining misuse**: `user?.address.street` — the `?.` stops at `user` but `street` still throws if `address` is null.

## 5. Async & Concurrency Bugs

- **Sequential code that should be concurrent** (or vice versa): Loop acquires semaphore but processes items one at a time, making semaphore useless.
- **Race conditions**: Shared mutable state accessed without locks. Two async functions writing to same file/variable without coordination.
- **Await missing**: `asyncFn()` without `await` — promise created but errors silently swallowed, execution continues before completion.
- **Cancellation token checked too late**: Cancel flag checked at top of loop but expensive operation inside doesn't check it.
- **Resource leaks**: File handles, database connections, HTTP clients opened but not closed in error paths.

## 6. Placeholder & Stub Code

- **Functions that build something but never use it**: Constructs a map/list/object but never calls the function that would use it (e.g., builds a label map but never calls `create_label()`).
- **TODO/FIXME/placeholder values**: Hardcoded strings like `"NEW_"`, `"placeholder"`, empty catch blocks, `// TODO: implement`.
- **Unused imports and dead code**: Functions imported or defined but never called. Variables assigned but never read.
