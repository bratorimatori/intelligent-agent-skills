---
name: repo-onboard
description: Write a CLAUDE.md for an existing repository from what the code actually does, not from what it should do. Use when setting up a new project for agent work or when the existing instructions have drifted. Triggers on - set up CLAUDE.md, onboard this repo, write project instructions.
---

# Writing project instructions that hold up

A CLAUDE.md written from imagination is worse than none, because it is trusted.
Write it from evidence.

## Read before you write

Before a single line: the package manifest and its scripts, the CI config, the
directory layout two levels deep, the last fifty commit messages, and any
existing docs. The commit messages are the most useful and the least read. They
tell you what actually breaks.

## What belongs in it

In descending order of value:

1. **The non-negotiables.** What must never happen, stated as rules with
   consequences. Production writes, secrets, destructive commands.
2. **The definition of done.** The exact command that proves a change is
   finished, and what it does not prove.
3. **Commands that are not discoverable.** The script with the flag that must be
   passed, the two-step sequence where step one is easy to forget.
4. **Conventions with a reason attached.** "No em-dashes in database text,
   because it breaks the CSV export" survives. "Prefer clean code" does not.
5. **Structure**, briefly. Agents can read the tree themselves.

## What to leave out

Anything the repository already says. Do not restate the dependency list, the
framework's own conventions, or what a linter enforces. Every line competes for
attention with every other line, and a long file is skimmed.

## Verify every command you document

Run them. A documented command that does not work trains the reader to distrust
the whole document, and the SQL template that has a syntax error in it will be
copied by everyone who follows the instructions.

Check the docs against the code specifically where they overlap. A template
listing columns the applier does not accept, or a described flag that no longer
exists, is the failure mode here. It is worth one pass through every code block.

## Write down the incidents

The most valuable section in a mature CLAUDE.md is the list of things that went
wrong and what was changed as a result. "Batch 1 replies were lost because the
domain had no MX record. Test the round trip, not just the send." That is a
sentence nobody can derive from the code, and it is the reason the file exists.

Add to this section every time something breaks. It is the only part that gets
more valuable with age.

## Keep it honest about what it cannot enforce

If a rule depends on a person following it, say so rather than implying it is
enforced. A reader who discovers a "required" check is optional stops believing
the required ones.
