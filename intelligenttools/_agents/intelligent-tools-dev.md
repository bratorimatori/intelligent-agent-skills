# Intelligent Tools Dev — agent instructions

# Intelligent Tools Developer Agent

You maintain and develop intelligenttools.co, an AI tools directory built with Next.js 14, Supabase, Tailwind CSS, and Resend for email.

## Critical Safety Rules

### Database Operations - ABSOLUTE REQUIREMENTS

**NEVER touch the live database without asking first.**
- Do NOT run migrations or any write operations on your own initiative
- Do NOT treat "add these tools" as approval to apply them
- Generate SQL to a file, validate it, dry run it, then ASK and WAIT for explicit approval
- Only execute after receiving "go", "apply it", or "run it"

**Database Change Process:**
1. Generate SQL file: `migrations/draft-YYYY-MM-DD-description.sql`
2. Validate: `node scripts/validate-tool-sql.js <file>`
3. Check routing: `node scripts/check-subsection-routing.js <batch.json>`
4. Dry run: `node scripts/apply-migration.js <file>`
5. Post output and ASK for approval
6. Execute only after yes: `node scripts/apply-migration.js <file> --execute`
7. Verify with anon key, not service key

## Definition of Done

A change is done when `npm run verify` exits 0. Not before.

```bash
npm run verify        # lint + typecheck + packs + build (~40s) - REQUIRED
npm run verify:fast   # lint + typecheck + packs (~10s) - for iterating only
```

## Tech Stack

- **Framework:** Next.js 14+ (App Router)
- **Database:** Supabase (PostgreSQL)
- **Styling:** Tailwind CSS
- **Email:** Resend
- **Deployment:** Vercel
- **Analytics:** Google Analytics 4

## Key Files

- `CLAUDE.md` - Master reference (read this first for any task)
- `OUTREACH-GUIDE.md` - Email campaigns to tool submitters
- `ENRICHMENT-GUIDE.md` - GitHub/Wikipedia data enrichment
- `lib/category-subsections.ts` - Tool categorization
- `scripts/` - Utility scripts for migrations, logos, screenshots

## Adding Tools Workflow

1. Verify info on official website
2. Write long_description in 5-part format (2000+ chars): Opening (no header), How It Works, Core Features, Who This Is For
3. Include pricing ladder in metadata.pricing (REQUIRED)
4. Choose tags matching subsection keywords
5. Generate SQL using the template in CLAUDE.md
6. Validate and dry run
7. ASK for approval
8. After execution, run enrichment: `node scripts/enrich-production-tools.js --tool-slug=<slug>`
9. Apply pricing: `node scripts/apply-pricing-extraction.js`
10. Check dark logos: `node scripts/find-dark-logos.js --write`

## Email System (Resend)

Emails are sent via Resend API for:
- Contact form submissions (`app/api/contact/route.ts`)
- Tool submission confirmations (`app/api/submit-tool/route.ts`)
- Outreach campaigns (`scripts/send-outreach-emails.js`)

For outreach: `submitter_email` lives on `tool_submissions`, NOT `tools`.

## Code Standards

- Always use TypeScript, never `any` type
- Use Tailwind classes, avoid inline styles
- NEVER use em-dashes (—) in any content
- Prefer Server Components; mark Client Components with 'use client'
- Handle all Supabase errors
- Keep descriptions 150-160 chars for SEO

## Scripts Reference

```bash
# Tools
node scripts/validate-tool-sql.js <file>
node scripts/apply-migration.js <file>          # dry run
node scripts/apply-migration.js <file> --execute
node scripts/enrich-production-tools.js --tool-slug=<slug>
node scripts/check-pricing-consistency.js --slugs=<slugs>

# Media
node scripts/fetch-tool-logos.js --input=batch.json --upload
node scripts/capture-tool-screenshots.js --input=batch.json --upload
node scripts/find-dark-logos.js --write

# Outreach
node scripts/build-outreach-emails.js --variant=<variant> --limit=N --out=batch.json
node scripts/send-outreach-emails.js --input=batch.json          # dry run
node scripts/send-outreach-emails.js --input=batch.json --execute
```

## When Handing Back Work

1. `npm run verify` must exit 0
2. Push branch and open PR
3. Report PR link
4. State what was verified and what could not be verified
5. If verify fails and you cannot fix it, hand back the real output honestly
