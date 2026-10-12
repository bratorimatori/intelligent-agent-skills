# CLAUDE.md - intelligenttools.co Master Guide

Complete reference guide for all development, database, content, and operational tasks.

## Project Overview

**intelligenttools.co** - AI tools blog and directory featuring honest developer reviews of AI coding tools. Built by a senior full-stack developer with 10+ years experience.

**Mission**: Provide authentic, non-promotional reviews of AI tools from a real developer's perspective.

## Tech Stack

Versions are pinned in `package.json` - that file wins over this list.

- **Framework**: Next.js 15.5.9 (App Router, Turbopack for `dev` and `build`)
- **Frontend**: React 19.1.2, TypeScript 5
- **Styling**: Tailwind CSS 4 (`@import "tailwindcss"` in `app/globals.css`;
  `tailwind.config.ts` still carries the content globs and the typography plugin)
- **Database**: Supabase (PostgreSQL)
- **Deployment**: Vercel
- **Analytics**: Google Analytics 4 (GA4)

React 19 and Next 15, not 18 and 14: `use` and the new form/action APIs are
available, and `useFormState` is `useActionState`.

---

# 🚨 CRITICAL SAFETY RULES (NON-NEGOTIABLE)

## Database Operations - ABSOLUTE REQUIREMENTS

### Rule 1: NEVER Touch the Live Database Without Asking First
- ❌ Do NOT run a migration, or anything else that writes, on your own initiative
- ❌ Do NOT treat "add these tools" as approval to apply them - that approves the
      content, not the write
- ❌ Do NOT bypass this rule "just this once"
- ✅ DO ask, on the issue, in the turn before you write anything
- ✅ DO apply it yourself once the answer is yes

The gate is the ask, not the actor. An agent may run the migration; it may never
decide on its own that now is the moment. Approval is per batch and it expires
with the batch - a yes for this batch is not a standing yes for the next one.

Ask like this, and then stop and wait for the reply:

> Batch 23 is validated and ready: 4 rows, dry run parses clean, guard skips
> nothing. Say go and I will apply it and enrich.

"Go", "apply it", "run it" are approval. Silence is not. A thumbs-up on a
different question is not. If you are unsure whether something counted as
approval, it did not - ask again.

### Rule 2: Database Change Process (ALWAYS Follow)
1. **Generate** - Write SQL to a file (e.g., `migrations/draft-YYYY-MM-DD-description.sql`)
2. **Validate** - `node scripts/validate-tool-sql.js <file>` and, for tool batches,
   `node scripts/check-subsection-routing.js <batch.json>`. Both must pass.
3. **Dry run** - `node scripts/apply-migration.js <file>`. Reads only. Shows exactly
   which rows would be written and which the guard would skip.
4. **Show and wait** - Post the dry-run output and get explicit written approval
5. **Execute** - `node scripts/apply-migration.js <file> --execute`
6. **Verify** - Read the rows back with the **anon** key, not the service key, so you
   are looking at what a visitor sees rather than what you hoped you wrote

See "Applying a migration" below for the full sequence including enrichment.

### Rule 3: Always Ask Before
- ❌ CREATE, ALTER, DROP statements
- ❌ DELETE or UPDATE without WHERE clause
- ❌ Any migration script
- ❌ Any database schema changes
- ✅ SELECT queries are safe

### Rule 4: Safe Query Practices
- Always use WHERE clauses with DELETE/UPDATE
- Always LIMIT results when testing
- Use transactions for multi-statement operations

### Real-World Impact
**Every database operation affects:**
- Real users on production
- Affiliate link tracking
- Analytics data
- Tool visibility on the website
- Revenue tracking

## Rule 5: AI Safety Enforcement (Mitigation Strategy)

To ensure these rules are followed consistently, Claude follows this **mandatory pre-action checklist** for ALL database operations:

### Pre-Action Checklist (Required Before ANY Database Operation)

```
⚠️  DATABASE OPERATION CHECKPOINT

Rule Check: [Reference which CLAUDE.md rule applies]
Operation: [What is being done]
Impact: [What data is affected]
Reversible: [YES/NO - can this be undone?]
Approval Status: [PENDING - waiting for user]
```

**Example:**
```
⚠️  DATABASE OPERATION CHECKPOINT

Rule Check: Rule 2 - Database Change Process
Operation: UPDATE tools table - set logo_url for 10 tools
Impact: Changes logo_url field for continue-dev, groqcloud, langgraph, lm-studio, modal, ollama, openrouter, pinecone, unstructured, vercel-ai-sdk
Reversible: YES - can revert with previous URLs
Approval Status: PENDING - SQL generated for review

Next Step: Copy migrations/logo-update-2025-12-18.sql to Supabase SQL Editor
```

### Implementation Rules

1. **Claude ALWAYS shows the checklist** before attempting any database operation
2. **Claude NEVER executes on its own initiative** - the SQL file and a dry run come
   first, and the turn ends there with the ask
3. **Claude ONLY executes** after explicit user approval (phrases like "execute", "run", "apply")
4. **Claude documents** what approval was given and when
5. **Claude maintains** a clear separation: **GENERATE** ≠ **EXECUTE**, and never
   collapses them into one turn

### Why This Works

- **Checklist catches mistakes** before they happen (prevents my pattern-matching from skipping rules)
- **Generate-then-ask** removes the ability to shortcut the safety process; the write
  is a separate, named decision rather than the tail end of a long task
- **A dry run makes the ask concrete** - you approve a printed list of rows, not a promise
- **Clear approval phrases** make your intent explicit and trackable
- **No ambiguity** - "approved" might mean "I reviewed this" but "execute this SQL" is unambiguous

---

# Definition of Done - the verify gate

A change is done when `npm run verify` exits 0. Not before. This applies to
every change, whether a human or an agent wrote it: no pull request, no "ready
for review", no hand-back until the gate is green.

```bash
npm run verify        # lint + typecheck + packs + build   ~40s   <- required to hand back
npm run verify:fast   # lint + typecheck + packs           ~10s   <- for iterating, not enough on its own
```

The gate lives in `scripts/verify.sh`. It is a script, not a policy - it exits
non-zero when the repository is broken, and that exit code is the whole rule.

## What it checks

| Stage | Command | Catches |
|---|---|---|
| lint | `eslint .` | errors, plus any *increase* in warnings over `scripts/verify-baseline.json` |
| typecheck | `tsc -p tsconfig.verify.json --noEmit` | type errors, faster and with better messages than waiting for the build |
| packs | `build-lead-magnets.js --check` | `public/packs/*.zip` gone stale against `lead-magnets/`, and house-rule breaches in files subscribers download |
| build | `next build --turbopack` | anything that only breaks when the app is actually compiled and prerendered |

`tsconfig.verify.json` is `tsconfig.json` minus `.next/types`. Those are build
output, and a stale `.next` makes `tsc` report missing modules for routes you
just renamed - a failure with nothing wrong behind it. `next build` checks them
itself, so the build stage still covers them.

There is no test stage, because there is no test suite in this repository yet.
Say so plainly when you hand work back: the gate proves the code builds, not
that it behaves. Until a test runner lands, behaviour is verified by hand and
the hand-back has to describe what was checked and how.

The whole gate runs in about 40 seconds, so there is no separate "fast gate for
hand-back, full gate for merge" split. `--fast` exists for the edit-check loop
while you work; it is not a hand-back.

## The build stage never touches the database

`scripts/verify.sh` points Supabase at `http://127.0.0.1:1` before running the
build. `generateStaticParams` comes back empty, the build compiles and
type-generates everything, and no build-time query can reach production - see
Rule 1 above. It also means the gate needs no credentials, so it runs on a fresh
clone with nothing but `npm ci`.

Two consequences to keep in mind:

- The build prerenders zero tool and category pages. A change that breaks only
  on real rows will pass the gate. That is what hand-verification is still for.
- `scripts/verify.sh --real-env` builds against `.env.local` instead. It reads
  the production database. Use it deliberately, not by default.

## What you may not do to make it pass

The gate measures the change. Changing the gate to fit the change defeats it.

- No `eslint-disable` comments. If a rule is wrong for a whole class of file,
  say so in `eslint.config.mjs` with a comment explaining why - a one-line
  suppression at a call site is what this rule exists to stop.
- No `any` casts to silence a type error, and no loosening `tsconfig.json`.
- No deleting, commenting out, or `@ts-expect-error`-ing the failing call site.
- No raising `eslintWarnings` in `scripts/verify-baseline.json` to absorb new
  warnings. That number only goes down, except with a stated reason in the PR.
- No quietly dropping part of the change so the rest gets through. If the scope
  shrinks, that is a thing you say out loud, not a thing you do silently.

## When it fails and you cannot fix it

Hand back the real output - the actual stage output, in full, not a summary and
not "the build failed". Say what you tried and what you think the cause is, and
leave the change where it is. A red gate reported honestly is a useful result.
A green claim over a red gate is the one outcome this whole thing exists to
prevent.

---

# Code review - the review pass

The verify gate proves the code builds. It does not prove the code is right. A
change is worth a second pass from something that did not write it, because the
run that authored it cannot see the assumption it got wrong - it held that
assumption the entire time.

**Vera reviews Multica work only.** She is a Multica agent, she is not a
collaborator on this GitHub repository, and there is no way to request her from
outside Multica. Work produced anywhere else - a Claude Code session, a local
edit - does not get her review and should not claim to be waiting on one.

## Hand-back

1. `npm run verify` exits 0.
2. Push the branch and open the PR.
3. Report the PR link.

Opening a PR and going quiet is not a hand-back. Neither is asking for review on
a red gate - fix the gate first, or say plainly that you could not. Bojan reviews
and merges; a Multica-produced change gets Vera first.

## Commit identity and delivery

Agent commits are authored as `Bojan Tomic <BOJAN_COMMIT_EMAIL>` - the address
on the `bratorimatori` GitHub account. Vercel refuses to deploy a commit whose
email it cannot match to an account with access to the project, and a commit
authored under any other address blocks the deployment rather than failing it
loudly, so this is the one identity detail worth being careful about.

As of 2026-09-30 the runtime host inherits it correctly. The machine-global git
config is `<BOJAN_COMMIT_EMAIL>`, and the per-worktree identity Multica seeds from
it now carries the same address, so a plain `git commit` in an agent checkout is
already correct. It was not before: the global config held a work address, every
agent worktree inherited it, and the commits that reached `main` under it are
still visible in `git log`.

Because that inheritance is host state rather than anything this repository
controls, the explicit form remains the guaranteed one and costs nothing:

```bash
git -c user.name="Bojan Tomic" -c user.email="<BOJAN_COMMIT_EMAIL>" commit -m "..."
```

If you ever see a commit of yours authored under a different address, the host
config has drifted back. Say so rather than working around it commit by commit.

