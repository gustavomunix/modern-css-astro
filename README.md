# modern-css-astro

Agent skill with opinionated rules for modern CSS in Astro projects. Derived from [moderncss/skills](https://github.com/moderncss/skills) and adapted to Astro's scoped styles, cascade order and build output.

## Install

Use the package manager of your project:

```bash
npx skills add gustavomunix/modern-css-astro
pnpm dlx skills add gustavomunix/modern-css-astro
bunx skills add gustavomunix/modern-css-astro
```

The skill depends on `modern-web-guidance`:

```bash
npx skills add https://github.com/googlechrome/modern-web-guidance --skill modern-web-guidance
```

The CLI installs into `.agents/skills/` and records a hash in `skills-lock.json`. Run `skills update` to pull new versions.

To copy by hand, put this folder in a project's `.agents/skills/` or in `~/.agents/skills/`.

## Contents

- `SKILL.md`: the rules and how to apply them.
- `references/astro-styling.md`: scoping, cascade order and bundling in Astro.
- `references/astro-delivery.md`: images, fonts, scripts and prefetch.
- `references/audit.md`: verification steps and companion skills.
- `references/sources.md`: where each rule came from, and what was not adopted.

## License

MIT. See `LICENSE`.
