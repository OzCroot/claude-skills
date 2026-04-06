# Tailwind CSS v4 Refactor Config (Configless)

> Assumes Tailwind v4 with CSS-first configuration (no `tailwind.config.*`).
> Theme is defined via `@theme` in CSS. Detection: presence of `@tailwind` or `@theme` in CSS files, or `@import "tailwindcss"` in the main stylesheet.

## Refactor Patterns

### Duplication
- **Repeated utility strings** — The same long class string (5+ utilities) appearing in multiple templates. Extract into a component, `@apply` directive, or shared class.
- **Duplicate responsive breakpoint chains** — Identical `sm:`, `md:`, `lg:` patterns across multiple elements. Consider a shared component or CSS layer.
- **Copy-pasted color/spacing values** — Arbitrary values like `text-[#1a2b3c]` or `p-[13px]` repeated across files instead of using theme tokens.

### Structural
- **Arbitrary values instead of theme tokens** — Using `bg-[#ff5733]` or `w-[247px]` when a theme token or design system value exists. Map to the token system.
- **Inline styles competing with Tailwind** — `:style` bindings doing what Tailwind utilities already handle (color, spacing, layout).
- **CSS file with utilities that should be Tailwind** — Custom CSS classes that replicate what Tailwind provides out of the box.
- **Legacy `tailwind.config.*` file** — v4 uses CSS-first configuration via `@theme` blocks. A JS/TS config file is a migration smell unless explicitly needed for plugin compatibility.
- **Unused `@apply` abstractions** — CSS classes created with `@apply` that are only used once — inline the utilities instead.

### Idiom
- **Flex/grid without gap** — Using `margin` or `space-x/y` utilities when `gap` is cleaner for flex/grid layouts.
- **Fixed widths on fluid containers** — Using `w-[500px]` instead of responsive utilities like `max-w-lg` or percentage-based widths.
- **Overqualified selectors** — Nesting utilities inside custom CSS selectors when Tailwind's variant system (`hover:`, `group-hover:`, `data-[state=]:`) handles it.
- **v3-era syntax** — Using deprecated utilities, `@variants`, `@responsive`, or `@screen` directives. v4 uses native CSS features and `@variant` instead.
- **`!important` modifier overuse** — Using `!` prefix on utilities to win specificity battles instead of fixing the cascade.
- **Manual dark mode classes** — Writing separate dark mode CSS instead of using Tailwind's `dark:` variant tied to the theme system.
- **Redundant utility pairs** — `flex flex-row` (row is default), `block` on divs (already block), `static` (already default position).

### Complexity
- **Utility soup** — A single element with 15+ utility classes that's hard to read. Break the element into sub-components or extract the visual pattern.
- **Deeply conditional class bindings** — Complex ternary chains in `:class` bindings assembling utility strings. Extract into a computed property or use a class variance utility (e.g., `cva`, `class-variance-authority`).
- **Competing responsive overrides** — An element with utilities at every breakpoint that override each other in confusing ways. Simplify the responsive strategy.

### Dead Code
- **Unused custom utilities** — `@layer utilities` definitions not referenced in any template.
- **Orphaned `@apply` classes** — CSS classes using `@apply` that are never applied to any element.
- **Stale `@theme` values** — Custom colors, spacing, or font values in `@theme` blocks that nothing references.
- **Commented-out utility classes** — Old class strings left in comments that add noise.

## Language Idioms

### Prefer
- Theme tokens (CSS custom properties) over hardcoded color values
- `gap` over `space-x`/`space-y` for flex and grid layouts
- Responsive utilities (`sm:`, `md:`, `lg:`) over media query CSS
- `dark:` variant over manual theme-switching CSS
- Tailwind's built-in design scale (`p-4`, `text-sm`, `rounded-lg`) over arbitrary values
- `group` and `peer` modifiers over JavaScript-driven hover/focus state
- `size-*` shorthand over separate `w-*` + `h-*` when width equals height
- Logical properties (`ps-*`, `pe-*`, `ms-*`, `me-*`) for RTL-safe spacing
- `data-[state=]:` variants for component state styling over dynamic class strings

### Avoid
- Arbitrary values (`[#hex]`, `[13px]`) when a design token exists
- `@apply` for one-off styles — inline the utilities
- `!important` modifier — fix specificity issues at the source
- Mixing Tailwind utilities with custom CSS for the same property on the same element
- Pixel-based breakpoints in arbitrary variants when Tailwind's breakpoint scale matches
- Over-abstracting visual patterns into CSS classes too early — components are usually the right boundary

## Structural Conventions

### Token System (v4 CSS-first)
- Theme is defined via `@theme` blocks in CSS — no JS config file
- `@theme` generates CSS custom properties automatically (`--color-*`, `--spacing-*`, etc.)
- All design primitives (colors, spacing, typography) live in `@theme` and are the single source of truth
- When a project has additional CSS custom property tokens (e.g., for multi-theme support), utilities should reference those tokens via `var()` or mapped `@theme` values

### Class Organization in Templates
- Layout utilities first (`flex`, `grid`, `block`, positioning)
- Sizing next (`w-`, `h-`, `size-`, `max-w-`)
- Spacing (`p-`, `m-`, `gap-`)
- Typography (`text-`, `font-`, `leading-`, `tracking-`)
- Visual (`bg-`, `border-`, `rounded-`, `shadow-`)
- Interactive states last (`hover:`, `focus:`, `disabled:`, `dark:`)

### When to Extract
- 3+ elements with the same 5+ utility string → extract to a component
- A utility string used across files with slight variations → parameterize as a component with props
- A complex `:class` binding with 3+ conditions → extract to a computed or `cva` pattern