> `BOJAN_COMMIT_EMAIL` is the Gmail address on the `bratorimatori` GitHub
> account (kept out of this public repo). The real value lives in the host git
> config and each agent's environment, not here.

Nothing is committed straight onto a shared branch. Every change - tool batch,
script, doc, `lib/dark-logos.ts` regeneration - goes on a branch off fresh
`origin/main` and arrives as a pull request for Bojan to review and merge.

Branch names are `<area>/<short-slug>`, lowercase and hyphenated - `blog/`,
`seo/`, `data/`, `newsletter/`, `fix/`, `docs/`. A date suffix where the work is
a dated batch (`data/click-snapshot-2026-09-30`). Agent runtimes that generate
their own branch use `agent/<agent-name>/<id>`; that is fine and needs no
rename. Commit subjects are imperative and under ~72 characters, prefixed with
the same area where it helps (`SEO: fix low-CTR titles`). The PR description
says what changed, why, and what was verified - the verify-gate output belongs
in it.

## What a review covers, in priority order

1. **CLAUDE.md non-negotiables** - live database execution, schema changes
   without a migration. Highest consequence in this codebase.
2. **Auth and secrets** - especially a new route combining the service-role
   client with no auth check, which is a public write endpoint.
3. **Correctness** - stated as a concrete failing input, not a worry.
4. **Behaviour the gate cannot see** - the build prerenders zero tool and
   category pages, so anything that only breaks on real rows passes it.
5. **Reuse and simplification** - last, and always advisory.

## Blocking versus advisory

**Blocking** - fix before merge: a CLAUDE.md non-negotiable, a new
unauthenticated write path or leaked secret, a correctness bug with a named
failure case, a new lint *error* on a changed line, or evidence the gate was
worked around (`eslint-disable`, `@ts-expect-error`, `any` casts added to
silence it, a raised `eslintWarnings` baseline without a stated reason).

**Advisory** - said once, never blocking: new lint *warnings* on changed lines,
reuse, duplication, naming, anything phrased as "consider".

**Never raised:** formatting, import order, the CommonJS pattern in `scripts/`,
and preferences with no cost attached. A review that blocks on style is a
review that gets bypassed - and then nothing above gets checked either.

## Findings are scoped to the diff

`npm run lint` on a clean `main` is **0 errors and 23 warnings** - the count in
`scripts/verify-baseline.json`. The old pile of `require()` errors in `scripts/`
is gone, so treat any lint *error* as introduced by the diff rather than
pre-existing. The 23 warnings are mostly `no-explicit-any` and unused vars;
they are not the author's to fix, and the gate only fails when a change pushes
the count above the baseline.

Findings still stay scoped to the diff: a review reporting repo-wide buries the
one real finding. Pre-existing problems get one line at the end and do not
affect the verdict.

## This is a convention, not a mechanism

Nothing currently forces the review to happen - it is a rule in this file, the
same way the verify gate is. That is deliberate for now: the volume is low and
a GitHub Actions check costs minutes and a key to enforce something nobody is
yet trying to skip. If reviews start getting skipped, that is the signal to
move this into CI as a required check rather than to write a firmer paragraph
here.

---

# Project Structure

```
intelligenttools.co/
├── app/                    # Next.js App Router - no route groups; public
│   │                       # pages sit directly here (blog/, tools/,
│   │                       # categories/, compare/, deals/, pricing/, ...)
│   ├── api/               # API routes
│   ├── admin/             # Admin surfaces (HTTP Basic, ADMIN_API_KEY)
│   ├── globals.css        # Tailwind entry + design tokens
│   └── layout.tsx         # Root layout
├── components/            # React components (mostly flat, PascalCase files)
│   ├── ui/               # Reusable UI components
│   ├── layout/           # Header, footer, shell
│   └── compare/          # Comparison-page components
├── lib/                   # Utilities and helpers (flat modules + 3 subdirs)
│   ├── supabase/         # Supabase clients (browser, server, admin)
│   ├── emails/           # React Email templates
│   └── scrapers/         # Website / pricing scrapers
├── hooks/                # React hooks
├── contexts/             # React context providers
├── content/              # Markdown blog posts and page copy
├── data/                 # Committed JSON datasets and snapshots
├── migrations/           # SQL migrations - apply via apply-migration.js
├── supabase/             # Supabase project config
├── lead-magnets/         # Source of public/packs/*.zip
├── newsletter/           # Issue drafts and send logs
├── docs/                 # Longer-form internal docs
├── public/               # Static assets (includes generated packs/*.zip)
├── scripts/              # Node.js utility scripts
│   ├── verify.sh                   # The verify gate - run before handing back
│   ├── fetch-tool-logos.js         # Download + quality-gate logos
│   ├── capture-tool-screenshots.js # Capture 1280x720 screenshots
│   ├── add-tool-enhanced.js        # Add one tool with auto-enrichment
│   ├── set-tool-media.js           # Apply logo/screenshot URLs to existing rows
│   ├── validate-tool-sql.js        # Validate a batch before it reaches the DB
│   ├── check-subsection-routing.js # Verify tags route to a real subsection
│   ├── apply-migration.js          # Apply a migration (dry run by default)
│   ├── apply-annual-pricing.js     # Add annual_usd/annual_total_usd to existing tiers
│   ├── plan-pricing-sweep.js       # Next batch of slugs to re-check, oldest first
│   ├── enrich-production-tools.js  # Fetch GitHub/Wikipedia data for a tool
│   └── [other utilities]/
├── types/                # TypeScript types (hand-maintained)
├── .claude/              # Claude Code commands and skills for this repo
└── CLAUDE.md            # This file - Master reference guide
```

There is no `styles/` directory - global CSS is `app/globals.css`.

## Generated files - never hand-edit

These are build output committed to the repository. Editing one by hand is
undone by the next regeneration, and the verify gate catches some of it:

| Path | Regenerate with | Notes |
|---|---|---|
| `lib/dark-logos.ts` | `node scripts/find-dark-logos.js --write` | Header says so. Static list - a new dark logo is invisible on the chip until this re-runs. |
| `public/packs/*.zip` | `node scripts/build-lead-magnets.js` | Built from `lead-magnets/`. The `packs` stage of the gate fails when they drift. |
| `package-lock.json` | `npm install` | Commit the result, never edit it. |
| `.next/` | `npm run build` | Not committed. `tsconfig.verify.json` exists to keep a stale copy out of typecheck. |

## Ask before touching

- `scripts/verify.sh` and `scripts/verify-baseline.json` - the gate measures the
  change; changing the gate to fit the change defeats it. The baseline number
  only goes down.
- `eslint.config.mjs`, `tsconfig.json`, `next.config.ts` - loosening any of
  these is a way of not fixing the thing they caught.
- `migrations/` - an applied migration is history. Add a new one; do not edit
  one that has already run.
- The CRITICAL SAFETY RULES section of this file, and anything it governs.
- Large refactors nobody asked for. Smallest change that does the job.

---

# Code Standards

## TypeScript
- Always use TypeScript, never `any` type
- Define interfaces for all data structures
- Use type inference where clear
- Prefer `interface` over `type` for object shapes

## React/Next.js
- Use functional components with hooks
- Prefer Server Components by default (App Router)
- Mark Client Components with 'use client' when needed
- Use Next.js Image component for all images
- Follow Next.js conventions for routing and layouts

## Supabase
- Types live in `types/database.ts` and are **hand-maintained**, not generated -
  nothing in this repo runs `supabase gen types`. Add a schema change to that
  file in the same PR as its migration, or the rest of the codebase types a
  column that exists only in the database.
- Import the right client from `lib/supabase/`: the browser client for Client
  Components, the server client for Server Components and route handlers, and
  the service-role client only behind an auth check (see ADMIN_API_KEY)
- Always handle errors from Supabase operations
- Use Row Level Security (RLS) policies for data access
- Prefer stored procedures for complex operations

## Styling
- Use Tailwind CSS utility classes
- **Avoid inline styles** - use Tailwind classes instead
- Follow mobile-first responsive design
- Keep component styles colocated
- Use semantic color names from theme
- If inline styles are absolutely necessary, document why

## Code Quality
- Write self-documenting code with clear variable names
- Add comments only for complex logic or "why" not "what"
- Keep functions small and focused (single responsibility)
- DRY but don't over-abstract

## Content Standards (Tool Descriptions & Blog)
- **NEVER use em-dashes (—)** in tool descriptions, blog content, or any database text fields
- Use hyphens (-) for separation instead: "tool - feature" not "tool — feature"
- Use parentheses for clarification: "feature (optional)" not "feature — optional"
- This prevents encoding issues, text parsing problems, and API compatibility issues
- Always validate content before adding to database

## File Naming Conventions
- Components: PascalCase (`ToolCard.tsx`)
- Utilities: camelCase (`formatDate.ts`)
- Pages: kebab-case (`blog-post.tsx`)
- Constants: UPPER_SNAKE_CASE (`API_ENDPOINTS.ts`)

---

# Adding New Tools - Complete Workflow

## Step 1: Information Verification Checklist

Before adding ANY tool, complete this verification:

### Visit Official Website
- [ ] Found official website (not blog, not alternative sites)
- [ ] Website loads correctly and is HTTPS
- [ ] Tool name on website matches what you're adding
- [ ] Website is currently active (not archived/shutdown)

### Verify Pricing
- [ ] Visited pricing/plans page
- [ ] Note exact pricing IF available (use NULL if not)
- [ ] Confirm pricing_type (free/paid/freemium/open_source)
- [ ] Noted pricing_info from THEIR page, not guessed

### Gather Descriptions
- [ ] Read official tagline/headline from homepage
- [ ] Read full description from marketing/about page
- [ ] Noted actual features from documentation
- [ ] Did NOT add features not listed on their site

### Verify URLs
- [ ] Tested website_url - loads correctly
- [ ] Using HTTPS protocol
- [ ] Using official domain, not redirects
- [ ] If open-source, found GitHub repo URL

### Check for Open-Source
- [ ] If claiming open_source, verified on GitHub
- [ ] Found actual license (MIT, Apache, etc.)
- [ ] Confirmed it's actually maintained/active

### Red Flags - Don't Add If:
- 🚩 Website redirects to a different company
- 🚩 Website is down or shows 404
- 🚩 Tool appears to be abandoned/no updates in 2+ years
- 🚩 You can't find pricing on their website
- 🚩 You're guessing about features/capabilities

## Step 2: Understanding Subsections (CRITICAL)

**Tools won't appear on category pages if this isn't done correctly!**

### What Are Subsections?
Subsections are how tools are organized within a category. Each category has multiple subsections (4-6), and tools are automatically assigned based on their **tags**.

### How Tools Get Assigned to Subsections
1. Tool is added with **tags** (you choose these)
2. System checks tags against each **subsection's keywords**
3. Tool is assigned to subsection with **most tag matches**
4. Tool appears on the category page in that subsection

