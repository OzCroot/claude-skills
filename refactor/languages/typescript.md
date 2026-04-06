# TypeScript Refactor Config

## Refactor Patterns

### Duplication
- **Repeated type narrowing** — Multiple places doing the same `if ('kind' in obj)` or `instanceof` checks. Extract into a type guard function.
- **Duplicate interface shapes** — Two or more interfaces with near-identical fields. Extract a shared base or use intersection types.
- **Copy-pasted utility functions** — Same transformation logic in multiple files. Consolidate into a shared utility.
- **Repeated error handling boilerplate** — Try/catch blocks with the same structure across multiple call sites. Extract into a higher-order function or shared handler.

### Structural
- **Barrel file bloat** — `index.ts` re-exporting everything from a directory, including internals. Only export the public API.
- **Circular dependencies** — Module A imports from B which imports from A. Restructure to break the cycle, often by extracting shared types.
- **Types mixed with runtime code** — Interfaces and type aliases defined in files that also contain functions/classes. Colocate types with their consumers or extract to a `types.ts`.
- **God file** — A single file with 500+ lines handling multiple concerns. Split by responsibility.
- **Flat utility directory** — A `utils/` folder with 20+ files and no grouping. Organize by domain.

### Idiom
- **`any` type usage** — Using `any` where `unknown`, a generic, or a specific type would work. `any` disables type checking.
- **Type assertions over narrowing** — Using `as Type` instead of type guards, discriminated unions, or `satisfies`.
- **Enum usage** — `enum` is not compatible with `erasableSyntaxOnly`. Replace with `as const` object + `keyof typeof` pattern: `const Status = { Active: 'active', Inactive: 'inactive' } as const; type Status = typeof Status[keyof typeof Status];`
- **Non-null assertion abuse** — Overusing `!` postfix instead of handling the `null`/`undefined` case or using optional chaining.
- **Redundant type annotations** — Explicitly annotating types that TypeScript already infers correctly. Let inference work.
- **`interface` where `type` suffices** — Using `interface` for object shapes that don't need inheritance. Prefer `type` by default; reserve `interface` for when `extends` or declaration merging is specifically needed.
- **Loose function signatures** — Functions accepting `object` or `Record<string, any>` when a specific type is known.
- **`.then()` chains** — Using `.then()/.catch()` instead of `async`/`await`. Always prefer `await` for readability and debuggability.
- **Mutable where immutable works** — Using `let` where `const` suffices, or mutable arrays where `as const` or `readonly` applies.

### Complexity
- **Overloaded function signatures** — A function with 4+ overloads that could be simplified with a discriminated union parameter or generics.
- **Deep generic nesting** — Types like `Promise<Result<Array<Map<string, T>>>>`. Introduce named type aliases for intermediate layers.
- **Long conditional chains** — `if/else if/else if` chains that could be a lookup map, switch, or strategy pattern.
- **God function** — A function exceeding ~60 lines. Extract sub-operations into named helper functions.

### Dead Code
- **Unused exports** — Functions, types, or constants exported but never imported anywhere.
- **Unreachable code after early returns** — Logic after a `return`, `throw`, or `process.exit` that never executes.
- **Unused function parameters** — Parameters accepted but never referenced in the function body.
- **Dead type definitions** — Interfaces or type aliases not referenced by any runtime code or other types.

## Language Idioms

### Prefer
- `unknown` over `any` for values of uncertain type
- Type guards and discriminated unions over type assertions
- `satisfies` operator for validating object shapes while preserving literal types
- `as const` object + `keyof typeof` pattern over `enum` (required by `erasableSyntaxOnly`)
- `type` over `interface` for object shapes unless `extends` inheritance is needed
- `async`/`await` over `.then()` chains
- `readonly` arrays and tuples where mutation is not needed
- `const` assertions for literal types (`as const`)
- Optional chaining (`?.`) over manual null checks
- Nullish coalescing (`??`) over logical OR (`||`) for default values
- Named return types on public API functions for documentation
- Generic constraints (`<T extends Base>`) over unconstrained generics

### Avoid
- `any` — almost always a better option exists
- `@ts-ignore` / `@ts-expect-error` without a comment explaining why
- Type assertions (`as`) when narrowing is possible
- `Function` type — use specific callable signatures
- `object` type — use `Record<string, unknown>` or a specific shape
- Namespace merging and declaration merging for application code
- Triple-slash directives — use module imports
- Default exports — named exports are more refactor-friendly and searchable
- `enum` — not compatible with `erasableSyntaxOnly`; use `as const` objects instead
- `interface` for non-inherited shapes — `type` is the default; `interface` only when `extends` is needed
- `.then()` / `.catch()` — use `async`/`await` with try/catch instead

## Structural Conventions

### File Organization
- One primary export per file (a function, a class, a type, or a closely related group)
- Types colocated with their primary consumer, or in a sibling `types.ts` if shared
- Constants in a `constants.ts` file within the relevant module
- Utility functions grouped by domain, not dumped in a single `utils.ts`

### Naming
- Files: `camelCase.ts` for utilities, `PascalCase.ts` for classes/components
- Types/Interfaces: `PascalCase`
- Functions/variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE` for true constants, `camelCase` for derived values
- Type parameters: Single uppercase letter (`T`, `K`, `V`) for simple generics, descriptive names (`TResult`, `TInput`) for complex ones

### Module Boundaries
- Modules expose a public API through explicit exports — do not reach into internal files
- Shared types used across modules live in a top-level `types/` directory
- Avoid circular dependencies — if two modules need each other's types, extract shared types to a common location
