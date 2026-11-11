# Sources

## Derived from

- moderncss/skills, `modern-css` (MIT). https://github.com/moderncss/skills. The rule set (layers, nesting, range queries, flow-relative layout, two-value display, fluid spacing and type, `text-wrap`, `oklch`, `light-dark()`, relative colors, reduced motion) comes from here. Rules changed for Astro: `@scope` dropped, `@layer` extended to component styles, root font size made explicit.

## Consulted

- Astro documentation, https://github.com/withastro/docs/tree/main/src/content/docs/en: styling, configuration reference (`scopedStyleStrategy`, `build.inlineStylesheets`), images, fonts, client-side scripts, prefetch.
- GoogleChrome/modern-web-guidance: the guide set this skill defers to.
- vercel-labs/agent-skills, `web-design-guidelines`: a review wrapper that fetches Vercel's guidelines at run time. Referenced, not copied.
- cloudflare/skills, `web-perf`: audit workflow with Chrome DevTools MCP. Referenced, not copied.
- addyosmani/web-quality-skills, `best-practices`: security, compatibility and code quality checklist. Referenced, not copied.
- Mindrally/skills, `astro`: reviewed and not adopted. It recommends `Astro.glob()` and Tailwind utility classes, and cites FID, a metric that INP replaced. Its project structure and "minimize client JavaScript" points are already covered by the Astro guides.
- Mindrally/skills, `modern-web-development`: reviewed and not adopted. It targets Next.js, React Server Components, Supabase and Tailwind.

## Verified locally

A throwaway Astro 7.4.8 project built a page with `@layer` and `@scope` inside `<style>`. Results are in references/astro-styling.md under "Observed".
