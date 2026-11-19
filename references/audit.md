# Audit reference

## Verify CSS changes

1. The project's CSS lint on changed files. Base packages enforce logical properties, `display` notation, `oklch`, nesting notation, media ranges, duplicate properties and unknown properties; `@layer` discipline, nesting cohesion and fluid `clamp()` stay review rules. Unknown values are not covered (`overflow: hidd` passes): add `declaration-property-value-no-unknown` as an extra rule to catch typos. `.astro <style>` needs [`stylelint-config-modern-astro`](https://github.com/gustavomunix/stylelint-config-modern-astro).
2. `pnpm build`, then read the built CSS in `dist/`. Confirm layer order, that scope attributes are present, and that no unlayered component rule slipped in.
3. Compare each affected page against a baseline screenshot before and after.
4. Search for banned patterns in the files you touched: `px` spacing, `@media (max-width`, `min-width`, hex colors, `prefers-reduced-motion: reduce`.

## Companion skills

Install the ones that apply; this skill does not depend on them except `modern-web-guidance`.

| Skill | Use it for |
|---|---|
| `modern-web-guidance` (GoogleChrome) | Required. Feature guides and fallbacks. |
| `web-design-guidelines` (vercel-labs/agent-skills) | UI review against Vercel's Web Interface Guidelines. It fetches the rules from `https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md` at run time. |
| `web-perf` (cloudflare/skills) | Core Web Vitals and Lighthouse audits with Chrome DevTools MCP. |
| `best-practices` (addyosmani/web-quality-skills) | Security headers, deprecated APIs, doctype, charset, console errors. |
| `astro` (mindrally/skills) | Not recommended as-is. See sources.md. |

## Review checklist

- Component styles live in the component's `<style>` and sit inside a layer.
- No `@scope` in `.astro` files.
- Spacing and type use container units with a `rem` term, converted for the project's root size.
- No physical properties, no single-value `display`, no non-`oklch` colors.
- Motion is opt-in under `prefers-reduced-motion: no-preference`.
- The LCP image is not lazy-loaded and has dimensions.
- No render-blocking script or library that the markup does not use.