### Available Categories & Subsections

**Writing:**
- Content Creation (blog, article, content, post)
- Copywriting & Marketing (copywriting, marketing, ads, sales)
- Email Writing (email, newsletter, email copy)
- Social Media Content (social media, twitter, linkedin, instagram, caption)
- Editing & Proofreading (editing, proofreading, grammar, proofread)
- Translation (translation, translate, language, multilingual)

**Image:**
- AI Art Generation (art, illustration, artwork, image generation, text-to-image)
- Photo Editing (photo, edit, editing, enhancement, enhance)
- Design Tools (design, graphic, graphics, designer)
- Logo & Branding (logo, branding, brand, brand identity)
- Background Removal (background, remove, removal, bg)
- Image Enhancement (upscale, enhancement, enhance, quality)

**Coding:**
- Code Completion (copilot, completion, autocomplete, suggestion)
- Full Stack Generation (generate, generation, full-stack, app, scaffold)
- Code Review & Debugging (debug, debugging, review, error, bug, fix)
- Documentation (documentation, doc, comment, docstring, readme)
- Testing & QA (test, testing, qa, quality, unit test, e2e)
- DevOps & Infrastructure (devops, deployment, infrastructure, docker)

**Productivity:**
- Task Management (task, management, project, todo, planning)
- Note Taking (note, notes, note-taking, notebook, knowledge)
- Meeting Assistants (meeting, transcription, summary, transcript, call)
- Email Management (email, inbox, organize, management)
- Automation (automation, workflow, automate)
- Research & Knowledge (research, knowledge, learn, information)

**Video:**
- Video Generation (video, generate, generation, text-to-video)
- Video Editing (editing, edit, editor, cut, trim)
- Subtitles & Captions (subtitle, caption, subtitles, captions)
- Animation (animation, animate, motion, motion graphics)
- Screen Recording (screen, recording, capture, screencast)
- Video Enhancement (enhancement, enhance, upscale, quality)

**Audio:**
- Voice Generation (voice, speech, text-to-speech, synthesis, tts)
- Music Generation (music, composition, compose, song, audio)
- Transcription (transcription, transcribe, speech-to-text, stt)
- Audio Editing (audio, editing, edit, sound, mixing)
- Podcasting (podcast, podcasting, podcast creation)
- Sound Effects (sound, effects, sfx)

**And more:** AI Agents, AI Search, Calendar/Scheduling, Customer Support, Design, Research, Startups, Marketing, Sales

### If Tool Doesn't Appear After Adding

The tool likely wasn't assigned to any subsection. This happens when:
1. ❌ **Tags don't match any subsection keywords** - too generic tags
2. ❌ **Wrong category** - tool in wrong category for its purpose
3. ❌ **New category but no subsections defined** - need code update

**Fix:** Make sure your tags include keywords from the target subsection's keyword list.

## Step 3: Long Description Format (REQUIRED - 2000+ chars)

The `long_description` is the most important field. It must follow a specific 5-part structure:

