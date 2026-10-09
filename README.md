# intelligent-agent-skills

Shared agent skills for the Multica workspace. Agents and runtimes load skills from
this repo instead of from scattered local copies.

A skill is a directory containing a `SKILL.md` with YAML frontmatter (`name`,
`description`) and Markdown instructions. Long reference material lives in a
`references/` subfolder beside the `SKILL.md`.

## Tiers

### `general/` — reusable by anyone

| Skill | What it does |
|---|---|
| [`blogging-anti-patterns`](general/blogging-anti-patterns/SKILL.md) | Editing checklist for software blog posts — structural mistakes that lose readers. Distilled from Michael Lynch's "Anti-Patterns in Software Blogging." |
| [`stop-slop`](general/stop-slop/SKILL.md) | Remove predictable AI writing patterns from prose. Third-party (Hardik Pandya, MIT) — see its inline license. |

### `intelligenttools/` — specific to our setup

| Skill | What it does |
|---|---|
| [`supabase-intelligenttools-co`](intelligenttools/supabase-intelligenttools-co/SKILL.md) | How to reach the Supabase database behind intelligenttools.co: which key, the RLS zero-row trap, which scripts read vs. write. Sanitized — carries no credentials. |

## Upstream skills (not vendored here)

These are maintained elsewhere. Install them from their source rather than copying
them into this repo:

- **PixiJS skills** — 2D/WebGL/WebGPU rendering guidance for agents. MIT (© PixiJS).
  Install: `npx skills add https://github.com/pixijs/pixijs-skills`
  Source: https://github.com/pixijs/pixijs-skills
- **browser-automation** — headless page-load verification (console errors, failed
  requests, screenshots). Third-party pack; install from its upstream source.
- **game-development** — run-and-look-at-it loop for games across engines.
  Third-party pack; install from its upstream source.
- **Anthropic example skills** — `import-memory`, `docs`, `docx`, `pdf`, `pptx`,
  `xlsx`, `skill-creator`, `google-workspace`. Vendor-shipped by Anthropic; loaded
  through the assistant's own skill-sync, not redistributed here.

## Licensing

Each skill keeps its own license and attribution. `general/stop-slop` is MIT by Hardik
Pandya and carries its MIT text and attribution inline; do not strip them. Everything
authored in this workspace is covered by this repo's `LICENSE`.

## Status

Private during rollout. Flips to public after a final secrets/licensing pass
(tracked on INTE-108).
