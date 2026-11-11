# Astro delivery reference

Choices that change how CSS behaves in Core Web Vitals. Sources: the Astro guides for images, fonts, client-side scripts and prefetch, https://docs.astro.build/

## Images

- Store optimizable images in `src/`, not `public/`. `public/` files are served as-is.
- Use `<Image />` for one optimized file and `<Picture />` for several formats. Both set `width` and `height`, which prevents layout shift.
- Add `loading="lazy"` below the fold. Do not lazy-load the LCP image.
- `layout` on `<Image />`/`<Picture />`, or `image.layout` globally, generates `srcset` and `sizes` and injects styles. Check those styles against your layered CSS.
- Use plain `<img>` only for images that must not be processed.

## Fonts

- Configure fonts in `astro.config.mjs` through `fontProviders` and render `<Font cssVariable="--font-x" />` in the head of the base layout.
- Reference the font through the CSS variable in your own `font-family` declarations.
- Preload sparingly, with the `preload` prop, and only for fonts used above the fold.
- Prefer variable fonts and Astro's generated fallback metrics to reduce layout shift.

## Scripts

- A plain `<script>` in an `.astro` file is bundled, deduplicated per page and loaded as `type="module"`.
- `is:inline` opts out of processing. Use it for third-party snippets or scripts that must run before paint.
- Use custom elements for behavior tied to markup. Prefer Astro components with no client framework when no state is needed.
- Check for dead scripts before keeping them. A library loaded on every page with no matching attribute in the markup is dead weight.

## Prefetch

- Enable with `prefetch: true` in `astro.config.mjs`, then opt links in with `data-astro-prefetch`, or set `prefetch.prefetchAll`.
- Strategies: `hover` (default), `tap`, `viewport`, `load`. Data saver mode and slow connections disable prefetch.

## Structure

- One layout owns the document: `<html lang>`, head metadata, the layered global stylesheet and fonts.
- Components own their markup and `<style>`. Pages compose components.
- Content that repeats across pages belongs in typed data or content collections, not copied markup.
