---
name: article-draft
description: Draft a long-form article in the user's voice from a topic and an angle, refusing to proceed without an argument. Use for Medium posts, blog posts and newsletters over 800 words. Triggers on - write an article, draft a blog post, write a Medium piece.
---

# Drafting an article

Read `~/.claude/voice-profile.md` first. If it does not exist, run
`voice-profile` before anything else.

## Step 1: Refuse to start without an angle

Ask two questions and wait for real answers:

1. **What are you actually claiming?** Not the topic. The claim. "AI coding
   tools" is a topic. "Most AI coding tools optimise for the demo, not the
   second week" is a claim.
2. **What would someone reasonably say against it?**

If they cannot answer the first, the piece has no argument and will read like
every other piece on the topic. Say that, and offer to help find the angle
rather than drafting around the gap. This is the single highest-leverage moment
in the whole process and it takes one exchange.

The second question matters because a piece that never acknowledges the
counter-argument reads as either naive or dishonest, and it is the fastest way
to sound like a person who has actually thought about something.

## Step 2: Gather the specifics

Ask what they have: numbers, a story, a screenshot, something that happened to
them, a thing they got wrong. One concrete specific carries more weight than
three paragraphs of reasoning.

**Never invent these.** No fabricated statistics, no invented anecdotes, no
"studies show". If they have no specifics, the piece will be thinner and it is
better to say so than to fill the gap with plausible fiction. A made-up number
in a published piece is the one mistake that cannot be walked back.

## Step 3: Structure before prose

Propose the shape in five or six lines and get agreement before writing
paragraphs. Rewriting a structure costs a minute. Rewriting prose costs an hour.

A shape that works for an argument:

1. The specific thing that happened, or the number
2. Why the obvious reading of it is wrong
3. The claim
4. The strongest objection, taken seriously
5. What follows if the claim holds
6. What the reader should do differently

## Step 4: Draft

Match the voice profile on sentence length variance, contractions, person and
formatting. Variance matters more than average: uniform sentence length is the
most recognisable tell in AI prose.

The opening does one job: make the second sentence necessary. Open with the
specific, not the context. "Last month I lost four hours to a tool that had
shipped a breaking change" earns attention. "In today's rapidly evolving
landscape of AI development tools" does not.

**Never write:**

- A rhetorical question as the first line
- "In today's fast-paced world" or any variant
- "It's important to note that", "Let's dive in", "In conclusion"
- A closing paragraph that summarises the piece you just read
- Three-item lists where two items would do
- Structure announcements: "First, I'll explain X, then Y"
- Enthusiasm the writer does not feel

## Step 5: Cut

Read the draft and remove: every sentence that restates the previous one, every
qualifier that weakens a claim the writer actually believes, every transition
that only exists to be a transition. Expect to cut ten to twenty percent.

Then check the opening again. Most drafts can lose their first paragraph
entirely and start at the second.

## Step 6: Hand back honestly

Say which claims came from the user, which came from a source, and which are
unsupported and should be cut or softened before publishing. If a product
mention was requested, show it separately so they can judge whether it sits
naturally or reads like an advert.
