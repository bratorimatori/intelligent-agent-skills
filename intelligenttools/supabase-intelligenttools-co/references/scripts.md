# Scripts, keys and which ones write

<!-- Sanitized: exact script counts generalized; classification is what matters. -->

`scripts/` holds the project's automation. Classification below is the shape to
expect; verify a given script against its own source, because one-off repair scripts
are dated and some are already spent.

## The convention

    node scripts/<name>.js <args>              # dry run, reads only
    node scripts/<name>.js <args> --execute    # writes

The dry run is not a courtesy. It is the artifact a human approves, so post its real
output and quote it rather than summarising it.

## Safe to run unattended: read-only, anon key

These need only `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`, and
they cannot write:

| Script | Answers |
|---|---|
| `plan-pricing-sweep.js --status` | What is stale and what has no pricing ladder. Ask this instead of working out the queue yourself. |
| `check-pricing-consistency.js --slugs=a,b` | Does a row contradict itself. Exits 1 if so. |
| `check-subsection-routing.js <batch.json>` | Will these tags route to a real subsection. |
| `host-mismatch.js` | Is this still the same product's pricing page. |
| `find-discontinued-tools.js` | Which listings look dead. |
| `check-comparisons.js` | State of the compare pages. |
| `tool-of-the-day.js` | Today's pick. |

`capture-pricing-pages.js` is also anon-only, but it drives a real browser and writes
screenshots plus `.pricing-temp/captures.json` to disk.

## Needs the service role

A large share of scripts read `SUPABASE_SERVICE_ROLE_KEY`. Two different reasons, and
the distinction matters:

**Reads a locked table, writes nothing to the database:**

- `screen-submissions.js` - the submission queue is invisible to anon, so this must
  hold the service role. Use it rather than querying `tool_submissions` yourself.
- `find-dark-logos.js --write` - reads `tools`, writes only the local file
  `lib/dark-logos.ts`. That file is generated and never computed at runtime, so a dark
  logo stays invisible on the site until its slug lands in it.
- `click-report.js` - reads `tool_click_events`.

**Writes to the database, and therefore needs an explicit human go:**

- `apply-migration.js <file> [--execute]` - the tool-batch path. Parses only the exact
  `INSERT ... SELECT ... WHERE NOT EXISTS` shape documented in `CLAUDE.md`.
- `apply-pricing-extraction.js [--execute]` - the pricing path. Computes `price_from`
  and `price_to` from the ladder. `--metadata-only` records the ladder while leaving
  the price columns alone, which is the right answer whenever confidence is low.
- `reject-pending-submissions.js`, `reconcile-submissions.js` - submission status.
- `enrich-production-tools.js --tool-slug=<slug>` - GitHub, website and Wikipedia
  enrichment on an existing row.
- `fetch-tool-logos.js --upload`, `capture-tool-screenshots.js --upload`,
  `set-tool-media.js`, `set-tool-gallery.js`, `upload-blog-images.js --execute` -
  storage uploads, and they return the URLs you then put in the row.
- `send-outreach-emails.js --execute`, `send-newsletter.js` - these send **real email
  to real people**. Treat the approval bar as higher than a database write, because a
  database write can be reverted and a sent email cannot.
- Everything matching `apply-*`, `fix-*`, `backfill-*`, `set-tool-*`, `delete-*`,
  `mark-*`: one-off repair scripts, most of them dated and already spent. Read the
  source before running one. A script written for a specific batch months ago will
  happily run again today against rows it was never meant to touch.

## Do not use

`extract-pricing.js` calls the Anthropic API with a key that has no credit. A Claude
subscription does not fund it. Use the digest path instead:
`pricing-digest.js --count`, then `--batch=N --size=20`, read the pages yourself, then
`merge-pricing-extractions.js`.

## Temp directories are shared per clone

`.pricing-temp/`, `.logos-temp/`, `.screenshots-temp/` and `.screening-temp/` are one
directory per checkout, not one per run. Two sweeps in the same clone overwrite each
other's `captures.json`. If you run long batch work, work in your own clone.

None of these directories is ever committed.
