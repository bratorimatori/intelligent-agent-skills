---
name: blogging-anti-patterns
description: "Structural mistakes that lose readers in software blog posts: meandering intros, knowledge assumptions, link overuse, sequel dependencies, excessive formality, and mobile rendering failures. Use as an editing checklist alongside stop-slop."
---

# Blogging Anti-Patterns

Six structural mistakes that lose readers in software blog posts, distilled from Michael Lynch's "Anti-Patterns in Software Blogging" (October 2026, https://refactoringenglish.com/blog/anti-patterns-software-blogging/).

This skill covers **post structure**. It complements stop-slop, which covers **prose patterns**. Use both: stop-slop catches AI writing tells at the sentence level, this catches blogging mistakes at the section and page level.

## The Six Anti-Patterns

### 1. The Meandering Intro

Developers start posts with backstory, historical context, and whatever else is on their minds. Readers have a billion other articles they could read. They won't invest 20 minutes unless they expect a payoff.

**The two questions every reader asks:**
1. Did the author write this for someone like me?
2. How will I benefit from reading it?

**Rule:** Answer both in the title and first three sentences.

Benefits you can offer: teaching a skill, explaining a concept, illustrating a perspective, or delivering an entertaining rant. The reader won't read your post just because it's there.

**Preamble counts as meandering.** Subtitles, bios, images, and famous quotes all chip away at the reader's finite attention before you've given them a reason to care.

### 2. Reader Knowledge Assumptions

"The reader knows everything I know except this one thing" is rarely true. Developers explaining Docker assume the reader knows cgroups, jails, or *BSD. Many Docker users don't know any of those terms.

**Fix:** Think of a specific person you know. List what they would and wouldn't recognize. Re-read your post and check every technical term against that list.

### 3. Overreliance on Links

When's the last time you read a book that directed you to stop reading, go buy a different book, read it in full, then continue your original book?

Bloggers slap a link on unfamiliar terms and think "problem solved." It isn't. The reader doesn't want to read a 20,000-word manual to understand one sentence of your post.

**Fix:** Give the reader the minimum explanation needed to understand your article. Links are a bonus, not a prerequisite. Your reader should be able to finish your article without clicking anything.

### 4. The Sequel Injection Bug

"In part one, we learned about X. In today's post, I'll show you Y (a term I invented in part one—remember?)."

Most readers have not read part one. If you assume your last article is fresh in their mind, they'll think "Oh, now there's extra work to even *start* reading?"

**Fix:** Summarize what's relevant instead of forcing the reader to go back. Most sequel posts could be standalone with 3% more effort.

### 5. Excessive Formality

> "Several static analysis tools were utilized by my teammates and myself throughout the duration of this project's lifetime."

You're not writing for 1988 IBM executives. Your reader is probably in pajamas eating cereal. With AI making writing bland and homogenous, readers are hungry for personality.

**Fix:** Write how you talk. Joel Spolsky, Kathy Sierra, Terence Eden, and Raymond Chen sound like themselves telling stories to friends at lunch. They're not trying to sound smart.

### 6. Fumbling HTML Basics

**Mobile overflow:** Images or code snippets that don't resize force readers to scroll horizontally. Check Firefox/Chrome's mobile preview before publishing.

**Unreadable fonts:** Dark gray text on light gray background. Use browser accessibility tools to flag low-contrast text. If unsure, the Braille Institute's Atkinson Hyperlegible font is free and comfortable for readers with poor vision.

## Checklist

Before publishing:

- [ ] Title + first 3 sentences answer: who is this for, and what's the payoff?
- [ ] No preamble (subtitle, bio, image, quote) blocking the reader from the intro
- [ ] Technical terms checked against what a real person I know would recognize
- [ ] No links used as explanation shortcuts—every term explained inline or not used
- [ ] Post is standalone—no assumed knowledge from previous posts
- [ ] Post sounds like me talking, not a legal document
- [ ] Mobile preview shows no horizontal scroll
- [ ] Font has sufficient contrast (browser accessibility tools)

## Key Principles

> "Give the reader a reason to continue reading."

> "Keep the reader on the page."

> "Write the way you talk."

## Source

Michael Lynch, "Anti-Patterns in Software Blogging," October 7, 2026.
https://refactoringenglish.com/blog/anti-patterns-software-blogging/

Lynch is the author of *Refactoring English: Effective Writing for Software Developers* and runs https://mtlynch.io.