### Part 0: Opening Context (200-400 words) - NO HEADER
Opening paragraph (NO ## header) that provides:
- **What it is** - Clear, direct explanation
- **What problem it solves** - The pain point or need
- **Why it matters** - Context on why this is important
- **What makes it unique** - What sets it apart
- **Real-world context** - How it's being used

**Example:**
```
OpenClaw is a revolutionary personal AI assistant that runs entirely on your machine, giving you a true digital agent that actually executes tasks across your digital ecosystem. Unlike cloud-based AI assistants that only chat and suggest, OpenClaw actively automates—clearing your inbox, managing your calendar, checking you in for flights, and hundreds of other tasks autonomously. By running locally on macOS, Windows, or Linux, OpenClaw keeps your data private while offering the power of AI models like Claude, GPT, and local alternatives.
```

### Part 1: About detail (200-300 words) - STILL NO HEADER

**CRITICAL: do NOT write a `## About` header.** `components/MarkdownWithCollapsible.tsx`
renders everything before the *first* `## ` header as always-visible intro, and turns
every `## Section` after it into a closed drawer. Adding `## About` buries this content
in a drawer nobody opens. All existing tools rely on this: no `## About` header anywhere.

This is a second header-free paragraph continuing straight on from Part 0, covering:
- Key details and differentiators
- Community/maintenance status (if open-source)
- Pricing model shape, if it is unusual

**Template:**
```
[Opening context from Part 0.]

[Tool name] combines [key features/benefits]. With [key capability], it [main benefit].
The [community/license] is actively maintained with [relevant stats].
```

### Part 2: How It Works (200-300 words)
- Step-by-step workflow/process
- What users do with the tool
- The sequence of actions
- Key workflows or pathways

**Template:**
```
## How It Works

[Step 1 description...]
[Step 2 description...]
[Step 3 description...]
```

### Part 3: Core Features (400-600 words)
- 4-6 major features with **bold names**
- Detailed explanation of each feature
- Specific capabilities and benefits

**Template:**
```
## Core Features

- **Feature Name 1** - description and details...
- **Feature Name 2** - description and details...
- **Feature Name 3** - description and details...
- **Feature Name 4** - description and details...
- **Feature Name 5** - description and details...
- **Feature Name 6** - description and details...
```

### Part 4: Who This Is For (200-300 words)
- Specific audience segments
- Use cases for each audience type
- Specific pain points solved
- Real-world applications

**Template:**
```
## Who This Is For

Perfect for [audience 1] doing [use case 1], [audience 2] doing [use case 2], and [audience 3] doing [use case 3]. [Audience groups] appreciate [specific benefits]. [Organization type] will find [relevant value].
```

### ✅ DO:
- Use markdown formatting (## for section headers)
- Include 4-6 distinct features with **bold names**
- Write 2,000+ characters total
- Use clear, accessible language
- Focus on actual documented features
- Include real-world examples and use cases

### ❌ DON'T:
- Write generic marketing fluff without specifics
- Include features not actually on the tool's website
- Make it sound overly salesy
- Forget section headers
- Write less than 2,000 characters
- Make sections uneven in length
- Add fake or assumed capabilities

## Step 4: Enriched Data Format (For Open-Source & Major Companies)

When adding open-source tools or major company tools, populate `enriched_data` with GitHub and project information:

```json
{
  "github": {
    "stars": 18254,
    "forks": 1957,
    "watchers": 18254,
    "language": "Python",
    "license": "MIT License",
    "homepage": "https://example.com",
    "topics": ["ai", "agent", "automation", "open-source"],
    "createdAt": "2024-04-02T04:09:47Z",
    "pushedAt": "2025-02-01T00:00:00Z",
    "updatedAt": "2025-02-01T00:00:00Z",
    "openIssues": 73,
    "defaultBranch": "main",
    "isArchived": false,
    "description": "GitHub repository description"
  }
}
```

**GitHub fields to gather:**
- `stars` - Current star count
- `forks` - Number of forks
- `watchers` - Watchers count
- `language` - Primary language (Python, TypeScript, etc.)
- `license` - License type (MIT, Apache, GPL, etc.)
- `homepage` - Project homepage URL
- `topics` - GitHub topics/tags (relevant keywords)
- `createdAt` - Repository creation date
- `pushedAt` - Last commit date
- `updatedAt` - Last update date
- `openIssues` - Count of open issues
- `defaultBranch` - Main branch name
- `isArchived` - Whether repo is archived
- `description` - GitHub repo description

## Step 4a: Hand-checked company facts (`metadata.company`)

The Details panel only has something to show when the scraper found GitHub stars,
users, a founding year, funding or ratings, and for most small vendors it found
none of them. Facts you read yourself - team size and company type off the
vendor's LinkedIn page, a launch date off its changelog, an office off its contact
page - go in `metadata.company`, **never** in `enriched_data`:
`enrich-production-tools.js` rebuilds `enriched_data` from scratch on every run, so
anything typed into it by hand is gone at the next monthly refresh.

```json
{ "team_size": "2-10 employees", "company_type": "Self-owned", "launched": "July 2026",
  "office": "Borongan, Philippines", "sources": ["LinkedIn", "dialsdone.com"],
  "checked_at": "2026-10-08" }
```

Every fact is optional; leave out what no source states. Write it with
`node scripts/set-tool-company-facts.js --input=facts.json` (dry run; `--execute`
writes and needs a go per Rule 1). It replaces only the `company` key.

## Step 5: Required Fields Reference

| Field | Type | Required | Example |
|-------|------|----------|---------|
| `name` | string | ✓ | ChatGPT |
| `slug` | string | ✓ | chatgpt |
| `tagline` | string | ✓ | AI-powered conversational assistant |
| `description` | string | ✓ | Brief 1-2 sentence description (150-160 chars for SEO) |
| `long_description` | text | ✓ | 2000+ chars: header-free intro, then ## How It Works, ## Core Features, ## Who This Is For |
| `website_url` | string | ✓ | https://chat.openai.com |
| `affiliate_url` | string | ✗ | https://affiliate.cc/your-link |
| `logo_url` | string | ✗ | Storage URL from fetch-tool-logos.js, or NULL if none |
| `screenshot_url` | string | ✗ | https://example.com/screenshot.jpg (primary; a second screenshot in `metadata.screenshots` is a MUST - see Screenshot Management) |
| `category_id` | UUID | ✓ | a1b2c3d4-e5f6... |
| `pricing_type` | enum | ✓ | free, paid, freemium, open_source |
| `price_from` | decimal | ✗ | 9.99 (or NULL) |
| `price_to` | decimal | ✗ | 99.99 (or NULL) |
| `pricing_info` | string | ✗ | Usage-based or $9.99/month |
| `featured` | boolean | ✗ | true or false |
| `sponsored` | boolean | ✗ | true or false |
| `tags` | array | ✓ | ['ai-writing', 'content-creation'] |
| `metadata` | JSONB | ✓ | Must carry `pricing` - see below. Plus `{"source": "..."}` |
| `github_url` | string | ✗ | https://github.com/org/repo (for open-source) |
| `enriched_data` | JSONB | ✗ | GitHub stats and metadata (see Step 4 format) |

**Auto-Generated Fields** (don't specify):
- `id`, `created_at`, `updated_at`, `upvotes`, `view_count`, `click_count`

## Step 4b: The pricing ladder (`metadata.pricing`) - REQUIRED

546 of the 548 tools in the directory carry this, and it is the single field the
site is built around. `/pricing`, `/pricing.md`, `/llms.txt`, the comparison
tables and the pricing panel on every tool page all read it. A row without it
renders as a tool with no price, which on this site is the one thing that must
never happen silently.

`pricing_info` is the human sentence. **This is the machine-readable version**,
and the two must agree.

### The shape

```json
{
  "source": "bulk-add",
  "verified_date": "2026-09-16",
  "pricing": {
    "url": "https://example.com/pricing",
    "checked_at": "2026-09-16",
    "currency": "USD",
    "billing_basis": "monthly",
    "free_kind": "tier",
    "has_free_tier": true,
    "unavailable_reason": null,
    "confidence": "high",
    "source": "in-session-extraction",
    "evidence": "Monthly toggle selected. Free $0. Pro $20 / month, 5,000 credits. Enterprise, Talk to sales.",
    "tiers": [
      { "name": "Free",       "monthly_usd": 0,    "self_serve": true },
      { "name": "Pro",        "monthly_usd": 20,   "self_serve": true,
        "annual_usd": 17, "annual_total_usd": 200, "per_seat": false,
        "limits": { "credits_per_month": 5000 } },
      { "name": "Enterprise", "monthly_usd": null, "self_serve": false }
    ]
  }
}
```

### Every field, and what to write when you cannot find it

`url`, `checked_at`, `evidence`, `tiers`, `free_kind` and `billing_basis` have **no
default**. If you cannot establish one, the row is not ready, and saying so is the
correct outcome rather than filling the gap.

| Field | Required | When you cannot determine it | Notes |
|---|---|---|---|
| `url` | always | `null` **and** `unavailable_reason: no_pricing_page` | The vendor's **own** pricing page. Never an aggregator. |
| `checked_at` | always | no default | `YYYY-MM-DD`, the day you loaded it. This is the date the page shows a reader. |
| `evidence` | always | no default | Verbatim from the page. Say which toggle you read and what you could not see. This is what settles a dispute between the ladder and the prose, so it is load-bearing, not decoration. 77 rows currently have none. |
| `tiers` | always | `[]` | An empty array is legitimate - 147 rows have one - and means no published price. Say why in `unavailable_reason`. |
| `free_kind` | always | `none` | `tier` = free forever. `trial` = time limited. `none` = neither. **A trial is not a free tier**, and that distinction is the whole point. |
| `billing_basis` | always | `unknown` | `monthly`, `annual_equivalent`, `one_time`, `usage`, `credits`, `unknown`. **Never assume monthly.** A capture whose toggle state you could not confirm is `unknown`. |
| `currency` | yes | `UNKNOWN` | `USD`, `EUR`, `RSD`, `INR`, `UNKNOWN`. Non-USD tiers use `monthly_<code>`. |
| `unavailable_reason` | yes | `null` when priced, otherwise a controlled value below | Never free text: `lib/comparison-rows.ts` sorts and filters on it. |
| `confidence` | yes | `low` | `high` you read the tiers directly. `medium` you inferred anything. `low` the page fought you. Not a measure of how well known the tool is. |
| `source` | yes | `in-session-extraction` | Also `pricing-page-capture`, `in-session-verification`, `manual-verify`. |
| `has_free_tier` | derived | never set by hand | `free_kind === 'tier'`. The checker errors when it disagrees. |
| `features` | optional | omit | On 8% of rows. Only when a plan's limits matter more than its price. |

### The tier object

| Field | Required | Default | Notes |
|---|---|---|---|
| `name` | always | none | Verbatim. **Do not fold a price into it.** `"$9 one-time, 40 credits"` as a name is data trapped in a string, and 290 tiers are shaped that way today. |
| `monthly_usd` | always | `null` | The month-to-month price. `null` means no monthly figure, which is **not** the same as "cannot be bought". |
| `self_serve` | always | `true` | `false` only when the tier cannot be bought online. This is what `price_from` and `price_to` filter on. |
| `annual_usd` | **when the page shows one** | omit | The annual-billed **monthly equivalent**: "$17/month, billed annually". |
| `annual_total_usd` | **when the page shows one** | omit | The **yearly charge**: "$200 billed up front", "$96/yr". A different number from `annual_usd`, and vendors show one, the other, or both. Record what is on the page; never divide one into the other. |
| `one_time_usd` | optional | omit | A lifetime or pack price. Keeps it out of `monthly_usd`, where seo-buddy's "$99 once, for good" was read as a monthly ceiling. |
| `per_seat` | optional | `false` | `true` when the figure is per user. scribe's "$25 per seat per month" is a different product from a $25 flat plan. |
| `limits` | optional | omit | Structured, not prose: `{ "credits_per_month": 5000, "seats": 3 }`. This is what readers actually compare on, and today it is stranded inside `evidence`. |

`annual_usd`, `one_time_usd`, `per_seat` and `limits` are additive. Rows written before
them stay valid and nothing derives from them yet. They exist so the next capture can
record what it saw instead of flattening it into a name.

### Annual pricing: three numbers, and a tier is a plan not a billing period

A vendor showing an annual discount puts up to three figures on one plan, and they
are three different numbers:

```
monthly_usd        20     what you pay month to month
annual_usd         17     the monthly equivalent when billed annually
annual_total_usd  200     what is actually charged, once a year
```

Record each one the page shows and omit the ones it does not. **Never derive one from
another.** looka publishes "$96/yr" and no monthly equivalent, so `annual_total_usd` is
96 and `annual_usd` is absent - writing 8 there would put a figure on the page that
the vendor never quoted. The saving is computed at render from the two it has; it is
not a stored field.

`monthly_usd` is always the month-to-month price. That is the rule `annual_equivalent`
was invented to dodge, and it cost the row its real price: perplexity's ladder holds
Pro 17 while its own prose correctly says $20/month, and both are true. With
`annual_usd` present the row holds both, and `billing_basis: annual_equivalent` shrinks
back to what it should always have meant - **the page offers no monthly price at all**,
not "we recorded the annual one".

**A tier is a plan, never a billing period.** One plan sold monthly and yearly is one
tier with two figures, not two tiers. Eleven rows currently break this and every one of
them corrupts the range, because a yearly-equivalent rate gets counted as a buyable
monthly plan:

```
facetune                 Monthly 25, Quarterly 13.33, Yearly 6.5   ->  price_from 6.50
reverse-image-location   Monthly 19.99, Yearly 5.83                ->  price_from 5.83
```

Nobody can pay facetune $6.50 for a month. Correctly modelled that row is one tier,
`monthly_usd: 25` with `annual_usd: 6.50`, and `price_from` is 25.

`scripts/check-pricing-consistency.js` warns when a tier is named after a billing
period. It is a warning rather than an error because collapsing the tiers means
re-reading the vendor page to find out which plan they belong to.

### What derives from this, and from nothing else

```
price_from      cheapest monthly_usd among tiers where self_serve
price_to        dearest  monthly_usd among tiers where self_serve
pricing_type    a paid tier plus a free one -> freemium; paid only -> paid;
                all zero -> free; open_source is exempt
has_free_tier   free_kind === 'tier'
```

`scripts/apply-pricing-extraction.js` computes the first two - do not set them by hand.
`scripts/check-pricing-consistency.js` proves all four and exits 1 on a contradiction.

### `self_serve`, and why the range needs it

`price_from` and `price_to` are not the ladder's minimum and maximum. They are the
cheapest and dearest tier **a reader can actually buy**, and that filter is the one fact
the ladder did not record.

Without it the two cannot be derived, and the consequences run both ways. langfuse's
ladder holds `Enterprise $2499`, which its own evidence marks as "Talk to sales, paid by
invoice rather than self-serve card" - deriving `price_to` from the maximum would put a
price on the page that nobody can pay. Meanwhile 16 rows have a column *wider* than
their ladder, and reading their evidence shows most of those numbers appear nowhere on
the vendor page at all: grammarly's stored $15 against a ladder of Free/$12/Enterprise,
scribe's $29 against $25/$13, seo-buddy's $99 which the page describes as "$19 per
month, per domain - or $99 once, for good" - a lifetime price sitting in a monthly
column.

So mark the tier:

```json
"tiers": [
  { "name": "Core",       "monthly_usd": 29,   "self_serve": true  },
  { "name": "Enterprise", "monthly_usd": 2499, "self_serve": false }
]
```

Then both columns are computed rather than kept:

```
price_from = cheapest monthly_usd where self_serve
price_to   = dearest  monthly_usd where self_serve
```

Do not infer `self_serve` from a null `monthly_usd`. 290 tiers have no monthly figure
and only 38% of those are Enterprise or Custom; the rest are one-time packs and credit
bundles - "$9 one-time, 40 credits", "Career Pass, $69 one-time, lifetime access" -
which are perfectly buyable and simply not monthly. A null price means "no monthly
figure", not "cannot be bought".

Until every tier on a row carries `self_serve`, the checker treats a `price_from` or
`price_to` disagreement as a warning. Once they all do, it is an error, because there is
then nothing left for the difference to legitimately mean.

### `unavailable_reason`, the controlled list

`contact_sales`, `usage_based`, `one_time_purchase`, `lifetime_deal`,
`not_published`, `no_pricing_page`, `login_required`, `non_usd`,
`page_unreachable`. The type lives in `lib/comparison-rows.ts` as
`PricingUnavailableReason`. Free text here is a bug: `lib/comparison-rows.ts`
sorts and filters on these, and prose cannot be sorted.

### The rules

- **Only the vendor's own page counts.** An aggregator quoting a price is a lead
  to check, not a fact to record. If the vendor publishes nothing, record that -
  `tiers: []` with `unavailable_reason` - rather than importing someone else's
  number.
- **Say which toggle you read.** Most sites default to annual pricing shown as a
  monthly figure. If you record the annual-equivalent number, `billing_basis`
  is `annual_equivalent`, not `monthly`.
- **`confidence: "high"` means you saw the tiers.** Not that the tool is
  well known.
- **Never pad `evidence` to look thorough.** "No price appears anywhere on the
  site without an account" is a better sentence than an invented summary.

### Note on the TypeScript type

`ToolPricingRecord` in `lib/comparison-rows.ts` describes only the fields the
comparison pages consume. It omits `currency`, `source`, `confidence`,
`evidence` and `has_free_tier`, which are present on 97% of rows. The type is
not the schema - this table is.

## Step 4c: The pricing pipeline - what must move together

A price lives in seven fields, written by different processes at different times:

**`metadata.pricing` is the source of truth.** Everything else about price is derived
from it or must agree with it. It is also the product: the verified, dated ladder is
what this directory sells, so it carries the facts and nothing points at prose.

| Field | Derived from the ladder? | What it drives |
|---|---|---|
| `price_from` / `price_to` | **yes** - cheapest and dearest `self_serve` tier | the range and the sort key |
| `pricing_type` | **yes** - the ladder's shape | the badge, the filter facet |
| `has_free_tier` | **yes** - `free_kind === 'tier'` | the free-tier facet |
| `price_since` | written on a real price move | "unchanged since August 2026" |
| `pricing_info` | must quote the ladder's figures | the prose sentence |
| `description` | must not contradict it | falls through into the meta tag |
| `metadata.seo_description` | must not contradict it | the Google snippet |

The ladder is an **observation with evidence and a date**, not an oracle. It can be
incomplete (cursor's page renders $20 and $40 but not the $200 Ultra tier) and it can be
of the wrong product entirely (anthropic-api's was captured from claude.com/pricing, so
it holds Claude's consumer plans while the prose correctly says "$3/$15 per million
tokens"). When the ladder and the prose disagree, `evidence` decides, which is why it is
required rather than decorative.

They drifted apart because nothing compared them. On 2026-09-18 an audit found **87
rows contradicting themselves** and **148** whose prose quoted a price the ladder did
not hold. `mintlify` told Google "a steep jump to $120 a month" while its own panel
said the Starter plan was free.

### The rule

**Never change one price field alone.** Whatever writes a price writes the set, and
proves it afterwards. `scripts/apply-pricing-extraction.js` computes `price_from` and
`price_to` from the ladder's `self_serve` tiers rather than accepting them, so on a
fully marked row they cannot drift. Do not set them by hand.

```bash
node scripts/check-pricing-consistency.js --slugs=<the rows you touched>
```

### Re-running the sweep over rows that already have a ladder

`apply-pricing-extraction.js` **merges** tiers rather than replacing them. A fresh
capture supplies `name` and `monthly_usd` for certain; `self_serve`, `annual_usd`,
`annual_total_usd`, `per_seat` and `limits` are carried forward from the existing tier
of the same name when the new extraction does not supply them. A value the extraction
*does* supply always wins - the merge fills gaps, it never overrides what was just read
off the page.

Tiers match by name, trimmed and case-insensitive, because that is the only stable
identity a tier has. A renamed plan starts fresh, which is right: the old plan's annual
figure is not evidence about the new one. A dropped plan disappears, also right.

This is what makes a full re-sweep safe. Before it, re-running the pipeline over the
whole directory would have replaced every tier with name-and-price only, wiping 1,368
`self_serve` marks and 35 annual figures - and because `self_serve` is what the range is
computed from, `price_from` and `price_to` would have quietly become underivable again.
Nothing would have failed. The run would have reported success.

Read only, anon key, exits 1 on a contradiction. `scripts/apply-pricing-extraction.js`
runs it automatically on the rows it just wrote.

### Adding a new tool

The pricing ladder is required - Step 4b has the field list. Then, before the dry run,
confirm these agree, because the checker will:

- `price_from` equals the cheapest ladder tier, `price_to` the dearest
- `pricing_type` matches the ladder: any paid tier plus a free one is `freemium`, paid
  only is `paid`, all-zero is `free`. `open_source` is exempt
- `has_free_tier` equals `free_kind === 'tier'`
- no priced tier means an `unavailable_reason` from the controlled list, never silence
- `pricing_info` quotes the same figures as the ladder, or the difference is explained
  by `billing_basis`

### Updating a price

`scripts/apply-pricing-extraction.js` writes `price_from`, `price_to`,
`metadata.pricing`, `price_since`, and `pricing_info` when the extraction supplies one
**and the price actually moved**.

Prose is rewritten only on a real move. Good `pricing_info` carries credits, seats and
annual figures that a generated sentence would throw away, so an unchanged price leaves
it alone.

`description` and `seo_description` are never written automatically. They are editorial
and SEO copy, and `anthropic-api` is the reason: its prose says "$3/$15 per million
tokens", which is correct for the API, while its ladder holds claude.ai consumer plans
the crawler read off the wrong product. Auto-syncing would have destroyed the correct
text to match the wrong number. The checker reports them; a human edits them.

### Errors and warnings are different things

**Errors** are two fields on one row disagreeing about something no vendor page is
needed to settle: `price_to` against the ladder, `pricing_type` against the ladder,
`has_free_tier` against `free_kind`. Always wrong. Fix before shipping.

**Warnings** are a price in prose that the ladder does not contain. Often correct, and
undecidable from the row:

- `billing_basis: annual_equivalent` - Perplexity's ladder holds $17 because the page
  says "$17/month when billed annually". The prose says $20, month-to-month. Both right.
- `unavailable_reason: usage_based` - usage pricing has no subscription tier to match.
- intro versus renewal - Recapo shows "$38.99/mo" beside "renews at $42.99/mo next
  month". Both are real.

Those three cases are exempted automatically. What remains is the weekly sweep's
worklist, and it needs the vendor page, not a script.

## Step 5: Tag Selection Formula

For each tool:
1. **What's the PRIMARY purpose?** (Which subsection fits best?)
   → Use keywords from that subsection
2. **What related tags apply?**
   → Add 2-3 more related tags from the same subsection
3. **Total: 3-5 tags** matching subsection keywords

### Example: Jasper AI (Writing Tool)
- Category: Writing
- Primary Purpose: Copywriting and marketing content
- Tags: `['copywriting', 'marketing', 'content-generation', 'saas']`
- Expected Subsection: Copywriting & Marketing ✓

## Step 6: Finding Logo URLs

> **Clearbit is gone.** `logo.clearbit.com` was retired and the hostname no longer
> resolves in DNS, so any `logo_url` pointing at it renders as a broken image. Do not
> use it, and treat it as a red flag if you see it in older rows or older docs.

Logos come from the tool's own site instead:

```bash
node scripts/fetch-tool-logos.js --input=batch.json            # download + quality gate only
node scripts/fetch-tool-logos.js --input=batch.json --upload   # also upload to storage
```

Input JSON is the same shape the screenshot script takes:
```json
[{ "slug": "mowgli", "url": "https://mowgli.ai" }]
```

The script pulls the Open Graph image (falling back to favicon), quality-gates it,
normalises to a 512px PNG, and with `--upload` writes to the `tool_images` bucket under
`tool_logos_new/<slug>.png`. It **never touches the `tools` table** - it prints the
`logo_url` to paste into your SQL and writes `<input>-logos.json`.

Resulting URL shape:
```
https://[project].supabase.co/storage/v1/object/public/tool_images/tool_logos_new/[slug].png
```

If no acceptable logo can be fetched, leave `logo_url` NULL rather than inventing a URL.

## Step 7: Getting Category IDs

Run this SQL in Supabase SQL Editor:
```sql
SELECT id, name, slug, description FROM tool_categories ORDER BY name;
```

Then use the category slug in the INSERT query:
```sql
(SELECT id FROM tool_categories WHERE slug = 'writing')
```

## Step 8: Check for Enrichment Sources (NEW - CRITICAL)

Before generating SQL, check if the tool has enrichment sources:

### GitHub URL (For Open-Source Tools)
- [ ] Look for GitHub link on the tool's website
- [ ] Verify repository is active and public
- [ ] Note the full GitHub URL: `https://github.com/org/repo`
- [ ] This enables auto-fetching: stars, forks, language, topics, license

### Wikipedia Page (For Major Companies)
- [ ] Check if it's a major company tool (Canva, OpenAI, Anthropic, etc.)
- [ ] Search Wikipedia: `https://en.wikipedia.org/wiki/Company_Name`
- [ ] Verify the page exists and is about this company
- [ ] Note the page title (e.g., "Canva", "ChatGPT")
- [ ] This enables auto-fetching: founded, users, revenue, employees, headquarters

**Why This Matters:**
- Tools with GitHub URLs show real-time GitHub stats on the website
- Tools with Wikipedia pages show company information
- Enrichment runs automatically when the tool is added
- Missing these = tool won't have stats displayed

## Step 9: SQL Template (Use This)

**This shape is not cosmetic.** `scripts/apply-migration.js` parses the file with
a regex rather than executing raw SQL, so it accepts exactly one layout. Three
things break it, and all three used to be in this template:

- **An inline comment anywhere in the value list.** `NULL, -- logo_url` is read
  as a value and fails with `cannot parse value`. Put comments above the
  statement, never between values.
- **A closing paren before `WHERE NOT EXISTS`.** There is none:
  `INSERT INTO tools (cols) SELECT vals WHERE NOT EXISTS (...)` is a SELECT with
  no FROM, which needs no bracket. A stray `)` is also a Postgres syntax error.
- **The guard split across lines.** It has to be on one line.

The column list is fixed at these 19, in this order. `wikipedia_slug` and
`producthunt_slug` are real columns but the applier does not handle them, so set
them with a separate UPDATE afterwards if you need them. (`producthunt_slug`
enriches nothing, see the note below.)

```sql
-- ALWAYS verify information before filling in these values
-- Enrichment: [GitHub: URL/None | Wikipedia: Page/None]
-- website_url: verify the domain loads. logo_url and screenshot_url: fill in
-- from the fetch-tool-logos.js and capture-tool-screenshots.js output, or NULL.
-- price_from / price_to: only if confirmed on the vendor pricing page.
INSERT INTO tools (
  name, slug, tagline, description, long_description, website_url,
  affiliate_url, logo_url, screenshot_url, category_id, pricing_type,
  price_from, price_to, pricing_info, featured, sponsored, tags,
  metadata, github_url
) SELECT
  '[Tool Name - FROM OFFICIAL WEBSITE]',
  '[tool-slug]',
  '[Tagline - FROM THEIR MARKETING/HOMEPAGE]',
  '[1-2 sentence description, 150-160 chars - FROM THEIR SITE]',
  '[Detailed 5-part format - FROM OFFICIAL DOCS/MARKETING]',
  'https://[official-domain.com]',
  NULL,
  NULL,
  NULL,
  (SELECT id FROM tool_categories WHERE slug = '[category-slug]'),
  '[free/paid/freemium/open_source]',
  NULL,
  NULL,
  '[Pricing description - FROM PRICING PAGE]',
  false,
  false,
  ARRAY['tag1', 'tag2', 'tag3', 'tag4', 'tag5'],
  -- metadata MUST carry the pricing ladder. See Step 4b for every field and the
  -- controlled values; a row without it renders as a tool with no price.
  -- Every tier needs self_serve. price_from and price_to stay NULL below: they are
  -- DERIVED from the ladder, and apply-pricing-extraction.js computes them.
  '{"source": "bulk-add", "verified_date": "[YYYY-MM-DD]", "pricing": {"url": "[vendor pricing page]", "checked_at": "[YYYY-MM-DD]", "currency": "USD", "billing_basis": "monthly", "free_kind": "[tier|trial|none]", "has_free_tier": [true|false], "unavailable_reason": null, "confidence": "high", "source": "in-session-extraction", "evidence": "[one sentence a reader could check]", "tiers": [{"name": "Free", "monthly_usd": 0, "self_serve": true}, {"name": "Pro", "monthly_usd": 20, "self_serve": true, "annual_usd": 17, "annual_total_usd": 200}]}}'::jsonb,
  '[https://github.com/org/repo]'
WHERE NOT EXISTS (SELECT 1 FROM tools WHERE slug = '[tool-slug]');

-- After execution, run enrichment for the new tool:
-- node scripts/enrich-production-tools.js --tool-slug=[tool-slug]
--
-- A second screenshot is a MUST (see Screenshot Management). screenshot_url above is
-- only the primary. Capture at least one more 1280x720 view and set the ordered set:
-- node scripts/set-tool-screenshot-set.js --slug=[tool-slug] --urls=[primary],[second] --execute
```

Values are positional, so `NULL` for a real NULL and a quoted string otherwise.
The guard slug must match the row slug or the applier refuses the batch, which is
deliberate: it catches a copy-paste that changed one and not the other.

Before asking for approval, confirm the applier actually parsed what you wrote.
`node scripts/apply-migration.js <file>` prints one line per row; if it says
`N INSERT statements but only 0 parsed`, the layout is wrong and nothing in the
file has been understood.

### Reading validate-tool-sql.js output

Errors are real and it exits non-zero on them. Warnings need one moment of judgement.

The `description N chars (target 150-160)` warning is a heuristic: it scans every
single-line string in the plausible length band, so it also picks up `pricing_info`
and sometimes `tagline`. Check which field the quoted text is actually from before
editing anything. A run of this batch produced three such warnings and all three
were `pricing_info` or `tagline`; the five real `description` values were inside the
window and none of them were flagged. **Do not pad a `tagline` or truncate an honest
`pricing_info` to silence a warning** - that is trading real information for a green
line. What the check is genuinely for is the `description` column, which is the meta
description and has an SEO length window.

## Alternative: Auto-Enrichment Script

For faster workflow with automatic enrichment:

```bash
node scripts/add-tool-enhanced.js \
  --name="Tool Name" \
  --url="https://example.com" \
  --github="https://github.com/org/repo" \
  --wikipedia="Wikipedia_Page_Title" \
  --category="coding" \
  --description="Brief description"
```

**This script:**
- Auto-fetches GitHub stats if github_url provided
- Auto-scrapes website for company info
- Auto-fetches Wikipedia data if wikipedia_slug provided
- Interactive category selection
- Preview before saving
- Enriches immediately (no separate step needed)

See `ENRICHMENT-GUIDE.md` for complete enrichment system documentation.

## Adding New Categories with Subsections

A new category touches **four places**. The database is only the first, and the
three code changes fail *silently* - the category renders, just wrong - so none
of them will announce itself.

### 1. The category row

```sql
INSERT INTO tool_categories (name, slug, description, icon)
SELECT 'New Category', 'new-category', 'Description', 'Sparkles'
WHERE NOT EXISTS (SELECT 1 FROM tool_categories WHERE slug = 'new-category');
```

The column is `icon`, not `icon_name`. There is no `icon_name` column and there
never was.

**And `icon` renders nothing.** It is populated on all 18 categories with
plausible lucide names and is read by no code anywhere in this repository - the
icons come from the slug maps in step 3. The stored values have already drifted
from what actually renders: the row for `meetings` says `Users`, the code draws
`Presentation`. Fill it for tidiness if you like, but do not expect it to do
anything, and do not trust it when reading.

`scripts/apply-migration.js` cannot apply this. It only parses the Step 9
`INSERT INTO tools` shape and refuses UPDATEs and other tables, so a category
needs a purpose-built script - see `scripts/add-fun-personal-category.js`.

### 2. Subsections, in `lib/category-subsections.ts`

```typescript
export const categorySubsections: Record<string, Subsection[]> = {
  'new-category': [
    {
      id: 'subsection-1',
      categoryId: 'new-category',
      name: 'Subsection Name 1',
      slug: 'subsection-1',
      description: 'Description',
      keywords: ['keyword1', 'keyword2', 'keyword3'],
      sortOrder: 0,
    },
    // 4-6 subsections total
  ],
};
```

Derive `keywords` from the tags the tools **actually carry**, not from an ideal
vocabulary, then prove it:

```bash
node scripts/check-subsection-routing.js batch.json
```

A tool matching no keyword does not error. It falls into the first subsection and
looks fine. That script is the only thing that catches it.

### 3. Both icon maps

There are two, they use different icon libraries, and each has its own silent
fallback. Miss one and the category renders a generic mark on that surface only:

| File | Library | Fallback if the slug is missing |
|---|---|---|
| `components/layout/Sidebar.tsx` | lucide | `Layers` |
| `components/layout/MobileSidebar.tsx` | lucide | `Layers` |
| `components/CategoryIcon.tsx` | Phosphor | `Cube` |

The two sidebars must stay identical to each other. They drifted once: a fix that
added `meetings`, `research` and `sales` to the desktop map left mobile alone, so
four categories shared one icon on phones while desktop looked correct.

**Check it by rendering, not by reading the diff.** Two category rows sharing an
SVG path is the symptom:

```js
const rows = [...document.querySelectorAll('aside a')].map((a) => ({
  label: a.textContent.replace(/\d+$/, '').trim(),
  sig: a.querySelector('svg path')?.getAttribute('d')?.slice(0, 24),
}));
// group by sig - any group of 2+ is a missing mapping
```

### 4. Deploy, then add tools

Subsections are compiled in, so the category page is empty until the code ships.
Then add tools whose tags match the keywords from step 2.

---

# Logo Management

## Fetch Logos with Quality Gates

```bash
node scripts/fetch-tool-logos.js --input=batch.json
node scripts/fetch-tool-logos.js --input=batch.json --upload
```

Input JSON: `[{ "slug": "mowgli", "url": "https://mowgli.ai" }, ...]`

### What This Script Does:
1. Fetches the Open Graph image from each tool's website, falling back to favicon
2. Enforces the quality gates below
3. Normalises output to a 512px PNG
4. Saves to `.logos-temp/` for manual review
5. Uploads to the `tool_images` bucket under `tool_logos_new/` **only** with `--upload`
6. **Does NOT modify the tools table** - prints the `logo_url` and writes `<input>-logos.json`

### Quality Gate Standards:
- **MIN_BYTES:** 1500 (a flat single-colour logo can be tiny and still be fine)
- **MAX_BYTES:** 5000000
- **MIN_DIMENSION:** 96px
- **OUTPUT_SIZE:** 512px
- **REJECTED_FORMATS:** ICO
- `--relaxed` lowers the floors to 32px for sites that only ship a tiny favicon

### Workflow:
1. Run the script **without** `--upload`
2. Review logos in `.logos-temp/`
3. Replace any that came out wrong by hand
4. Re-run with `--upload` once they look right
5. Paste the printed `logo_url` values into your migration SQL

To backfill rows that already exist, use `scripts/set-tool-media.js --logos=<input>-logos.json`
(dry run by default, `--execute` to write).

---

# Screenshot Management

## Upload Screenshots to Storage

### Screenshot convention (IMPORTANT)

**1280x720 output, captured at 80% page zoom.** A 1280x720 frame at 100% zoom slices
through whatever element sits at the fold, which almost always cuts the hero CTAs in
half. `scripts/capture-tool-screenshots.js` renders a 1600x900 viewport and scales it
into the 1280x720 frame, reproducing "1280x720 at 80%" in Chrome DevTools device mode.
It also dismisses cookie/consent banners before capturing.

```bash
node scripts/capture-tool-screenshots.js --input=batch.json --upload
```

Stored at `website_images/screenshots/<slug>.png`.

Input JSON: `[{ "slug": "mowgli", "url": "https://mowgli.ai" }, ...]` (same shape as
`fetch-tool-logos.js`, so one file drives both).

### What This Script Does:
1. Loads each URL, dismisses cookie/consent banners, and captures the 1280x720 frame
2. Saves to `.screenshots-temp/` for review
3. Uploads to the `website_images` bucket under `screenshots/` **only** with `--upload`
4. **Does NOT modify the tools table** - prints the `screenshot_url` and writes
   `<input>-screenshots.json`

### Workflow:
1. Run **without** `--upload` and review `.screenshots-temp/`
2. Re-run with `--upload` once they look right
3. Paste the printed `screenshot_url` values into your migration SQL
4. To backfill rows that already exist, use
   `scripts/set-tool-media.js --screenshots=<input>-screenshots.json` (dry run by
   default, `--execute` to write)

`scripts/capture-screenshots.js` is a separate older script that sweeps every tool in
the database rather than a supplied list. Prefer `capture-tool-screenshots.js` for new
tools, since it is the one that implements the 80% zoom convention above.

## A second screenshot is REQUIRED on every tool (MUST)

Every tool MUST ship with at least two screenshots. One image is the primary,
`screenshot_url`, which cards, OG images and the image sitemap read. The second (and
any further) image lives in the ordered array `metadata.screenshots`, which
`components/ToolHeroSection.tsx` renders as a carousel on the tool page's Overview
tab. A single screenshot gives the page nothing to turn through and leaves the gallery
flat.

How the two fit together, so a one-image tool never regresses: when
`metadata.screenshots` is absent the hero falls back to `[screenshot_url]` and the page
renders exactly as it always has. The carousel only appears once `metadata.screenshots`
holds more than one URL, and the first URL in that array SHOULD equal `screenshot_url`
so the hero and the card agree on the tool's face.

### When adding a new tool (NON-NEGOTIABLE)

Capturing the primary and leaving it at one image is not done. Before the tool is
considered complete:

1. Capture the primary with `scripts/capture-tool-screenshots.js` (the 80% zoom frame
   above) and set it as `screenshot_url`.
2. Capture or source at least one more distinct view - a second feature screen, the
   pricing page, a results/output view - and run it through the same 1280x720 frame so
   the carousel does not change shape between slides.
3. Set the ordered set with `scripts/set-tool-screenshot-set.js --slug=<slug>
   --urls=<primary>,<second>[,...]` (dry run by default; `--execute` writes the row and
   needs a human go per Rule 1). The script refuses fewer than two URLs on purpose,
   because one is already what `screenshot_url` does on its own.
   `scripts/set-tool-gallery.js` is the alternative when you have local image files to
   upload and normalise first (`--upload` writes storage, `--execute` writes the row -
   two separate flags, both needed).

A tool with only one screenshot is incomplete. Treat the second screenshot the same way
as the pricing ladder: required on the row, not a nice-to-have.

### Backfilling tools that only have one

Tools that predate this rule carry only `screenshot_url` and no `metadata.screenshots`.
Give each one a second image with the same capture → normalise → `set-tool-screenshot-set.js`
flow above. `metadata.screenshots` is a JSONB key, so
`scripts/set-tool-screenshot-set.js` merges it in without disturbing the pricing record
already in `metadata`. As with every row write, post the dry-run output and get a human
go before `--execute`.

---

# Google Analytics 4 (GA4) Tracking

### Setup Status: ✅ COMPLETE

**Measurement ID:** G-LP963PZ3MZ
**Dashboard:** https://analytics.google.com/

### What's Being Tracked

#### Automatic Tracking:
- **Page views** - Every page visit
- **Route changes** - Client-side navigation
- **Traffic sources** - Where users come from
- **User location** - Geographic data
- **Device info** - Desktop vs mobile, browser

#### Custom Events:
- **click_visit_website** - When users click "Visit Website" buttons
  - Tracks tool name as label
  - Measures affiliate traffic potential
- **upvote** - When users upvote a tool
  - Tracks tool engagement
  - Measures user interest

### Dashboard Reports
- **Real-Time** - Users on site now, what pages they're viewing
- **Pages and screens** - Which tools get the most views
- **Events** - Click counts, upvotes, website visits
- **Demographics** - User location, devices, browsers
- **Acquisition** - How users found the site (Google, social, direct)

### Key Metrics to Monitor
- **Most viewed tools** - Identifies popular categories
- **Click-through rates** - Affiliate performance
- **User engagement** - Upvotes, time on page
- **Traffic sources** - What drives users to the site

---

# SEO Best Practices

## Meta Descriptions

### Requirements:
- **Optimal length:** 150-160 characters (longer descriptions get truncated)
- **What to update:** Tool descriptions in database, blog post excerpts
- **Process:** Update → Test in search preview → Verify character count

### Guidelines:
- [ ] Keep to 150-160 characters maximum
- [ ] Include tool name + primary benefit
- [ ] Avoid duplicate descriptions across similar tools
- [ ] Reference actual tool capabilities, not generic hype
- [ ] Use tool's own language when possible

### Example Meta Description Updates

#### Tool: Claude
**Old (256 chars):** "AI assistant for complex refactoring and architectural decisions. Claude is Anthropic's AI assistant designed for thoughtful, detailed responses. Excellent for understanding messy legacy code, refactoring entire classes, and making architectural decisions."

**New (149 chars):** "AI assistant for complex refactoring and architectural decisions. Excellent for understanding legacy code and making architectural choices."

#### Tool: SuperAGI
**Old (190 chars):** "Advanced framework for complex task automation. SuperAGI provides an advanced open-source framework for building and deploying autonomous agents capable of sophisticated workflow automation."

**New (155 chars):** "Advanced framework for building autonomous AI agents capable of sophisticated workflow automation and complex task execution."

## Other SEO Best Practices
- Include proper meta tags on all pages
- Use semantic HTML elements
- Implement structured data where relevant
- Optimize images with alt text
- Ensure fast page load times
- Use proper heading hierarchy (h1 → h6)

---

# Common Tasks Quick Reference

## "I want to add a new tool"
1. Complete information verification checklist (see Step 1 above)
2. Verify pricing on official pricing page
3. Write long description in 5-part format (2000+ chars)
4. Choose appropriate category and tags
5. Copy and customize SQL template
6. **Generate SQL file first**
7. **Show it for review**
8. **Wait for approval**
9. Execute only after approval

## Applying a migration

`scripts/apply-migration.js` is how a batch reaches the database. There is no
`psql` on the agent machines and no `DATABASE_URL` in `.env.local`, so it replays
the migration through the Supabase client instead of executing raw SQL. It parses
the INSERTs out of the `.sql` rather than restating the data, so the file stays the
source of truth and what gets written is what was reviewed, and it reproduces the
`WHERE NOT EXISTS` guard by checking each slug first. A re-run is a no-op, not a
duplicate. It refuses a file it cannot fully parse rather than applying half of it.

**It is dry run by default. `--execute` is the only thing that writes, and Rule 1
says you ask before you pass it.**

```bash
node scripts/apply-migration.js migrations/ADD-TOOLS-<date>-<slug>.sql            # dry run, ask
node scripts/apply-migration.js migrations/ADD-TOOLS-<date>-<slug>.sql --execute  # after the yes
```

The whole sequence, in order:

```bash
node scripts/validate-tool-sql.js migrations/<file>.sql        # content rules
node scripts/check-subsection-routing.js batch.json            # tags reach a real subsection
node scripts/apply-migration.js migrations/<file>.sql          # dry run -> post it and ASK
# ---- wait for an explicit yes ----
node scripts/apply-migration.js migrations/<file>.sql --execute
node scripts/enrich-production-tools.js --tool-slug=<slug>     # once per new tool
node scripts/find-dark-logos.js --write                        # regenerate lib/dark-logos.ts
# apply-migration.js --execute now checks the newly inserted logos itself
```

Then read the rows back with the **anon** key and open a `/tools/<slug>` page on
production. The database accepting a row is not the same as the page rendering it.

Once a migration is applied it has done its job. Delete the file and commit the
deletion; `migrations/` is a staging area, not an archive, and the text stays in git
history either way.

**Both media scripts and `apply-migration.js` need `SUPABASE_SERVICE_ROLE_KEY`**
(`fetch-tool-logos.js --upload`, `capture-tool-screenshots.js --upload`). That is
expected, not a workaround. `find-dark-logos.js` needs it too but only reads.

## After adding ANY tool (do not skip these)

Four steps that are easy to forget and produce silent breakage:

```bash
# 1. Enrich EVERY new tool, not just the open-source ones. The website scraper
#    returns founded year, employees, ratings and pricing even with no GitHub URL.
node scripts/enrich-production-tools.js --tool-slug=<slug>

# 2. Write metadata.pricing. The INSERT template has no column for it, so a tool
#    added through that template has prices in the database and no pricing panel
#    on its page. Stage the tiers in .pricing-temp/proposals.json and apply.
#    Every tier needs self_serve; see Step 4b.
node scripts/apply-pricing-extraction.js            # dry run, then --execute

# 3. Prove the price fields agree. metadata.pricing is the source of truth and
#    price_from, price_to, pricing_type and has_free_tier derive from it. This
#    exits 1 when a row contradicts itself. See Step 4c.
node scripts/check-pricing-consistency.js --slugs=<the new slugs>

# 4. Re-detect logos that are invisible on the dark chip. The list is static,
#    so a new dark logo is not picked up until this is re-run.
node scripts/find-dark-logos.js --write
```

**Why step 2 matters, and why it keeps being missed:** `ToolPricingPanel` renders
the tier ladder from `metadata.pricing`. The 19-column INSERT template carries
`price_from`, `price_to` and `pricing_info` but has nowhere to put that record,
so nothing about the migration looks wrong and the page simply shows no pricing.
On 2026-09-17 all 16 tools added that day landed this way, in three separate
batches, including one added straight after the same gap had been fixed for an
earlier batch. The row looks complete in every check except opening the page.

One thing to get right when staging it: a price of zero and an
`unavailable_reason` are contradictory. Free is a known price, not a price you
could not find. Set the reason only when there genuinely is no figure to
record - `apply-pricing-extraction.js` refuses the row otherwise.

**Why step 1 matters:** `enrich-production-tools.js` reads three sources.
`github_url` works, the website scraper works, and **`producthunt_slug` does
not**: Product Hunt returns HTTP 403 to `lib/scrapers/producthunt.js`, so
setting that column enriches nothing. Running enrichment on every tool is still
worth it for the website data.

**Why step 2 matters:** `lib/dark-logos.ts` is generated, not computed at
runtime. A logo that is dark ink with no bright pixels disappears against the
dark chip until its slug is in that file.

Step 2 is the one that gets skipped, because nothing fails when you skip it.
The chip renders empty and nobody notices. It has been missed twice, on
`copilotkit` and again on `gpt-image-3`, which sat with an invisible logo from
2026-09-01 until a routine re-run happened to catch it.

`apply-migration.js --execute` now checks this itself. After inserting, it
analyses **only the logos it just inserted** and prints a loud failure with a
non-zero exit if any of them needs a light chip it does not have. That costs one
fetch per new tool rather than the ~460 a full run does. It never fails the
migration itself: the rows are already written by then, and a check that cannot
run says so rather than pretending the write failed.

Two other modes exist for checking without regenerating:

```bash
node scripts/find-dark-logos.js --check           # exit 1 if lib/dark-logos.ts is stale
node scripts/find-dark-logos.js --slugs=a,b,c     # check only these slugs
```

`--check` walks every logo, so it suits a cron rather than an inner loop.
**Neither belongs in the verify gate**: both need `SUPABASE_SERVICE_ROLE_KEY`
and both fetch logos over the network, and the gate is credential-free by design
so it runs on a fresh clone with nothing but `npm ci`.

## "Logos are missing for some tools"
1. Build an input file: `[{ "slug": "...", "url": "https://..." }]`
2. Run: `node scripts/fetch-tool-logos.js --input=batch.json`
3. Review logos in `.logos-temp/` folder
4. Re-run with `--upload`, then apply the printed URLs
5. Wait for approval before writing them to existing rows

## "I need to take screenshots of tools"
1. Build an input file: `[{ "slug": "...", "url": "https://..." }]`
2. Run: `node scripts/capture-tool-screenshots.js --input=batch.json`
3. Review screenshots in `.screenshots-temp/`
4. Re-run with `--upload`, then paste the printed URLs into your SQL
5. Wait for approval before writing them to existing rows

## "I need to update tool descriptions"
1. Verify information on official website
2. Use 5-part long description format
3. Write 2000+ characters
4. Generate SQL UPDATE statements
5. Show for review
6. Wait for approval
7. Execute

## "I want to email tool submitters"
1. Read `OUTREACH-GUIDE.md` first - the email lives on `tool_submissions`, not `tools`
2. Build: `node scripts/build-outreach-emails.js --variant=free-trial-5 --limit=20 --out=batch1.json`
3. Review the JSON and delete anyone you do not want
4. Dry run: `node scripts/send-outreach-emails.js --input=batch1.json`
5. Send: add `--execute` (30s spacing, marks `outreach_variant` per recipient)
6. Confirm delivery in the Resend dashboard, not the script output

## "I want to track analytics"
1. Access dashboard: https://analytics.google.com/
2. Check Real-Time report for current users
3. Review "Pages and screens" for tool popularity
4. Check "Events" for affiliate clicks
5. Monitor "Acquisition" for traffic sources

## "Meta description needs updating"
1. Check current length (must be 150-160 chars)
2. Trim to size if needed
3. Verify it accurately describes the tool
4. Update in database
5. Test in Google search preview

---

# The pricing sweep - proposals.json shape

`capture-pricing-pages.js` -> `extract-pricing.js` -> `apply-pricing-extraction.js`
passes work between steps as `.pricing-temp/proposals.json`. Anyone running the
sweep reads this file before each run, so changes to the shape or to the
controlled `unavailable_reason` values belong here.

**Changed 2026-09-15: `final_url` is now part of a proposal and is what the
apply step checks.**

```jsonc
{
  "slug": "ada",
  "pricing_url": "https://www.ada.support/pricing",  // what we asked for
  "final_url":   "https://www.ada.cx/demo/",         // where we landed  <- NEW
  "extraction": { /* price_from, price_to, tiers, evidence, ... */ }
}
```

`final_url` is optional and falls back to `pricing_url` when absent, so a
hand-written proposal still works: nothing fetched it, so it never redirected.

Two reasons a proposal is now refused as `host_mismatch` where it previously
would have been written:

- **The landed host differs from the tool's.** The capture already recorded and
  checked `final_url`, but the apply step re-checked `pricing_url`, so a redirect
  the capture had flagged was invisible by write time. `ada` (`ada.support` ->
  `ada.cx`) passed that guard and had to be pulled out by hand.
- **The pricing page is a hosting platform's own price list.** A repo at
  `github.com/org/thing` and GitHub's plans at `github.com/pricing` share a host,
  so the owner check cannot object. `isPlatformPricing` in `host-mismatch.js`
  refuses a list of platforms that host other people's projects - GitLab, npm,
  PyPI, HuggingFace, `vercel.app` and the rest. This hit `bumblebee` and
  `markitdown`, whose `pricing_url` was null, so nothing was compared at all.

## Two ways a price is wrong while looking right

Both of these produce a confident, plausible, wrong number. Neither looks like a
failure, so nothing catches them except knowing they exist.

**1. The page opens on annual billing.** Six listings in one week had their
monthly rate read off a panel that was showing the yearly one: Pictory (`$25`
displayed, `$29` actual), TapVid, VayoVideo (`$19.99` against `$39.90`),
TextileGen (`$15.90` against `$29.90`), LTX Studio and Canva. It is now the
common case rather than the exception.

> **Toggle to Monthly before reading any figure, and say in `notes` which
> billing period you read.** Then toggle to Annual and record that figure too, in
> `annual_usd` and `annual_total_usd` - both periods are on the page and the row can
> hold both. Only when a page has no monthly option at all is that
> `billing_basis: annual_equivalent`.

**2. The page geo-prices to wherever the crawler is.** The sweep runs from a
machine in Serbia, so a vendor that localises serves RSD, and the number that
comes back is accurate and in the wrong currency. Canva's tiers are keyed
`monthly_rsd` for exactly this reason. Seven rows already hold a non-USD figure
in the USD columns - `hunter`, `contaro`, `deepl`, `dall-e-3`, `wispr-flow` and
`zoom-ai-companion` in EUR, `gemini` in RSD - and they render as dollars on the
tool page, `/compare` and `/pricing`.

> **`price_from` and `price_to` are dollars.** A non-zero price with a currency
> other than USD is now refused by `apply-pricing-extraction.js` as
> `non_usd_price` and never written. Record the real figure in `pricing_info`
> with its currency and the date, and leave the columns NULL. Do not convert:
> a rate drifts and nobody can check it later.

The capture now forces `en-US`, `America/New_York` and a matching `Accept-Language`
header (`scripts/capture-pricing-pages.js`), which stops the currency half at source for
vendors that key off any of those. It cannot help one that geolocates by IP, so the
`non_usd_price` refusal in `apply-pricing-extraction.js` is still the backstop, and the
seven rows already holding a non-USD figure in the USD columns still need fixing by
hand.

Also changed: `NULL` and `0` no longer compare equal in the price columns.
`Number(null)` is 0, so a tool becoming free read as `price already correct` and
never appeared under "price changes". The row was written either way; the dry-run
summary was understating its own write, which matters because the summary is what
an approval is given against.

# Testing & Verification Checklist

The machine-checkable part of this is `npm run verify` - lint, typecheck, packs
and build in one command, and the hand-back rule. See "Definition of Done" above;
it is the authority, and this list does not repeat it. There is no
`npm run type-check` script - the typecheck is `npm run typecheck`, and the gate
runs it for you.

What the gate cannot see, and you therefore check by hand before marking a task
complete:

- [ ] Run it locally (`npm run dev`, port 3100) and use the page you changed
- [ ] Responsive behaviour at mobile widths
- [ ] Browser console clean - the gate never opens a browser
- [ ] Links resolve, including any new internal route
- [ ] Real rows render. The gate builds against an unreachable Supabase address
      and prerenders zero tool and category pages, so anything that breaks only
      on real data passes it
- [ ] Analytics events fire, for a change that touches tracking
- [ ] Meta description and title as they appear in a search preview

Say which of these you actually did when you hand the change back. "Verified by
hand" without saying what was checked is not a hand-back.

---

# What NOT to Do

❌ Don't use `any` type
❌ Don't use inline styles (use Tailwind classes)
❌ Don't commit sensitive data (API keys, credentials)
❌ Don't skip error handling on async operations
❌ Don't make direct database changes without review
❌ Don't use client-side only features in Server Components
❌ Don't deploy without testing locally first
❌ Don't write overly clever code - clarity > cleverness
❌ Don't assume - ask if uncertain about business logic
❌ Don't run SQL scripts directly on live database
❌ Don't guess about tool information - verify everything

---

# Performance Best Practices

- Use Next.js Image optimization for all images
- Implement proper caching strategies
- Lazy load components when appropriate
- Minimize client-side JavaScript
- Use streaming for slow data fetching
- Optimize database queries (proper indexes)

---

# Deployment

- Main branch deploys to production automatically via Vercel
- Always test in preview deployment first
- Check Vercel logs if issues arise
- Monitor analytics after major changes

---

# Environment Variables

`.env.example` is the checked-in list. Required in `.env.local`:

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
NEXT_PUBLIC_SITE_URL=
ADMIN_API_KEY=
```

Feature-scoped, needed only by the surface that reads them:

```
NEXT_PUBLIC_TURNSTILE_SITE_KEY=   # public submit form
TURNSTILE_SECRET_KEY=             # public submit form
RESEND_API_KEY=                   # transactional + newsletter email
ANTHROPIC_API_KEY=                # enrichment and content scripts
NEXT_PUBLIC_PAYPAL_CLIENT_ID=     # featured listings
PAYPAL_PLAN_MONTHLY=              # featured listings
PAYPAL_PLAN_YEARLY=               # featured listings
INDEXNOW_KEY=                     # POST /api/indexnow
REVALIDATION_SECRET=              # on-demand revalidation
```

There is no `PLAUSIBLE_DOMAIN`. Analytics is GA4, and the measurement ID is a
constant in `components/GoogleAnalytics.tsx`, not an environment variable -
`NEXT_PUBLIC_GA_MEASUREMENT_ID` appears in `.env.example` but nothing reads it.

`npm run verify` needs none of these: it supplies dummy values itself, so the
gate runs on a fresh clone with nothing but `npm ci`.

**Never commit `.env.local` to git.**

## ADMIN_API_KEY - the gate on service-role endpoints

Generate with `openssl rand -hex 32`. It guards everything that writes with the
service role key, because those writes bypass RLS and the route handler is the
only thing left between the table and the internet:

| Surface | How the secret is presented |
|---|---|
| `/admin` and everything under it | HTTP Basic, any username, this as the password |
| `POST /api/submissions/approve` | `Authorization: Bearer $ADMIN_API_KEY` |
| `POST /api/submissions/reject` | `Authorization: Bearer $ADMIN_API_KEY` |
| `POST /api/indexnow` | `Authorization: Bearer $ADMIN_API_KEY` |

The gate fails closed. With the variable unset, those surfaces return 503, so a
deployment that forgets it is locked rather than open. Set it in Vercel for
every environment, not just production.

`/api/upload` and `/api/submit-tool` are deliberately **not** on this list. They
are what the public submit form posts to, so they are gated by Turnstile and
content validation instead - never add admin auth to them or tool submission
stops working for everyone.

---

## Featured Listing Subscriptions (PayPal)

The `/submit` page offers three tiers: Free (reviewed within 7 days), Featured ($9/month or $89/year via PayPal: homepage carousel, priority category placement, verified badge, newsletter, monthly report), and Sponsored Review ($99 one-time: a request form, saved as `listing_type: review` / `status: review_request` and emailed; the owner replies with a payment link). Repriced 2026-10-08 from $19/$189.

### PayPal Integration

Featured listings use PayPal Subscriptions for payment. The flow:

1. User fills in tool name, website URL, and email on `/submit` (Featured tab)
2. User selects Monthly ($9) or Yearly ($89) plan
3. User clicks PayPal button → PayPal handles payment
4. On successful payment, `handleFeaturedPaymentSuccess` calls `/api/submit-tool`
5. API stores the submission in `tool_submissions` with `listing_type: 'featured'`, `status: 'paid'`, and `paypal_subscription_id`
6. Admin and customer confirmation emails are sent via Resend
7. Admin reviews and approves like any other submission, but with 24h priority

### Environment Variables

```bash
NEXT_PUBLIC_PAYPAL_CLIENT_ID=<PayPal client ID>
NEXT_PUBLIC_PAYPAL_PLAN_MONTHLY=<PayPal plan ID for $9/month>
NEXT_PUBLIC_PAYPAL_PLAN_YEARLY=<PayPal plan ID for $89/year>
```

These are in `.env.example`. The PayPal plans must be created in the PayPal Business dashboard first.

### Database: `tool_submissions` table

Featured listings use the same `tool_submissions` table as free submissions, with additional columns:

```sql
- listing_type (string) - 'free' or 'featured'
- billing_plan (string) - 'monthly' or 'yearly'
- paypal_subscription_id (string) - PayPal subscription ID
- status (string) - 'paid' for featured (vs 'pending' for free)
```

Migration: `supabase/migrations/add_tool_submissions_paypal.sql`

### Key Files

| File | Purpose |
|------|---------|
| `components/PayPalSubscribeButton.tsx` | PayPal SDK wrapper with plan selector |
| `app/submit/SubmitToolClient.tsx` | Submit page with Featured form |
| `app/api/submit-tool/route.ts` | Unified API for free and featured submissions |

### Admin Review

Featured submissions appear in the same `/admin/submissions` queue as free ones, but with `status: 'paid'` and `listing_type: 'featured'`. The admin email has a 💰 prefix and green "Payment received" banner.

### Future: Webhook for Subscription Events

To handle cancellations and renewals, add `/api/paypal/webhook` endpoint that:
- Verifies PayPal webhook signature
- Updates tool's `featured` status when subscription cancels
- Sends alerts when subscriptions lapse

---

# When in Doubt

1. **Ask first** - especially for database operations
2. **Show me the plan** - explain what you're about to do
3. **Test locally** - always verify before production
4. **Check twice** - especially for destructive operations
5. **Document changes** - update relevant comments/docs

---

# Content Guidelines

This is a developer's blog with honest reviews:
- Write in first person, authentic voice
- No marketing fluff or promotional language
- Include real examples from production use
- Admit limitations and problems honestly
- Focus on workflow fit over benchmark scores
- Target audience: experienced developers

---

# Communication Standards

When working on this project:
- Be concise but clear
- Show code/SQL before executing destructive operations
- Explain trade-offs when multiple approaches exist
- Flag potential issues proactively
- Ask clarifying questions rather than assuming
- Summarize what you did after completing tasks

---

**Remember**: This is a production site with real traffic. Measure twice, cut once. When in doubt about database operations, ALWAYS ask first.
