# Astro styling reference

Facts from the Astro styling guide, plus what was observed in a build on Astro 7.4. Source: https://docs.astro.build/en/guides/styling/

## Scoping

- `<style>` in an `.astro` file is scoped by default. Selectors get a `data-astro-cid-*` attribute (or `:where()`, or a class, depending on `scopedStyleStrategy`).
- Low-specificity selectors such as `h1 {}` or `p {}` are safe inside a scoped `<style>`.
- Observed: `@layer` inside `<style>` is preserved and the scoped selector stays inside it.
- Observed: nested child selectors are not scoped individually. `.card { h2 { } }` builds to `.card[data-astro-cid-x] { & h2 { } }`.
- Observed: `@scope (.card)` also works and scopes `.card`, but adds nothing over the scope Astro already applies.

## Escape hatches

- `:global(h1)` inside a scoped rule styles descendants the component does not own: `article :global(h1) { }`.
- `<style is:global>` turns scoping off for the whole block. Use it only in a stylesheet that is meant to be global.
- Prefer one layered global stylesheet imported by the base layout over many `is:global` blocks.

## Classes and children

- Passing `class` to a child component only works if the child forwards the scope attribute. Spread `...rest` props onto the child's root, or style the child from the child.
- Because the attribute includes the child in the parent's scope, styles can cascade from parent to child. Keep parent selectors specific to what the parent owns.

## Dynamic values

- Prefer `class:list` for state (`<div class:list={['box', { red: isRed }]}>`).
- For values that come from frontmatter, `define:vars` on `<style>` passes them as CSS variables. Read the directive reference before using it with layers or large style blocks.

## Cascade order

Astro orders CSS from lowest to highest precedence: `<link>` stylesheets, imported stylesheets (in import order), then scoped styles. Specificity still decides first, then order.

- Importing a component applies the CSS it imports, even when the component is never rendered. Imported CSS can leak between pages. Keep imported CSS to layout, token and reset files, and put component CSS in `<style>`.
- Layer order is the intended tool for managing precedence. Do not rely on import order to win conflicts.

## Bundling

- Astro minifies and splits CSS per page and shared chunks.
- Chunks under 4 kB are inlined as `<style>`; larger ones are linked. `vite.build.assetsInlineLimit` changes the limit and also affects scripts and images.
- `build.inlineStylesheets` accepts `'auto'`, `'never'`, `'always'`. `'never'` costs one extra request per page and keeps the CSS cacheable across pages.
- Avoid `?raw` and `?url` CSS imports. They skip Astro's bundling.

## Preprocessors and frameworks

- Plain CSS is the default here. Add Sass, Tailwind or CSS-in-JS only when the project already uses them; they do not enforce these rules.
- Scoped styles inside React, Vue or Svelte components follow those frameworks' rules, not Astro's.
