# Rust Refactor Config

## Refactor Patterns

### Duplication
- **Repeated match arms** — Multiple `match` expressions with near-identical arm patterns. Extract into a shared function or use a macro.
- **Copy-pasted error handling** — The same `map_err` / `unwrap_or_else` chain in multiple call sites. Extract into a trait impl, a helper, or use `?` with `From`.
- **Duplicate struct construction** — Building the same struct with default-like patterns in multiple places. Implement `Default` or a builder.
- **Repeated iterator chains** — Identical `.iter().filter().map().collect()` pipelines. Extract into a named function or method.

### Structural
- **Monolithic `lib.rs` / `main.rs`** — All logic in a single file. Split into modules by responsibility.
- **Module with too many `pub` items** — Over-exposed internals. Narrow visibility to `pub(crate)` or `pub(super)` where possible.
- **Circular module dependencies** — Modules that import from each other. Restructure by extracting shared types into a parent or sibling module.
- **Misplaced type definitions** — Types defined far from where they're used. Colocate types with their primary consumers.
- **God module** — A module with 500+ lines handling multiple concerns. Split by responsibility.
- **Orphaned helper modules** — Utility modules that exist only to serve one consumer. Inline or colocate.

### Idiom
- **`unwrap()` / `expect()` in library code** — Panicking where `Result` or `Option` propagation with `?` is appropriate.
- **Manual `From` conversions** — Explicit conversion functions where `impl From<T> for U` (and using `.into()`) is idiomatic.
- **Nested `match` instead of combinators** — Deep match nesting where `.map()`, `.and_then()`, `.unwrap_or()` on `Option`/`Result` is cleaner.
- **String-typed errors** — Returning `String` or `Box<dyn Error>` where a typed error enum with `thiserror` is more informative.
- **Clone-heavy code** — Unnecessary `.clone()` calls where borrowing, `Cow`, or restructuring ownership is possible.
- **Manual iterator loops** — `for` loops building a `Vec` that could be an iterator chain with `.collect()`.
- **Index-based access in loops** — `for i in 0..vec.len()` + `vec[i]` instead of `for item in &vec` or `.iter().enumerate()`.
- **Stringly-typed APIs** — Functions accepting `&str` for values from a known set. Use an enum.
- **`Box<dyn Trait>` where generics work** — Dynamic dispatch where static dispatch (generics) would be zero-cost and sufficient.
- **Redundant closures** — `.map(|x| foo(x))` instead of `.map(foo)` when the signature matches.
- **`if let` chains** — Multiple `if let Some(x)` / `if let Ok(x)` that could use `let-else` or early returns.

### Complexity
- **Functions over ~60 lines** — Long functions that handle multiple steps. Extract sub-operations into named helpers.
- **Deeply nested control flow** — 4+ levels of nesting from `if`, `match`, `for`, `while`. Flatten with early returns, `?`, or guard clauses.
- **Trait with too many methods** — A trait requiring 10+ method implementations. Consider splitting into smaller, focused traits.
- **Complex generic bounds** — Functions with 4+ generic parameters and `where` clauses spanning multiple lines. Introduce trait aliases or restructure.
- **Builder pattern without validation** — Builders that silently use defaults for required fields. Use typestate or validate at `.build()`.

### Dead Code
- **Unused `pub` functions** — Public functions not called from any other module or binary.
- **Dead feature flags** — `#[cfg(feature = "...")]` blocks for features that are never enabled.
- **Unreachable match arms** — Arms that can never match due to prior arm coverage.
- **Unused struct fields** — Fields set during construction but never read.
- **Commented-out code blocks** — Old implementations left in comments.

## Language Idioms

### Prefer
- `?` operator over manual `match` on `Result`/`Option` for error propagation
- `impl From<T>` over manual conversion functions
- Iterator combinators over manual `for` loops when the chain is clear
- `thiserror` for library errors, `anyhow` for application errors
- `Cow<str>` over `String` when a function may or may not need to allocate
- `let-else` for early exits on pattern match failure
- `#[must_use]` on functions where ignoring the return value is likely a bug
- Tuple structs for newtypes (`struct UserId(u64)`)
- `Default` trait for structs with sensible defaults
- `pub(crate)` over `pub` for items not part of the public API
- `impl` blocks colocated with their type definition
- Derived traits (`Debug`, `Clone`, `PartialEq`) on most types unless there's a reason not to

### Avoid
- `unwrap()` in library code and non-test production code
- `clone()` as a first resort for borrow checker issues — understand the ownership first
- `String` as an error type
- `Box<dyn Any>` — almost always a design smell
- Excessive `Arc<Mutex<T>>` — consider message passing or restructuring ownership
- `unsafe` without a `// SAFETY:` comment explaining the invariant
- Wildcard imports (`use module::*`) except in test modules and preludes
- `.to_string()` on string literals — use `.into()` or keep as `&str`

## Structural Conventions

### Project Layout
```
src/
├── main.rs           # Entry point — thin, delegates to lib
├── lib.rs            # Public API, module declarations
├── config.rs         # Configuration types and loading
├── error.rs          # Error types (or errors/)
├── types.rs          # Shared types (or split by domain)
└── <domain>/
    ├── mod.rs         # Module root — re-exports public API
    ├── types.rs       # Domain-specific types
    └── *.rs           # Implementation files
```

### Rules
- `main.rs` should be thin — parse args, initialize, call into `lib.rs`
- Each module has a clear responsibility; a module doing two things should be split
- Public API is explicit — don't re-export internal implementation details
- Types used across modules live at the crate root or in a `types` module
- Error types get their own module, using `thiserror` derive macros
- Tests live in a `#[cfg(test)] mod tests` at the bottom of the file they test, or in a `tests/` directory for integration tests
- Keep `use` statements organized: std → external crates → crate modules
