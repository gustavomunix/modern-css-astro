---
name: modern-css-astro
description: Opinionated rules for writing CSS in Astro projects. Use whenever CSS is written, refactored or reviewed in an Astro site, when styling an .astro component or page, adding a global stylesheet, scoping styles, building layouts, fluid type and spacing, colors, dark mode, motion, or when asked to "style this component", "make this responsive", "clean up the CSS" or "set up base styles" in Astro. Adapts the moderncss/skills rules to Astro's scoped styles, cascade order and build output.
license: MIT
compatibility: Verified on Astro 7.4. Requires the modern-web-guidance skill (`pnpm dlx skills add GoogleChrome/modern-web-guidance`; `npx` is equivalent where the project allows it).
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
| `1rem` is 16px | Not assumed. Read the project's root font size (`100%` or `62.5%`) and convert. |
| Linter enforces most rules | `stylelint-config-modern` covers logical properties, display notation, `oklch` and nesting notation. This fork adds media ranges (`media-feature-range-notation: 'context'`) and `.astro` coverage (`postcss-html`); `@layer` discipline, nesting cohesion and fluid `clamp()` remain review rules. |

## Rules

### Architecture

#### Scoping (`<style>` in `.astro`)

- **Rule**: Put component styles in the component's own `<style>`. Do not add `@scope` inside it.
- **Constraint**: Avoid component CSS in global files, `<style is:global>` for component styles, and `@scope` in `.astro` files.
- **Rationale**: Astro rewrites selectors with a `data-astro-cid-*` attribute, so scope is already enforced. `@scope` only duplicates it.
- **Not reached by scope**: Elements rendered by a child component and DOM created at runtime by script (slider pagination, cloned items) do not get the parent's `data-astro-cid-*`. Style them with `:global()`, or from the component that renders them.
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

- **Check**: After a build, confirm the order in `dist/` before relying on it. Astro splits CSS into chunks, and layer order follows the first appearance in the cascade. A bare order statement with no rules of its own is dropped from the built CSS (see [references/astro-styling.md](references/astro-styling.md)).

#### Scope strategy (`scopedStyleStrategy`)

- **Rule**: Set `scopedStyleStrategy: 'where'` in `astro.config.mjs` when the project uses layers.
- **Rationale**: Per the Astro docs, `'where'` adds no specificity, so layers and selectors decide conflicts instead of the scope attribute.
- **Constraint**: Keep the default `'attribute'` if existing global CSS relies on scoped styles winning on specificity. Change it in its own commit and compare pages.

#### Nesting rules and at-rules (`&`)

- **Rule**: Nest rules and at-rules inside their parent rule. Write `&` for attached selectors (`&:hover`, `&.is-open`, `&::before`) and omit it for combinators (`b`, not `& b`).
- **Constraint**: Avoid unnested selectors such as `a {} a:hover {}`. Never write `&--modifier` or `&__element`: that is Sass, it is invalid in native CSS, and the browser drops the whole rule without an error. Write `&.block--modifier`.

#### Ranged queries (`20em < width <= 40em`)

- **Rule**: Use range syntax for size queries, with ranges that do not overlap.
- **Constraint**: Avoid `min-width`/`max-width` and overlapping ranges.

### Linting (`stylelint`)

- **Rule**: Lint `src/**/*.css` **and** `src/**/*.astro` through the [`stylelint-config-modern-astro`](https://github.com/gustavomunix/stylelint-config-modern-astro) shareable config. It ships the `.astro` override (`customSyntax: 'postcss-html'`, so `<style>` blocks are parsed — static `style="…"` attributes are linted too, dynamic `style={…}` are not), range enforcement (`media-feature-range-notation: 'context'`, covering every `min-`/`max-` prefix without listing names), `declaration-block-no-duplicate-properties` (allowing `100vh; 100dvh` fallbacks) and `property-no-unknown` for typos. Project glue stays in the consuming config: `ignoreFiles` for legacy vendor exports, and a control fixture with known wire violations that must keep failing the run, so a parser upgrade that silently skips `<style>` blocks fails instead of staying green.
- **Constraint**: Acceptance criterion for a new rule: every consuming project lints green; a rule that flags intended patterns only lands after those are fixed.
- **Rationale**: Verified on stylelint 18 + stylelint-config-modern 2.0.1 + postcss-html 2.0.1: `overrides[].customSyntax` resolves from the package's own dependencies under pnpm's strict `node_modules`; media feature rules never reach custom property names (`--max-width` is unaffected).

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
- **Root size**: Do not assume it. Read the project's `html { font-size }` and convert with it. At `100%` (the browser default), `1rem` is 16px and the examples apply as written. At `62.5%`, `1rem` is 10px and `1.6rem` is 16px. Both are valid; keep whichever the project already uses and write the same convention everywhere.
- **Percent, not px**: Keep the root in `%` or `rem`-friendly units, never `px`, so user font-size settings and zoom still apply.

#### Widow and orphan words (`text-wrap`)

- **Rule**: `text-wrap: balance` on headings, `text-wrap: pretty` on other text.
- **Constraint**: During a migration this rule is suspended: keep whatever the legacy text already had and do not add `text-wrap` to text that lacked it, since it changes line breaks. Apply the rule afterwards, in its own commit gated by a visual diff.

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
- **Constraint**: Avoid `prefers-reduced-motion: reduce` overrides. When migrating legacy CSS, do not mechanically convert an existing `reduce` block into a `no-preference` wrapper: the two are not equivalent (one disables, the other gates), so check the result with reduced motion on and off before accepting the diff.
- **JS animation**: Prefer CSS and the Web Animations API (`element.animate()`) over an animation library. Check the same media query in script with `matchMedia`.

## Applying these rules

When asked to author, refactor or review CSS in an Astro project:

1. Read `astro.config.mjs` for `scopedStyleStrategy`, `build.inlineStylesheets` and integrations, and find the root `font-size` before writing any `rem`.
2. Decide the home of each rule: component `<style>` for component styles, one global layered stylesheet for tokens, reset and base.
3. Apply the rules above to every touched rule, not only the lines you edit.
4. Retrieve the Modern Web Guidance guides for the task and apply them within these rules.
5. Run the project's CSS lint (e.g. `pnpm lint:css`; it covers `.astro` `<style>` blocks via `stylelint-config-modern-astro`) on the changed files and fix what it reports.
6. Build, then inspect the built CSS for layer order and the scope attribute. See [references/audit.md](references/audit.md).
7. When the CSS is legacy (a WordPress, Bricks or Webflow export), follow [references/migration.md](references/migration.md): layer the legacy CSS first, migrate one component or page at a time, and gate every step with a pixel diff of all routes. A passing linter does not show that a page looks the same.

## References

- [references/astro-styling.md](references/astro-styling.md): scoping, `:global()`, passing classes to children, cascade order, import leakage, CSS bundling.
- [references/astro-delivery.md](references/astro-delivery.md): images, fonts, scripts, prefetch and structure choices that affect CSS and Core Web Vitals.
- [references/migration.md](references/migration.md): porting legacy CSS to components: layers for legacy CSS, dead rules, pitfalls, the visual gate, working with several agents.
- [references/audit.md](references/audit.md): how to verify the result, and which companion skills to use.
- [references/sources.md](references/sources.md): where each rule came from.
