# Text Proofreader Skill

Proofreads the prose that ships to readers: blog post markdown and tool descriptions in the database.

## What This Skill Does

- Finds grammar, spelling and punctuation errors
- Catches the mistakes spellcheck misses: wrong word ("its self" for "itself"), doubled words, subject/verb disagreement, broken sentences left behind by an edit
- Flags sentences that are hard to parse, and says why
- Enforces the project text rules that nothing else checks on tool descriptions
- Reports findings with an exact location and a suggested replacement
- Never edits the database directly. Fixes to tool text are written as a migration SQL file for approval

## What This Skill Does NOT Do

- It does not run the SEO checks. Title length, excerpt length, internal links and heading hierarchy belong to `seo-blog-validator`. Run that too before publishing a post; this skill will not repeat its verdicts.
- It does not rewrite for style. The site's voice is deliberately first person and informal, and that is not an error to be corrected.
- It does not touch quoted material. Vendor copy quoted in a review keeps its own typos, because it is a quote.

## When to Use This Skill

- After drafting or editing a blog post, before publishing
- After writing a tool's `long_description`, before the INSERT is approved
- When sweeping existing tool descriptions for quality
- When a reader reports a typo and you want to check whether the same mistake appears elsewhere

## Usage

```
Proofread content/blog/my-new-post.md
Proofread the long_description for tool slug "maxdesk"
Proofread all tool descriptions added since 2026-08-01
```

## How To Run It

### Blog posts

Read the markdown file directly. Check the body prose and the `excerpt` in the frontmatter. Skip fenced code blocks, URLs, and frontmatter keys.

### Tool descriptions

Query with the anon key, which is read-only and sufficient:

```js
const { data } = await supabase
  .from('tools')
  .select('slug, name, tagline, description, long_description')
  .eq('slug', SLUG);            // or .gte('created_at', DATE) for a sweep
```

For a sweep, work in batches and report per tool. Do not hold 410 descriptions in context at once.

## What To Check

### Grammar and spelling

- Misspellings, including ones that are valid words in the wrong place ("its self", "there" for "their", "affect" for "effect")
- Doubled words, the most common survivor of an edit. Match these **within a single line** (`\b(\w+)[ \t]+\1\b`), not with `\s`, which spans newlines: a `## Who This Is For` header followed by a paragraph opening "For performance marketers..." looks like a doubled "For" and is not. That mistake produced 138 false positives out of 139 hits on a real sweep
- Subject/verb and singular/plural disagreement
- Sentence fragments and run-ons created by a half-finished revision
- Punctuation: missing commas that change meaning, stray apostrophes, unbalanced brackets or quotes

### Project text rules

These come from CLAUDE.md and apply to anything stored in the database:

- **No em-dashes (—) or en-dashes (–).** Use a hyphen or parentheses. This is a hard rule: em-dashes cause encoding problems downstream
- `description` reads as a complete sentence and stays at or under 160 characters
- `long_description` is at least 2000 characters
- `long_description` has no `## About` header. Everything before the first `## ` renders as always-visible intro; an `## About` header buries it in a closed drawer
- Section headers are exactly `## How It Works`, `## Core Features`, `## Who This Is For`
- `## Core Features` has 4 to 6 items, each with a **bold name**

### Claims

Flag any sentence that asserts something the tool's own site does not say: invented pricing, invented user counts, features that were not verified. Overstating a discount or a capability is worse than a typo, because a reader acts on it.

## Known False Positives

Verified against a full sweep of the catalogue. Do not report these:

- **Doubled words across a paragraph break.** See the matching rule above. Nearly every naive hit is a section header whose last word repeats at the start of the next paragraph
- **Repeated "that", "had", "the"** inside a single line are occasionally correct English ("the fact that that model...")
- **Technical strings and brand names** flagged as misspellings: `Namelix`, `Seedream`, `Kling`, `ghostwriter`
- **Sentence fragments in blog posts** used deliberately for rhythm. House style, not an error

## Output Format

Report findings grouped by severity, most severe first. Every finding needs a location, the text as written, and the replacement.

```
PROOFREAD: content/blog/my-new-post.md

ERRORS (3)
  L42  "turns its self into a ticket"
       -> "turns itself into a ticket"
  L88  "the the same result"
       -> "the same result"
  L116 "Neither of the tools are free"
       -> "Neither of the tools is free"

PROJECT RULES (1)
  L203 em-dash found: "fast — and cheap"
       -> "fast - and cheap"

CLARITY (1)
  L57  34-word sentence with three subordinate clauses; the subject changes
       halfway through. Suggest splitting after "...in production."

CLEAN: frontmatter, code blocks skipped (4 fenced blocks)
```

If nothing is found, say so plainly and state what was checked. A clean pass is a real result.

## Rules

1. **Report first, edit only when asked.** Show findings and wait. Do not silently rewrite a draft.
2. **Never write to the database.** Corrections to tool text go into `migrations/draft-YYYY-MM-DD-text-fixes.sql` as UPDATE statements with a rollback comment, following the process in CLAUDE.md. The user runs it.
3. **Quote the original exactly.** A finding the user cannot locate is useless.
4. **Do not flag deliberate voice.** Contractions, sentence fragments used for rhythm, first person, and blunt opinions are the house style.
5. **Do not invent a correction you are unsure of.** If a sentence is ambiguous rather than wrong, put it under CLARITY and explain the ambiguity instead of guessing at intent.
6. **Say when something is a judgement call**, so the user can overrule it quickly.

## Related Skills

- `seo-blog-validator` - title, excerpt, links, heading hierarchy. Run before publishing a post
- `tool-adder` - verifies a new tool's facts and pricing before it is added
- `blog-post-creator` - drafts a new post in the house format
