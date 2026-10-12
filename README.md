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
| [`article-draft`](general/article-draft/SKILL.md) | Draft a long-form article in the user's voice from a topic and an angle, refusing to proceed without an argument. |
| [`headline-work`](general/headline-work/SKILL.md) | Generate and score titles against explicit criteria and recommend one, instead of listing ten. |
| [`list-email`](general/list-email/SKILL.md) | Write an email to a subscriber list that reads like a person wrote it, with one job per email. |
| [`repurpose`](general/repurpose/SKILL.md) | Cut one finished piece into the other formats it can carry, keeping only the parts that survive the move. |
| [`social-batch`](general/social-batch/SKILL.md) | Turn one idea into a week of short posts across X, Threads or LinkedIn, mixing formats deliberately. |
| [`video-script`](general/video-script/SKILL.md) | Write a video script structured for retention, in spoken language rather than written language. |
| [`voice-profile`](general/voice-profile/SKILL.md) | Derive a reusable writing voice profile from work the user has already published. |
| [`verify-gate`](general/verify-gate/SKILL.md) | Run the repository's definition of done before handing work back. |
| [`claim-check`](general/claim-check/SKILL.md) | Verify every factual claim in a draft against its source before publishing. |
| [`safe-db-change`](general/safe-db-change/SKILL.md) | Take a production database change through generate, validate, dry run, ask, execute, verify. |
| [`page-check`](general/page-check/SKILL.md) | Load a URL in a real headless browser and report what actually happened, instead of trusting a status code. |
| [`send-safely`](general/send-safely/SKILL.md) | Pre-flight a recipient list and dry-run an email send before anything leaves the machine. |

### `intelligenttools/` — specific to our setup

| Skill | What it does |
|---|---|
| [`supabase-intelligenttools-co`](intelligenttools/supabase-intelligenttools-co/SKILL.md) | How to reach the Supabase database behind intelligenttools.co: which key, the RLS zero-row trap, which scripts read vs. write. Sanitized — carries no credentials. |
| [`blog-post-creator`](intelligenttools/blog-post-creator/SKILL.md) | Create SEO-optimized blog posts for intelligenttools.co following the project's content standards. Sanitized — project ref replaced with a placeholder. |
| [`frontend-generalist`](intelligenttools/frontend-generalist/SKILL.md) | Frontend development for intelligenttools.co (Next.js 14, React, TypeScript, Tailwind). |
| [`newsletter-issue`](intelligenttools/newsletter-issue/SKILL.md) | Build and send the intelligenttools.co newsletter. |
| [`seo-blog-validator`](intelligenttools/seo-blog-validator/SKILL.md) | Validate and optimize blog posts against intelligenttools.co SEO standards. |
| [`text-proofreader`](intelligenttools/text-proofreader/SKILL.md) | Proofread prose that ships to readers — blog markdown and tool descriptions. |
| [`tool-adder`](intelligenttools/tool-adder/SKILL.md) | Safely add new AI tools to the directory DB with verification and SEO validation. |
| [`pricing-read`](intelligenttools/pricing-read/SKILL.md) | Read a vendor's real pricing off their own page and record it under one convention. |
| [`repo-onboard`](intelligenttools/repo-onboard/SKILL.md) | Write a CLAUDE.md for an existing repo from what the code actually does. |

Reference material (not triggered skills) lives alongside the tier:

- [`intelligenttools/CLAUDE.md`](intelligenttools/CLAUDE.md) — the directory's project bible (rules, SQL templates, pricing ladder). Carries only env-var *names*, no values.
- [`intelligenttools/_commands/`](intelligenttools/_commands) — first-party slash commands (`seo-blog-post`, `process-daily-submissions`, `add-new-tool`). `seo-blog-post` sanitized (project ref → placeholder).
- [`intelligenttools/_agents/`](intelligenttools/_agents) — the 6 portable workspace agent instructions (Scout, Vera, Pricing Sweep, Tools Submissions, Intelligent Tools Dev, SEO Blog Writer). Agent secrets live in each agent's `custom_env`, never in these texts.

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

Public. Populated with the Stage 1 clear-to-move assets; the project ref and all
secret values were stripped before anything landed here (tracked on INTE-108).
