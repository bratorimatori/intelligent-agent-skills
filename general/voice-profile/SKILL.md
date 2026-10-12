---
name: voice-profile
description: Derive a reusable writing voice profile from work the user has already published, rather than from how they describe themselves. Run once before the other content skills. Triggers on - set up my voice, learn how I write, onboard me, first time setup.
---

# Building a voice profile from evidence

Run this once. Every other content skill reads what it produces.

Do not ask the user to describe their voice. They will say "conversational but
authoritative" and it will be useless. Ask for the writing instead.

## Step 1: Get the samples

Ask for three to ten pieces they have published and are happy with. Links,
pasted text or files all work. If they offer something they dislike, ask for
that too and label it: knowing what they want to move away from is as useful as
knowing what they want more of.

If they have published nothing, stop and say so plainly. This skill has nothing
to work from, and a profile invented from a questionnaire will make every other
skill worse while appearing to work. Suggest they write three pieces by hand and
come back.

## Step 2: Measure, do not impressionise

Go through the samples and record what is actually there:

- **Sentence length**: the average, and how much it varies. A writer who mixes
  four-word sentences with forty-word ones has a rhythm. Flattening it is the
  single most detectable AI edit.
- **Paragraph length**, in sentences.
- **Person and tense.** First person singular, plural, second person.
- **Contractions**: used or not. Be exact, this is highly visible.
- **Vocabulary level**, and any jargon they use without explaining, which tells
  you who they assume is reading.
- **Recurring moves**: do they open with a story, a claim, a number, a concession?
  How do they close? Do they use questions, and are those questions answered?
- **Formatting habits**: headers, lists, bold, code blocks, pull quotes.
- **Words and constructions they never use.** Absence is signal.

## Step 3: Find the tells

Two or three things that make their writing recognisably theirs. A recurring
structural habit, a way of conceding a point before making one, a particular
kind of joke, a refusal to hedge. Quote a real example of each.

This is the part that matters most and the part that cannot be faked. Do not
invent a tell to fill the slot. If you can only find one, record one.

## Step 4: Write the profile

Save to `~/.claude/voice-profile.md`. Structure:

```markdown
# Voice profile

## Measured
Sentence length: average N, range N to N
Paragraphs: N sentences typical
Person: ...
Contractions: yes / no
Formatting: ...

## Moves
Openings: ...
Closings: ...
Argument shape: ...

## Tells
1. ... e.g. "quoted example"
2. ...

## Never
Words, structures and habits absent from every sample.

## Audience
Who the writing assumes is reading, inferred from what goes unexplained.

## Topics and stance
What they write about and what they argue for.

## Products
Anything they sell, and how they mention it. Note whether mentions are soft or
direct, because guessing this wrong is the most embarrassing failure mode.
```

## Step 5: Test it before you trust it

Take one of the samples. Show the user its first paragraph rewritten from the
profile alone, without looking at the original. If it does not read like them,
the profile is wrong, and it is much cheaper to find that out now than after it
has shaped forty posts. Adjust and repeat once.

Then show them the profile and ask what is wrong with it. They will correct one
or two things immediately, and those corrections are usually the most valuable
lines in the file.
