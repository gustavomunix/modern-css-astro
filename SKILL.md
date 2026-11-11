---
name: modern-css-astro
description: Opinionated rules for writing CSS in Astro projects. Use whenever CSS is written, refactored or reviewed in an Astro site, when styling an .astro component or page, adding a global stylesheet, scoping styles, building layouts, fluid type and spacing, colors, dark mode, motion, or when asked to "style this component", "make this responsive", "clean up the CSS" or "set up base styles" in Astro. Adapts the moderncss/skills rules to Astro's scoped styles, cascade order and build output.
license: MIT
compatibility: Verified on Astro 7.4. Requires the modern-web-guidance skill (npx skills add GoogleChrome/modern-web-guidance).
metadata:
  tags: astro, css, modern-css, baseline, cascade-layers, scoped-styles, oklch, logical-properties, container-queries
  derived-from: https://github.com/moderncss/skills
---

# Modern CSS for Astro

Rules for modern CSS in Astro. They keep the rules of the original `modern-css` skill and change the ones where Astro already does the job or where its build changes the outcome.

These rules sit on top of [Modern Web Guidance](https://developer.chrome.com/docs/modern-web-guidance), which explains what each CSS feature does and how to fall back. Where the two disagree, these rules win.

Browser support policy: features within Baseline Newly Available are used natively, without `@supports` fallbacks, as progressive enhancement.

## What changes in Astro

| Original rule | In Astro |
|---|---|
| `@scope` for every component | Dropped. A component `<style>` is already scoped. Use `:global()` or a parent-owned style for content the component does not own. |
| Every rule in a `@layer` | Kept, and stricter: scoped component styles are unlayered unless you wrap them, and unlayered rules beat every layer. |
| `1rem` is 16px | Read the project's root font size first. If `html` is `62.5%`, `1rem` is 10px. |
| Linter enforces most rules | `stylelint-config-modern` only covers logical properties, `display` notation and `oklch`. `@layer`, nesting, range queries and fluid `clamp()` are review rules. |

## Rules

### Architecture

#### Scoping (`<style>` in `.astro`)

- **Rule**: Put component styles in the component's own `<style>`. Do not add `@scope` inside it.
- **Constraint**: Avoid component CSS in global files, `<style is:global>` for component styles, and `@scope` in `.astro` files.
- **Rationale**: Astro rewrites selectors with a `data-astro-cid-*` attribute, so scope is already enforced. `@scope` only duplicates it.
- **Known edge**: A nested selector such as `.card { h2 { } }` compiles to `.card[data-astro-cid-x] { & h2 { } }`. Only the outer selector is scoped, so `h2` also matches headings inside child components and slots. Use a class on the element, or `:global()` with intent, when that matters.
- **Reference**: [references/astro-styling.md](references/astro-styling.md).

#### Organizing styles (`@layer`)

- **Rule**: Declare the layer order once, in a stylesheet imported first by the base layout, and put every rule in a layer, including rules inside a component `<style>`.
- **Constraint**: Avoid unlayered rules, including unlayered component styles.
- **Rationale**: Layer order alone then decides conflicts. Verified: `@layer` inside an Astro `<style>` survives the build and keeps the scope attribute.
- **Example**:

```css
@layer reset, tokens, base, components, utilities;
```

```astro
<style>
  @layer components {
    .card {
      padding: clamp(1rem, 0.5rem + 2cqi, 2rem);

      &:hover {
        box-shadow: var(--shadow-raised);
      }
    }
  }
</style>
```

- **Check**: After a build, confirm the order in `dist/` before relying on it. Astro splits CSS into chunks, and layer order follows the first appearance in the cascade.

#### Scope strategy (`scopedStyleStrategy`)

- **Rule**: Set `scopedStyleStrategy: 'where'` in `astro.config.mjs` when the project uses layers.
- **Rationale**: Per the Astro docs, `'where'` adds no specificity, so layers and selectors decide conflicts instead of the scope attribute.
- **Constraint**: Keep the default `'attribute'` if existing global CSS relies on scoped styles winning on specificity. Change it in its own commit and compare pages.

#### Nesting rules and at-rules (`&`)

- **Rule**: Nest rules and at-rules inside their parent rule. Write `&` for attached selectors (`&:hover`) and omit it for combinators (`b`, not `& b`).
- **Constraint**: Avoid unnested selectors such as `a {} a:hover {}`.

#### Ranged queries (`20em < width <= 40em`)

- **Rule**: Use range syntax for size queries, with ranges that do not overlap.
- **Constraint**: Avoid `min-width`/`max-width` and overlapping ranges.

### Layout

#### Flow-relative layout

- **Rule**: Use flow-relative properties (`padding-block-start`, `inset-inline`, `inline-size`), units (`cqi`, `vi`) and keywords (`text-align: start`).
- **Constraint**: Avoid physical equivalents (`padding-top`, `width`, `vw`, `text-align: left`).
- **Enforced by**: `stylelint-config-modern`.

#### Two-value display

- **Rule**: Use `display: block flex`, `inline grid`, `inline flow-root`.
- **Constraint**: Avoid `flex`, `grid`, `inline-block`, `inline-flex` as single values.
- **Enforced by**: `stylelint-config-modern`.

#### Fluid spacing (`cqi`)

- **Rule**: Size spacing with container units, clamped where it needs a floor or ceiling. Give the container `container: name / inline-size`.
- **Constraint**: Avoid fixed `px`/`rem` spacing, except for hairlines and borders.
- **Astro note**: `container-type` goes on the component root, so each Astro component declares its own container.

#### Auto-growing text areas (`field-sizing: content`)

- **Rule**: Use `field-sizing: content` on text areas with an explicit `inline-size` and `min-block-size`/`max-block-size` in `lh`.

### Typography

#### Fluid type sizes (`clamp()`)

- **Rule**: Use `clamp()` with container units and a `rem` term in the central value. The ratio grows with the container, for example 1.125 on narrow and 1.25 on wide containers.
- **Constraint**: Avoid fixed font sizes and central values without `rem`.
- **Root size**: Convert with the project's root. At `html { font-size: 100% }`, `1rem` is 16px and the original examples apply as written. At `62.5%`, `1.6rem` is 16px, so scale every `rem` value by 0.625 or the type shrinks by 37.5%.
- **New projects**: Keep the root at `100%`. A `62.5%` root is a legacy convention and zoom still works, but every guide, example and token tool assumes 16px.

#### Widow and orphan words (`text-wrap`)

- **Rule**: `text-wrap: balance` on headings, `text-wrap: pretty` on other text.

### Colors

#### Perceptual uniform lightness (`oklch()`)

- **Rule**: Use `oklch()` for all colors.
- **Constraint**: Avoid hex, `rgb()`, `hsl()`.
- **Enforced by**: `stylelint-config-modern`.

#### Respecting color preferences (`light-dark()`)

- **Rule**: Use `light-dark()` for every color that differs between schemes, and set `color-scheme` on `:root`.
- **Constraint**: Avoid `prefers-color-scheme` blocks that redeclare colors.

#### Relative color functions

- **Rule**: Derive related colors with `oklch(from var(--primary) l c h)` or `color-mix(in oklab, ...)`.
- **Constraint**: Avoid hardcoding colors that relate to other colors.

### Motion

#### Respecting motion preferences

- **Rule**: Wrap large animations in `@media (prefers-reduced-motion: no-preference)`.
- **Constraint**: Avoid `prefers-reduced-motion: reduce` overrides.
- **JS animation**: Prefer CSS and the Web Animations API (`element.animate()`) over an animation library. Check the same media query in script with `matchMedia`.

## Applying these rules

When asked to author, refactor or review CSS in an Astro project:

1. Read `astro.config.mjs` for `scopedStyleStrategy`, `build.inlineStylesheets` and integrations, and find the root `font-size` before writing any `rem`.
2. Decide the home of each rule: component `<style>` for component styles, one global layered stylesheet for tokens, reset and base.
3. Apply the rules above to every touched rule, not only the lines you edit.
4. Retrieve the Modern Web Guidance guides for the task and apply them within these rules.
5. Run `pnpm lint:css` (or the project's equivalent) on the changed files and fix what it reports.
6. Build, then inspect the built CSS for layer order and the scope attribute. See [references/audit.md](references/audit.md).

## References

- [references/astro-styling.md](references/astro-styling.md): scoping, `:global()`, passing classes to children, cascade order, import leakage, CSS bundling.
- [references/astro-delivery.md](references/astro-delivery.md): images, fonts, scripts, prefetch and structure choices that affect CSS and Core Web Vitals.
- [references/audit.md](references/audit.md): how to verify the result, and which companion skills to use.
- [references/sources.md](references/sources.md): where each rule came from.
