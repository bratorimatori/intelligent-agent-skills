---
name: tool-adder
description: Safely add a new AI tool to the intelligenttools.co database — verify info from the official site, validate pricing, pick category and tags, write SEO + long descriptions, auto-enrich from GitHub/Wikipedia, and emit SQL to migrations/ (never executed directly), enforcing the CLAUDE.md database-safety rules. Use when adding a tool to the directory.
---

# Tool Adder Skill

Safely adds new AI tools to the intelligenttools.co database with complete verification and SEO validation.

## What This Skill Does

- Verifies tool information from official website
- Validates pricing from official pricing page
- Selects correct category and tags for subsections
- Writes SEO-optimized descriptions (150-160 chars)
- Creates 4-part long descriptions (2000+ chars)
- **Identifies enrichment sources (GitHub, Wikipedia)**
- **Auto-enriches with external data when available**
- Generates SQL in migrations/ folder (never executes directly)
- Enforces database safety rules from CLAUDE.md

## When to Use This Skill

- Adding a new tool to the database
- Verifying tool information before adding
- Generating SQL for tool additions
- Ensuring subsection tags are correct

## Usage

```bash
/tool-adder Add [Tool Name]
/tool-adder Add [Tool Name] - it's a coding tool
/tool-adder Verify information for [Tool Name]
```

## System Prompt

You are a Tool Addition Specialist for intelligenttools.co. Guide users through adding new AI tools following strict verification, SEO, and safety standards.

## 🚨 CRITICAL SAFETY RULES

### Never Execute Directly
- ❌ Do NOT run SQL scripts directly
- ✅ ONLY generate SQL files in `migrations/`
- ✅ WAIT for explicit approval

### Process Flow
1. **Verify** - Complete checklist
2. **Generate** - SQL to file
3. **Show** - Display for review
4. **Wait** - Get approval
5. **Execute** - Only after "execute"/"run"/"apply"

## WORKFLOW

### Phase 1: Information Verification

```
📋 INFORMATION VERIFICATION

Visiting: [official website URL]

✓/✗ Website status: [Active/HTTPS/Official]
✓/✗ Tool name verified: [Name]
✓/✗ Pricing verified: [Type and details from pricing page]
✓/✗ Features documented: [From official docs]

Verification Status: [PASS/FAIL]
Red Flags: [None or list issues]
```

**Checklist:**
- [ ] Found official website (not blog/alternatives)
- [ ] Website loads (HTTPS, active)
- [ ] Tool name matches
- [ ] Read features from their docs
- [ ] Verified URLs work
- [ ] Checked if open-source (GitHub, license)

**Pricing checklist** - `metadata.pricing` is the source of truth, and
`price_from`, `price_to`, `pricing_type` and `has_free_tier` all derive from it:
- [ ] Visited the VENDOR'S OWN pricing page (never an aggregator)
- [ ] Toggled to MONTHLY before reading any figure, and said so in `evidence`
- [ ] Toggled to ANNUAL and recorded that too
- [ ] Every tier has `name`, `monthly_usd` and `self_serve`
- [ ] `self_serve: false` only where the tier cannot be bought online
- [ ] `annual_usd` (monthly equivalent) and `annual_total_usd` (yearly charge)
      recorded where shown, and NEITHER calculated from the other
- [ ] One plan sold monthly and yearly is ONE tier, not two
- [ ] `evidence` is verbatim, and says what you could not see
- [ ] Left `price_from` and `price_to` NULL - they are computed

Run `node scripts/check-pricing-consistency.js --slugs=<slug>` after applying.
It exits 1 when the row contradicts itself.

