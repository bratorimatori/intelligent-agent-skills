# Tables

<!--
  Sanitized: exact per-table row counts and submission-status breakdowns were
  dropping snapshots that drift. The anon-visibility column is the durable,
  non-guessable part and is kept. Measure counts live.
-->

Anon visibility is the part worth trusting, and the part you cannot guess. Row counts
drift — measure them with a probe or a script, never from a file.

## A warning about `supabase/migrations/`

Those files are a historical record, not a description of the live database. They will
mislead you about RLS specifically. For example, a migration can create a
`FOR SELECT USING (true)` policy on a table that the anon key nevertheless reads **0
rows** from today, because a later change tightened it and the file was never updated.

Do not infer current access from a migration file. Measure it, or use a script that
already holds the right key.

## Inventory

| Table | Anon | What it is |
|---|---|---|
| `tools` | full | The directory. The main table. |
| `tool_submissions` | **0 rows, no error** | Public submission queue. |
| `tool_categories` | full | Categories. Subsections are code, in `lib/category-subsections.ts`, not rows. |
| `deals` | full | Deals / marketplace listings. |
| `newsletter_subscribers` | **error** | Email addresses. Locked deliberately, and loudly. Service role only. |
| `blog_posts` | full | Mostly unused. Posts are markdown in `content/blog/`. |
| `tool_click_events` | **0 rows** | Affiliate click tracking. |
| `recommendation_logs` | **0 rows** | `/recommend` inputs and outputs. |
| `game_scores` | **0 rows** | Scores from `games-src/`. |
| `featured_products` | **0 rows** | Paid placements. |
| `analytics_events` | 0 rows | Empty under both keys at last check. |
| `tool_images` | unverified | Referenced by `set-tool-gallery.js`. Probe inconclusive. |
| `cron_logs` | unverified | Same. |

Every table marked **0 rows** is the silent-zero trap: the anon key reports an empty
table rather than refusing you.

## `tools`

The columns an `INSERT` writes, in the order `apply-migration.js` parses. It only
accepts this exact shape (see `CLAUDE.md` for the authoritative column list):

    name, slug, tagline, description, long_description, website_url,
    affiliate_url, logo_url, screenshot_url, category_id, pricing_type,
    price_from, price_to, pricing_info, featured, sponsored, tags,
    metadata, github_url

Plus, managed by the database or by scripts rather than by you: `id`, `upvotes`,
`view_count`, `click_count`, `created_at`, `updated_at`.

Notes that matter:

- `slug` is unique, and it is the identity of a tool everywhere: the URL
  `/tools/<slug>`, every script's `--tool-slug`, and the `WHERE NOT EXISTS` guard.
- `pricing_type` is constrained to `free`, `freemium`, `paid`, `open_source`.
- `price_from` and `price_to` are **derived** from `metadata.pricing`, computed by
  `apply-pricing-extraction.js` from the tiers where `self_serve` is true. Do not
  write them by hand.
- `metadata` is `jsonb` and holds the pricing ladder at `metadata.pricing`. Required
  on every row. `CLAUDE.md` is the full field reference.
- `tags` is `text[]`. Tags are what route a tool into a category subsection, so a tag
  that matches no subsection keyword makes the tool invisible on its category page.
  Check with `check-subsection-routing.js` before writing, not after.
- No em-dashes in any text column.

## `tool_submissions`

    id, name, website_url, tagline, description, category_id, pricing_type,
    price_from, price_to, submitter_email, status, admin_notes, reviewed_by,
    reviewed_at, created_at, updated_at

- `status` is `pending`, `approved` or `rejected`.
- `submitter_email` lives here and **only** here. It is not on `tools`. Every outreach
  campaign joins back to this table for the address.
- Approving a submission does not publish a tool. The two are separate writes.

## `tool_categories`

    id, name, slug, description, icon, created_at

Look up the id inline rather than pasting a UUID:

    (SELECT id FROM tool_categories WHERE slug = 'category-slug')
