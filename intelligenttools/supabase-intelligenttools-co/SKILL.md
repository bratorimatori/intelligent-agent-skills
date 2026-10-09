---
name: supabase-intelligenttools-co
description: "Reach the Supabase database behind intelligenttools.co: which key to use (anon vs service role), how to run an ad-hoc read, the tables and their real row-level-security visibility, which scripts read and which write. Use for any task touching the tools directory data, tool submissions, pricing ladders, deals, the newsletter list, or a migration against the live database."
---

<!--
  Sanitized for the shared skills repo. This skill carries no credentials and
  never did — only env-var *names*, which ship in a browser bundle or are
  injected by the runtime. Precise, drifting operational specifics (exact
  per-table row counts, submission-status breakdowns, exact script counts)
  were generalized here: the live source of truth is the app repo's CLAUDE.md
  and a measured probe, not a snapshot in a skill file. Read CLAUDE.md each
  cycle; it wins wherever it disagrees with this file.
-->

# Supabase for intelligenttools.co

How to reach the Supabase database behind intelligenttools.co: which key to use, how
to run a read, and what the platform will silently lie to you about.

Use this whenever a task touches the `tools` directory data, tool submissions, pricing
ladders, deals, the newsletter list, or any migration against the live database.

This file covers **access**. It does not govern **content**: the row format, the
long_description shape, the pricing ladder fields and the tag rules all live in the
app repo's `CLAUDE.md`, and `CLAUDE.md` wins wherever the two appear to disagree. Read
it every cycle rather than from memory, because it changes.

## Rule 0: there is one live database and it is production

There is no staging copy. Every write lands on the site visitors are reading right
now. `CLAUDE.md` CRITICAL SAFETY RULES bind you, and the short version is:

    generate to a file -> validate -> dry run -> post the output and ASK -> execute -> read back with the anon key

You may run the write. You may never decide on your own that now is the moment.
"Go", "apply it" and "run it" are approval. Silence is not. Approval is per batch and
expires with the batch.

`SELECT` is always safe and needs no approval.

## Credentials

Three variables, always under these exact names:

| Variable | What it is |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Project URL. Not secret. |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public key. Ships in the browser bundle, so it is not secret either. RLS applies to it. |
| `SUPABASE_SERVICE_ROLE_KEY` | **Bypasses RLS entirely.** Full read and write on every table. |

Scripts load them with `require('dotenv').config({ path: '.env.local' })` from the
**repo root**, and `.env.local` is not in git. dotenv does not override variables
already present in the environment, so a value injected by the runtime wins and the
file is only a fallback.

**If `.env.local` is absent and the variables are not in your environment, stop and
ask a human to provide them.** Do not improvise, do not reconstruct a key from
another clone, and do not carry on with a read that will return nothing. A fresh
checkout does **not** contain `.env.local`.

Never print, echo, log, `cat` or quote any of these values, and never ask for a key
to be pasted into a comment or a chat. Report the variable *name* that is missing.

Never import the service-role client into a Client Component. In app code the three
clients already exist, so use them instead of building your own:

- `lib/supabase/client.ts` - browser, anon
- `lib/supabase/server.ts` - Server Components and route handlers, anon, cookie aware
- `lib/supabase/admin.ts` - `createAdminClient()`, service role, server only

## Choosing the key

**Read with the anon key. Reach for the service role only when a named script needs
it.**

The anon key is not merely the safe choice, it is the *correct* choice for
verification: it shows you what a visitor actually sees. A row that exists but is
invisible to anon is a bug, and the service role hides that bug from you. After every
write, read the rows back with the anon key, never with the key you wrote them with.

### The trap: RLS returns zero rows, not an error

This is the one thing that will burn you. For most locked tables the anon key does not
get "permission denied". It gets an empty result and no error. A zero count under the
anon key does **not** mean the table is empty.

Locked tables (submission queue, click/recommendation/score logs, featured
placements) read as **empty** under anon while holding real rows under the service
role. One table (`newsletter_subscribers`) is locked *loudly* and errors instead.
Public content (`tools`, `tool_categories`, `deals`, `blog_posts`) reads identically
under both keys — that is the set you can verify as a visitor.

So: **never screen the submission queue with the anon key.** It will report an empty
queue while real submissions sit pending, and "nothing to do today" is the most
expensive wrong answer available to you. Use `scripts/screen-submissions.js`, which
holds the service role for exactly this reason.

Row counts drift; the visibility behavior is the part worth trusting. Measure counts
with a probe or a script, never from memory — see `references/tables.md`.

## Running a read

Ad-hoc reads have to run **from the repo root**, because that is where both
`node_modules` and `.env.local` live. A script in `/tmp` fails on
`Cannot find module 'dotenv'`.

Write it to a dot-prefixed temp file in the repo root, run it, delete it. Do not
commit it.

```js
// .probe.js  (delete after running)
require('dotenv').config({ path: '.env.local', quiet: true });
const { createClient } = require('@supabase/supabase-js');

const db = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
);

(async () => {
  const { data, error } = await db
    .from('tools')
    .select('slug,name,pricing_type,price_from,metadata->pricing')
    .eq('slug', 'some-tool')
    .limit(5);
  if (error) { console.error('ERR', error.message); process.exit(1); }
  console.log(data);
})();
```

```bash
node ./.probe.js && rm ./.probe.js
```

Always branch on `error`. A Supabase client returns `{ data: null, error }` rather
than throwing, so an unchecked call reads as success and then fails one line later on
`data.length`. `CLAUDE.md` requires every Supabase error to be handled in app code
too.

For counts use `.select('*', { count: 'exact', head: true })`. Always `LIMIT` an
exploratory query.

Before writing your own probe, check whether a script already answers the question.
Several read-only ones exist and they carry the project's own logic:
`plan-pricing-sweep.js --status`, `check-pricing-consistency.js`,
`check-subsection-routing.js`, `host-mismatch.js`, `find-discontinued-tools.js`,
`check-comparisons.js`.

## Writing

Never hand-write an `UPDATE` or `INSERT` against the live database. Every write goes
through a script that supports a dry run, because the dry run is what a human
approves.

`node scripts/<name>.js <args>` reads only. The same command with `--execute`
writes. That is the convention across the repo, and the gap between the two is where
the approval goes.

Schema changes (`CREATE`, `ALTER`, `DROP`) are a separate conversation every time and
never something you do as part of another task.

See `references/scripts.md` for which scripts read, which write, and which key each
one needs. See `references/tables.md` for the table inventory and the `tools` columns.