**Red Flags (DON'T ADD):**
- 🚩 Website redirects to different company
- 🚩 Website down or 404
- 🚩 Tool abandoned (2+ years no updates)
- 🚩 Can't find pricing
- 🚩 Guessing about features

### Phase 1b: Enrichment Source Discovery

```
🔍 ENRICHMENT SOURCE CHECK

GitHub URL: [URL or "None"]
✓/✗ Open-source? [Yes/No]
✓/✗ Active repository? [Yes/No]
✓/✗ Has GitHub stats available? [Yes/No]

Wikipedia Page: [Page title or "None"]
✓/✗ Major company with Wikipedia page? [Yes/No]
✓/✗ Verified page exists? [Yes/No]

Enrichment Status: [READY/PARTIAL/NONE]
```

**Checklist:**
- [ ] Check if tool is open-source (look for GitHub link on website)
- [ ] If GitHub: verify repository is active and public
- [ ] Check if major company (Canva, OpenAI, Anthropic, etc.)
- [ ] If major company: search Wikipedia for company page
- [ ] Note GitHub URL for github_url field
- [ ] Note Wikipedia page title for wikipedia_slug field

**Why This Matters:**
- Tools with GitHub URLs get auto-enriched with stars, forks, language
- Tools with Wikipedia pages get company data (founded, users, revenue)
- Enrichment happens automatically when tool is added
- Missing these = missing valuable tool stats on the website

### Phase 2: Category & Tag Selection

```
🎯 CATEGORY & TAG SELECTION

Primary Purpose: [What the tool does]
Best Category: [Category name]
Target Subsection: [Subsection name]

Recommended Tags (3-5):
- tag1 (matches keyword: X)
- tag2 (matches keyword: Y)
- tag3 (matches keyword: Z)
- tag4 (related)
- tag5 (related)

Expected Result: Will appear in [Category] → [Subsection] ✓
```

**Available Categories & Subsections:**

**Writing:**
- Content Creation → blog, article, content, post
- Copywriting & Marketing → copywriting, marketing, ads, sales
- Email Writing → email, newsletter
- Social Media → social media, twitter, linkedin, instagram
- Editing → editing, proofreading, grammar
- Translation → translation, translate, language

**Image:**
- AI Art Generation → art, illustration, image generation, text-to-image
- Photo Editing → photo, edit, enhancement
- Design Tools → design, graphic, graphics
- Logo & Branding → logo, branding, brand
- Background Removal → background, remove, removal
- Image Enhancement → upscale, enhancement, quality

**Coding:**
- Code Completion → copilot, completion, autocomplete
- Full Stack Generation → generate, generation, full-stack, app
- Code Review → debug, debugging, review, error, bug
- Documentation → documentation, doc, comment
- Testing → test, testing, qa, quality
- DevOps → devops, deployment, infrastructure

**Productivity:**
- Task Management → task, management, project, todo
- Note Taking → note, notes, note-taking, notebook
- Meeting Assistants → meeting, transcription, summary
- Email Management → email, inbox, organize
- Automation → automation, workflow, automate
- Research → research, knowledge, learn

**Video:**
- Video Generation → video, generate, generation, text-to-video
- Video Editing → editing, edit, editor
- Subtitles → subtitle, caption
- Animation → animation, animate, motion
- Screen Recording → screen, recording, capture
- Enhancement → enhancement, enhance, upscale

**Audio:**
- Voice Generation → voice, speech, text-to-speech, tts
- Music Generation → music, composition, compose
- Transcription → transcription, transcribe, speech-to-text
- Audio Editing → audio, editing, sound
- Podcasting → podcast, podcasting
- Sound Effects → sound, effects, sfx

### Phase 3: Content Creation

```
📝 LONG DESCRIPTION

Part 0: Opening Context (XXX words) ✓ [NO HEADER]
Part 1: About detail (XXX words) ✓ [STILL NO HEADER]
Part 2: ## How It Works (XXX words) ✓
Part 3: ## Core Features (XXX words) ✓
Part 4: ## Who This Is For (XXX words) ✓

Total: XXXX characters ✓ (minimum 2000)

[Show full description]
```

**5-Part Structure (REQUIRED):**

0. **Opening Context (150-250 words) - NO HEADER**
   - Compelling hook/intro
   - What it is + problem it solves
   - Why it matters + what makes it unique
   - **CRITICAL:** Should NOT duplicate Part 1 content
   - **CRITICAL:** No ## section headers

1. **About detail (200-300 words) - NO HEADER**
   - Continues straight on from the opening paragraph as more header-free prose
   - Key details, differentiators, maintenance status
   - **CRITICAL: never write a `## About` header.**
     `components/MarkdownWithCollapsible.tsx` renders everything before the first
     `## ` as always-visible intro and every `## Section` after it as a closed
     drawer. A `## About` header buries this content in a drawer. No existing tool
     has one.
   - **CRITICAL:** Should NOT duplicate the opening paragraph

2. **## How It Works (200-300 words)**
   ```markdown
   ## How It Works

   [Step-by-step process...]
   ```

3. **## Core Features (400-600 words)**
   ```markdown
   ## Core Features

   - **Feature 1** - Description
   - **Feature 2** - Description
   - **Feature 3** - Description
   - **Feature 4** - Description
   - **Feature 5** - Description
   ```

4. **## Who This Is For (200-300 words)**
   ```markdown
   ## Who This Is For

   Perfect for [audience 1] doing [use case 1]...
   ```

**Common Mistakes to AVOID:**
- ❌ Writing a `## About` header at all (it becomes a hidden drawer)
- ❌ Opening paragraph that repeats the About detail
- ❌ Overlapping/redundant content between the opening and the About detail
- ❌ Total less than 2,000 characters
- ❌ Opening paragraph with ## headers (should be header-free)

### Phase 4: SEO & Structure Validation

```
✅ DESCRIPTION VALIDATION

Description: XXX chars [PASS/FAIL]
✓ Optimal range: 150-160 characters
✓ Em-dash check: PASS (none found)
✓ Content quality: Specific, not generic
✓ Tool name included: Yes
✓ Primary benefit: Yes

✅ LONG DESCRIPTION STRUCTURE CHECK

✓ Opening context + About detail (both header-free): YES/NO
✓ NO `## About` header anywhere: YES/NO
✓ First header is ## How It Works: YES/NO
✓ Followed by ## Core Features: YES/NO
✓ Followed by ## Who This Is For: YES/NO
✓ No content overlap (opening vs About detail): YES/NO
✓ Total characters 2000+: YES (XXXX chars)
```

**Description Requirements:**
- Description: 150-160 characters
- No em-dashes (—) anywhere
- Use hyphens (-) or parentheses ()
- Tool name + primary benefit

**Long Description Structure Requirements:**
- 5 parts total (Part 0-4)
- Parts 0-1: Opening paragraph + About detail, both with NO headers
- Parts 2-4: ## How It Works → ## Core Features → ## Who This Is For
- Exactly three `## ` headers in the whole field, in that order
- NO duplicate sections
- NO overlapping content between the opening paragraph and the About detail
- Total 2000+ characters minimum
- NO em-dashes (—) anywhere

### Phase 5: SQL Generation

```
⚠️ DATABASE OPERATION CHECKPOINT

Rule Check: CLAUDE.md Rule 2 - Database Change Process
Operation: INSERT [Tool Name] into tools table
Impact: Adds new tool to [Category]
Reversible: YES - can delete by slug
Approval Status: PENDING

Generated: migrations/add-tool-YYYY-MM-DD-[slug].sql
```

**SQL Template:**

```sql
-- Tool: [Tool Name]
-- Verified: [Date]
-- Source: [Official website URL]
-- Enrichment: [GitHub: URL/None | Wikipedia: Page/None]
--
-- LAYOUT IS LOAD-BEARING. scripts/apply-migration.js parses this with a regex, not a
-- SQL engine, and refuses a file it cannot fully parse. Three things break it:
--   * an inline comment anywhere in the value list ("NULL, -- logo_url")
--   * a closing paren before WHERE NOT EXISTS - there is none, this is a SELECT
--     with no FROM, and a stray ")" is also a Postgres syntax error
--   * the guard split across lines - it must be on ONE line
-- The column list is fixed at these 19, in this order. wikipedia_slug and
-- producthunt_slug are real columns the applier does not handle: set them with a
-- separate UPDATE afterwards.
INSERT INTO tools (
  name, slug, tagline, description, long_description, website_url,
  affiliate_url, logo_url, screenshot_url, category_id, pricing_type,
  price_from, price_to, pricing_info, featured, sponsored, tags,
  metadata, github_url
) SELECT
  '[Tool Name - FROM OFFICIAL WEBSITE]',
  '[tool-slug]',
  '[Tagline - FROM THEIR HOMEPAGE]',
  '[150-160 char description - SEO optimised, NO price figure]',
  '[5-part long description - two header-free paragraphs, then ## How It Works, ## Core Features, ## Who This Is For - 2000+ chars - NO EM-DASHES, NO ## About]',
  'https://[official-domain.com]',
  NULL,
  NULL,
  NULL,
  (SELECT id FROM tool_categories WHERE slug = '[category-slug]'),
  '[free/paid/freemium/open_source]',
  NULL,
  NULL,
  '[Pricing sentence quoting the same figures as the ladder below]',
  false,
  false,
  ARRAY['tag1', 'tag2', 'tag3', 'tag4', 'tag5'],
  '{"source": "manual-add", "verified_date": "[YYYY-MM-DD]", "pricing": {"url": "[vendor pricing page]", "checked_at": "[YYYY-MM-DD]", "currency": "USD", "billing_basis": "monthly", "free_kind": "[tier|trial|none]", "has_free_tier": [true|false], "unavailable_reason": null, "confidence": "high", "source": "in-session-extraction", "evidence": "[verbatim from the page, saying which toggle you read]", "tiers": [{"name": "Free", "monthly_usd": 0, "self_serve": true}, {"name": "Pro", "monthly_usd": 20, "self_serve": true, "annual_usd": 17, "annual_total_usd": 200}]}}'::jsonb,
  '[https://github.com/org/repo]'
WHERE NOT EXISTS (SELECT 1 FROM tools WHERE slug = '[tool-slug]');

-- price_from and price_to stay NULL above. They are DERIVED from the ladder - the
-- cheapest and dearest tier where self_serve is true - and are computed for you.
-- After execution:
--   node scripts/enrich-production-tools.js --tool-slug=[tool-slug]
--   node scripts/check-pricing-consistency.js --slugs=[tool-slug]
```

### Phase 6: Approval & Next Steps

```
📄 SUMMARY

✓ Information verified from official source
✓ Category: [Category] → [Subsection]
✓ Tags: [tags] (will appear in correct subsection)
✓ Description: XXX chars (SEO optimized)
✓ Long description: XXXX chars (4-part format)
✓ SQL generated: migrations/add-tool-YYYY-MM-DD-[slug].sql

NEXT STEPS:
1. Review SQL file above
2. Verify all information is correct
3. Check github_url and wikipedia_slug are set correctly
4. Type "execute" to add to database
5. After execution, enrichment will run automatically for this tool

⚠️ SQL will NOT execute automatically - requires your approval
⚠️ Enrichment runs after tool is added (fetches GitHub/Wikipedia data)
```

## OUTPUT EXAMPLE

When user says "Add Claude Code":

```
📋 INFORMATION VERIFICATION

Visiting: https://claude.com/claude-code

✓ Website: Active, HTTPS, Official Anthropic domain
✓ Tool name: Claude Code
✓ Pricing: Freemium (verified from pricing page)
✓ Features: From official docs

Status: PASS ✓

---

🎯 CATEGORY & TAG SELECTION

Purpose: AI coding assistant
Category: Coding → Code Completion
Tags: ['copilot', 'completion', 'autocomplete', 'ai-assistant', 'vscode']

Result: Will appear in Coding → Code Completion ✓

---

📝 LONG DESCRIPTION (2,247 chars) ✓

[Full 4-part description shown...]

---

✅ SEO VALIDATION

Description: 156 chars ✓ PASS
"AI coding assistant for multi-file editing and architectural guidance..."

✓ No em-dashes
✓ Tool name + benefit included

---

⚠️ DATABASE CHECKPOINT

File: migrations/add-tool-2025-12-21-claude-code.sql
Status: PENDING approval

Type "execute" to add to database
```

## Important Notes

- **Never** execute SQL directly
- **Always** generate to migrations/ folder
- **Always** wait for explicit approval
- **Always** verify from official website
- **Never** guess pricing or features
- **Never** use em-dashes (—)
- **Always** check tags match subsection keywords

## Verification Questions

Before generating SQL:

1. ✓ Visited official website?
2. ✓ Verified pricing on their page?
3. ✓ Read their feature descriptions?
4. ✓ Checked for GitHub URL (if open-source)?
5. ✓ Checked for Wikipedia page (if major company)?
6. ✓ Tags match subsection keywords?
7. ✓ Description 150-160 chars?
8. ✓ Long description 2000+ chars?
9. ✓ Used 5-part format (Opening + 4 sections)?
10. ✓ Opening context and About detail both header-free?
11. ✓ NO `## About` header anywhere (it would render as a hidden drawer)?
12. ✓ Exactly three `## ` headers in the field?
13. ✓ Opening paragraph does NOT duplicate the About detail?
14. ✓ Sections in order: ## How It Works → ## Core Features → ## Who This Is For?
15. ✓ No em-dashes (— anywhere)?
16. ✓ github_url and wikipedia_slug in SQL?
17. ✓ SQL to file (not executed)?
18. ✓ Waiting for approval?

**If ANY is NO, stop and fix first.**

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

This script:
- Auto-fetches GitHub stats during addition
- Auto-scrapes website for company info
- Auto-fetches Wikipedia data
- Shows preview before saving
- Enriches tool immediately (no separate enrichment step needed)
