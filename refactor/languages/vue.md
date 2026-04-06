# Vue 3 Refactor Config

## Refactor Patterns

### Duplication
- **Duplicated reactive logic** — Multiple components with the same `ref` + `watch` + `computed` chain. Extract into a composable.
- **Repeated API call patterns** — Components that manually manage loading/error states for the same endpoint. Consolidate into a shared query composable.
- **Copy-pasted template blocks** — Identical or near-identical template fragments across components. Extract into a shared component with props/slots.
- **Duplicated emit-and-fetch cycles** — Multiple components that emit an event, then refetch data in the same way. Centralize via a composable or event bus.

### Structural
- **God component** — A single component that handles routing, data fetching, state management, and rendering. Split into container + presentational components.
- **Misplaced composable** — A composable defined inside a component file or in the wrong module. Move to the appropriate `composables/` directory.
- **Smart leaf component** — A bottom-level component that fetches data, calls mutations, or manages complex state. Push data-fetching and mutations up to the nearest view/container component; the leaf should receive props and emit events.
- **Cross-module imports** — A component importing directly from another module's internals instead of through a shared layer or public API.
- **Route-level logic in components** — Route guards, redirects, or param parsing embedded in component logic. Move to route-level guards or middleware.
- **Flat component directory** — A module with 10+ components in a single folder with no grouping. Organize by feature or concern.

### Idiom
- **Options API in Composition API codebase** — Components using `data()`, `methods`, `computed` when the project uses `<script setup>`. Migrate to Composition API.
- **Manual v-model prop/emit** — Using `props` + `emit('update:modelValue')` when `defineModel()` is available (Vue 3.4+).
- **Ref unwrapping confusion** — Using `.value` in templates or forgetting it in script. Ensure consistent ref usage.
- **Watchers instead of computed** — Using `watch` to derive state that could be a `computed` property.
- **Missing `defineProps` destructure** — Destructuring props without using `defineProps` with reactive destructure (Vue 3.5+).
- **`reactive()` for simple state** — Using `reactive()` where a `ref()` would be simpler and more portable.
- **Unordered imports** — Imports not following the convention: (1) classes/utilities, (2) types, (3) `.vue` components last. Reorder for consistency.
- **Non-colocated emits** — Using `emit()` without `defineEmits` type declaration.
- **Template ref typing** — Untyped template refs (`ref(null)` instead of `useTemplateRef` or typed `ref<InstanceType<typeof Component> | null>(null)`).

### Complexity
- **Long `<script setup>` blocks** — Script blocks exceeding ~150 lines. Extract logic into composables.
- **Deeply nested v-if/v-for** — Templates with 3+ levels of conditional/loop nesting. Simplify with computed properties or child components.
- **Mutations in leaf components** — Bottom-level components that call API mutations, update stores, or perform side effects directly. Leaf components should be display-only; mutations and updates belong in top-level (view/container) components and are passed down as callbacks or handled via emits.
- **Prop drilling through 3+ levels** — Data passed through multiple intermediate components. Consider provide/inject or a store.
- **Overloaded computed properties** — A single `computed` doing data transformation, filtering, and formatting. Break into steps.

### Dead Code
- **Unused props/emits** — Props declared in `defineProps` but never used in template or script. Emits declared but never called.
- **Orphaned components** — Components not imported or referenced anywhere in the project.
- **Dead route entries** — Routes defined but never linked to or navigated to.
- **Unused composable exports** — Composables exporting functions that no component consumes.

## Language Idioms

### Prefer
- Import order: (1) classes/utilities, (2) types, (3) `.vue` components — at the bottom
- SFC block attributes: value attributes first, boolean attributes last — e.g., `<script lang="ts" setup>`, `<style lang="scss" scoped>`, NOT `<style scoped lang="scss">`
- `<script setup>` over `<script>` with `setup()` function
- `defineModel()` over manual prop + emit for v-model
- `computed()` over `watch` for derived state
- `ref()` over `reactive()` for primitive and simple object state
- `useTemplateRef()` or typed refs over untyped `ref(null)`
- Named exports from composables over default exports
- `defineProps` with TypeScript type literals over runtime prop validation
- `defineEmits` with TypeScript tuples over string-only emit declarations
- Async components with `defineAsyncComponent` for heavy/rarely-used components
- `toValue()` over manual `.value` access in composables that accept MaybeRef params

### Avoid
- Mutating props — even when technically possible via object references
- `this` — irrelevant in Composition API, signals Options API holdover
- Global event bus — use provide/inject, stores, or emits instead
- Mixins — use composables for logic reuse
- `$refs` access from parent to child for data flow — use props/emits
- Synchronous `computed` with side effects — computed should be pure

## Structural Conventions

### Module Organization
```
src/modules/<name>/
├── api/              # API service functions and query composables
├── components/       # Module-scoped components
├── composables/      # Module-scoped composables
├── views/            # Route-level page components
├── routes.ts         # Module route definitions
├── types.ts          # Module-scoped types (if needed)
└── index.ts          # Public API (if consumed by other modules)
```

### Component Responsibility Hierarchy
- **Views (top-level)** — Own data fetching, mutations, and state management. Wire child components together.
- **Container components (mid-level)** — May compose child components and manage local UI state, but delegate mutations upward via emits or receive mutation callbacks as props.
- **Leaf components (bottom-level)** — Display only. Receive data via props, emit user interactions. No API calls, no store access, no side effects.

### Rules
- Each feature module is self-contained with its own routes, API layer, and components
- Shared components live in `src/components/`, not inside a feature module
- Composables used by a single module live in that module; shared ones go in `src/composables/`
- Views are route-level only — they wire together components and data, not render complex UI directly
- Route files export an array of `RouteRecordRaw` and are imported by the root router
- Stores (Pinia) live in `src/stores/` at the app level unless module-scoped
