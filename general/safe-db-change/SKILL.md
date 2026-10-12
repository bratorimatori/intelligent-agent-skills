---
name: safe-db-change
description: Take a production database change through generate, validate, dry run, ask, execute, verify. Use before any INSERT, UPDATE, DELETE, migration or schema change against a live database. Triggers on - update the database, run this migration, fix these rows, apply the batch.
---

# Changing a production database

The gate is the ask, not the actor. An agent may run the migration. It may never
decide on its own that now is the moment.

## The sequence

1. **Generate.** Write the change to a file. Never type it straight into a client.
2. **Validate.** Run whatever content checks the project has against that file.
3. **Dry run.** Execute it in a mode that reads only and prints exactly which
   rows would change, before and after.
4. **Show and wait.** Post the dry run output and stop. The turn ends here.
5. **Execute.** Only after an explicit written yes.
6. **Verify.** Read the rows back, with the *least* privileged credential
   available, not the one that did the write.

Step 6 is the one people skip. The database accepting a write is not the same as
a visitor seeing it. Read it back the way a visitor would.

## What counts as approval

"Go", "apply it", "run it", "execute" are approval. Silence is not. A thumbs-up
on a different question is not. Asking for the work is not approval to write it:
that approves the content, not the write.

Approval is per batch and it expires with the batch. A yes for this batch is
not a standing yes for the next one. If you are unsure whether something counted
as approval, it did not. Ask again.

## Post this before the ask

```
DATABASE OPERATION CHECKPOINT

Rule Check:  which rule applies
Operation:   what is being done
Impact:      which rows, which columns, how many
Reversible:  YES or NO, and how
Approval:    PENDING
```

Then the dry run output. Then stop.

## Writing the change itself

- **Always a WHERE clause** on UPDATE and DELETE. No exceptions.
- **Assert the row count before writing.** If a name lookup is expected to match
  one row, check that it matched one row and abort otherwise. A rename upstream
  turns a targeted update into a table-wide one.
- **Append, do not overwrite, anything that is a record of what happened.** A
  status column that says which message someone was sent is history. Replacing
  it destroys the only evidence of what you promised them.
- **Make re-runs no-ops.** Guard inserts on existence so a second run is safe.
  You will re-run it.
- **Guard the content rules in code, not in your head.** If the project forbids
  certain characters in database text, have the script refuse rather than
  trusting yourself to notice.

## Schema changes

CREATE, ALTER and DROP get the same sequence and are never routine. When a new
table holds behavioural data or anything money depends on, enable row-level
security and add no public policy at all, then verify with the public key that
it reads back empty and refuses a write. An unreadable table is easy to open
later. A leaked one cannot be closed retroactively.
