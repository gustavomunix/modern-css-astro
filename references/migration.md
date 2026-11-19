# Migrating legacy CSS to Astro components

Procedure and pitfalls from porting a Bricks/WordPress site to Astro. It applies to any site that arrives with large global stylesheets, unlayered rules and per-page CSS.

## Order of work

1. Build a visual baseline before touching CSS (see Visual gate).
2. Delete dead vendor CSS, fonts and media. Prove each deletion against the generated HTML and `src/`.
3. Declare the layer order and put the legacy CSS inside layers. This changes no pixel when done right.
4. Move CSS that belongs to shared components (header, footer, mega menu, badge) into their `<style>`, and remove the copies from page stylesheets.
5. Migrate page by page: each rule goes to the component that renders it, or to the page's `<style>` with `:global()` when two or more sections share it.
6. Only then run `--fix` and modernize values.

Steps 2 to 5 come before step 6 so a visual diff always has one cause.

## Layers for legacy CSS

- Unlayered rules beat every layer. Component styles written in `@layer components` lose to unlayered legacy CSS until the legacy CSS is layered too.
- A working order puts the legacy layers first and the new ones after, declared in the layer-order stylesheet imported first by the base layout. Example (Bricks export): `@layer bricks, vendor, tokens, base, components, utilities;`. Confirm the statement survives the build: a bare order statement with no rules of its own is dropped from the built CSS (see [astro-styling.md](astro-styling.md)).
- Import each third-party stylesheet into the legacy layer it belongs to, not into a shared one. Example (Bricks export): framework base, icon fonts and sliders go in `layer(bricks)`, not `layer(vendor)`. Importing a stylesheet into a different layer inverts its order against the copies of that layer inside page stylesheets and changes `display`, `min-width` and similar values.
- Layer order beats specificity. A `(0,3,0)` selector in a lower layer loses to a `(0,2,0)` selector in a higher one. When a legacy override relied on specificity, move it to a higher layer (`utilities`) instead of raising specificity.
- Wrap page stylesheets mechanically with a PostCSS script: keep top-level `@layer`, `@font-face`, `@keyframes`, `@property` and `@import` outside, put every other rule in one `@layer components { }`. Keep rule order.
- When layering turns on component styles that used to lose to legacy CSS, the page changes because the component is now correct. Look at the result before accepting it as the new baseline.

## Dead rules

- A rule is dead when no element in the generated HTML matches it at any width. Check the HTML, not only the markup in `src/`.
- Count states and classes that script adds: `.open`, `.is-active`, `.is-scrolled`, `.is-initialized`, `.splide__*`, `:hover`, `:focus`.
- Selectors keyed on builder attributes the markup never emits never match. Example (Bricks export): `[data-id="brxe-..."]` when the markup has no `data-id`. Remove them, and report which backgrounds never rendered. Do not recreate them without the owner's decision.
- Do not fix legacy typos during a migration (`overflow: hidd`). Fixing one changes the layout. List them.

## Writing the migrated CSS

- `&--suffix` and `&__element` are Sass. In native CSS the rule is invalid and the browser drops it. Write `&.block--modifier` for attached selectors and omit `&` for descendants.
- Astro's scope does not reach elements rendered by a child component, or DOM that script creates at runtime (Splide pagination, cloned marquee items). Use `:global()` for those parts: `> :global(.splide__pagination)`.
- A rule shared by two or more sections of one page goes in the page's `<style>` with `:global()`. A rule used by one section goes in that section's `<style>`.
- A repeated hero or title block across pages is a candidate for a shared component. Migrate each copy first, consolidate afterwards.
- `radial-gradient(circle at block-start inline-end)` is not supported. Keep `at top right`. Check every logical conversion in the browser, not only in the linter.
- Keep whatever `text-wrap` the legacy text already had. Adding `balance` or `pretty` to text that did not have it changes line breaks, so apply the `text-wrap` rule afterwards, in its own commit gated by a visual diff.
- Convert a color to `oklch()` only when the round trip returns the same hex. Prefer an existing token when the value is equal. Otherwise keep the value and report it.
- Convert `px` to `rem` only when exact: divide by the project's root factor (10 at `62.5%`; 16 at `100%`).
- `display: block` becomes `display: block flow`.
- Keep ids and classes that scripts or other CSS use.
- Do not mechanically convert a legacy `prefers-reduced-motion: reduce` block into a `no-preference` wrapper: the two are not equivalent, so check the result with reduced motion on and off. See the motion rule in [SKILL.md](../SKILL.md).

## Linting `.astro`

`stylelint` does not parse `.astro`. Extract each `<style>` block to a temporary `.css` in `/tmp`, run `stylelint --fix` on the copy, paste the result back, and run it again without `--fix`. Do not run `--fix` on repo files during a migration.

## Visual gate

A pixel diff is the safety net for CSS work. Keep the tooling deterministic and independent of the site's dependencies. It may live in the repo (e.g. `tools/`) or outside it.

- Capture every route at 375, 768 and 1440 px, full page.
- Make the capture deterministic: `reducedMotion: 'reduce'`, disable animations and transitions, abort requests to other origins, scroll the whole page so lazy images load, wait for fonts, then for `img.decode()`. Hide autoplay regions (carousels, decks).
- Capture the same build twice and diff the captures. Do not trust the gate until that diff is 0.000%. Without the steps above, two captures of the same build differed by 5 to 10% on image-heavy routes.
- Threshold: 0.02% of pixels per image.
- A diff in a region far from the edited rule usually comes from capture noise or CPU load when several browsers run at once, not from the CSS.
- The diff does not cover open menus, tabs or accordions. Capture those states with scripted interactions, before and after.
- Update the baseline only for changes you looked at.

## Working with several agents

- Give each agent a disjoint set of files. Shared files (the global stylesheets, the layer-order stylesheet, layout, header, footer) belong to one agent at a time.
- Each agent builds with `astro build --outDir /tmp/<name>`, never `pnpm build`. Parallel builds into `dist/` collide.
- Take a snapshot of `src/` and `public/` before each wave. Agents are told not to use `git checkout`, so the snapshot is the undo.
- Do not trust an agent's report. Re-run the diff and read the changed files. Reports have claimed a pixel-identical badge that had disappeared, and blamed noise for a real regression.
- Run sequentially what shares files (layers, then shared components), and in parallel what does not (one page per agent).
- Commit by reconstructing each stage from the snapshots, so each commit builds. Staging by file alone cannot split files that several stages touched.
