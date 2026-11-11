# modern-css-astro

Agent skill with opinionated rules for modern CSS in Astro projects. Derived from [moderncss/skills](https://github.com/moderncss/skills) and adapted to Astro's scoped styles, cascade order and build output.

## Install

Copy the folder into a project's `.agents/skills/` directory, or into `~/.agents/skills/` for personal use. It requires the `modern-web-guidance` skill:

```bash
pnpm dlx skills add GoogleChrome/modern-web-guidance
```

## Contents

- `SKILL.md`: the rules and how to apply them.
- `references/astro-styling.md`: scoping, cascade order and bundling in Astro.
- `references/astro-delivery.md`: images, fonts, scripts and prefetch.
- `references/audit.md`: verification steps and companion skills.
- `references/sources.md`: where each rule came from, and what was not adopted.

## License

MIT. See `LICENSE`.
