# Newsletter Issue Skill

Builds and sends the intelligenttools.co newsletter. Runs every two weeks.

## What This Skill Does

- Checks when the last issue went out and whether one is due
- Picks 5 recently added tools worth writing about, across different categories
- Verifies every price against the tool's own row before it is written down
- Drafts the issue in HTML and plain text, in the house voice
- Runs the house rules: no em-dashes, live tool count, working links
- Sends a draft to you first, then the list only when you say so

## When to Use This Skill

- Every two weeks, when `newsletter/SENT-LOG.md` says an issue is due
- When you want an issue drafted early
- When you want to check what is in the queue without sending anything

## Step 1: Is an issue due?

Read `newsletter/SENT-LOG.md`. The last row carries the send date and the next
due date. Issues go out every two weeks. If the next due date has not arrived,
say so and stop unless the user wants one anyway.

The next issue number is the last one plus 1.

## Step 2: Get the real numbers

Never write a tool count by hand. `OUTREACH-GUIDE.md` records a near miss where
"400+" would have been false at 399 tools.

```js
const { count } = await supabase.from('tools').select('*', { count: 'exact', head: true });
const label = `${Math.max(50, Math.floor(count / 50) * 50)}+`;   // matches lib/tool-count.ts
```

Also count what has been added since the previous issue's send date, and round
that down too. Both numbers go in the opening line.

## Step 3: Pick the tools

Query tools added since the last issue, free or freemium or open source:

```js
const { data } = await supabase
  .from('tools')
  .select('name, slug, tagline, description, pricing_type, pricing_info, tool_categories(slug)')
  .gte('created_at', LAST_SEND_DATE)
  .in('pricing_type', ['free', 'freemium', 'open_source'])
  .order('created_at', { ascending: false });
```

Pick **1 featured + 4 others, each from a different category**. What makes a
good pick, in order:

1. **A real free tier**, not a trial. The audience is developers who will not
   hand over a card to evaluate something.
2. **One specific, checkable fact** worth a sentence: a pricing model that is
   unusual, processing that runs locally, an actual constraint.
3. **Relevance to people who use coding agents.** A status bar for Claude Code
   lands better here than another image generator.
4. **Something you can say honestly.** If the only available line is the
   tagline reworded, pick a different tool.

Prefer tools whose pricing you verified when they were added. Say in the copy
if a claim comes from the vendor rather than from checking.

## Step 3b: The sponsored slot

A tool on a featured placement gets one clearly labelled sponsored block in
every issue sent while the placement runs. It is **in addition to** the 5
editorial picks, never one of them, and never the featured tool. The featured
slot is an editorial choice, and a paid (or free-trial) placement sitting in
it unlabelled would mislead the people who trust it.

```js
const { data: sponsored } = await supabase
  .from('tools')
  .select('name, slug, tagline, description, pricing_type, pricing_info, metadata')
  .eq('sponsored', true);
```

The created_at filter in Step 3 does not apply here: a sponsored tool is
usually older than the last issue.

- **Label it.** The block heading says "Sponsored", and one sentence says what
  that means, e.g. "Mowgli has a free featured placement on the site this
  month." Say "free trial placement" rather than "paid" if no money changed hands.
- **Same honesty rules as everything else.** Facts come from the row and its
  `metadata.pricing`. If nobody here has used the tool, say so ("I have not
  tested it yet") rather than writing as if we had. If a hands-on review
  exists on the blog, link that instead of the bare listing.
- **Tag the link** `?utm_source=newsletter&utm_medium=email&utm_campaign=issue-N-sponsored`
  so the click report can tell newsletter clicks from search clicks on the
  sponsor's listing. The listing's own Visit button already carries the
  sponsor's UTM.
- **Check the dates.** Only include a tool whose placement is still running on
  the send date. The end date is in the tracking issue for that placement
  (Mowgli: #108, ends 2026-10-30).

## Step 4: Write it

Copy `newsletter/issue-1.html` and `issue-1.txt` to the new issue number and
replace the content. Keep the structure:

- **Opening**, 2 to 3 sentences. What was added, how many tools now, what this
  issue is.
- **Featured tool**, 3 to 4 sentences: what it does, what makes it different,
  who should try it, and one honest limitation.
- **4 more**, 2 sentences each: what it does, why it is interesting.
- **Sponsored**, only when Step 3b found one: labelled, 2 to 3 sentences,
  after the editorial picks so it never reads as one of them.
- **This week on the blog**, if there is a recent post. The phrase stays "this
  week" even on a fortnightly send.
- **Footer**, unchanged, including the unsubscribe link.

Voice: short, honest, no hype. Like a friend recommending tools. First person
is fine. Say when something is not worth it, and say who should skip a tool.
Never write a claim you have not checked.

## Step 5: Check before sending

- **No em-dashes or en-dashes.** The sender aborts on them, but catch it first.
- **Both files updated**, HTML and text, with the same content.
- **Every link returns 200.** Curl each one. A dead link in an email cannot be
  fixed after sending.
- **Prices match the database.** Reread each against `pricing_info`.
- **The tool count is derived**, not typed.

## Step 6: Send

```bash
node scripts/send-newsletter.js --issue=N                     # dry run, shows the list
node scripts/send-newsletter.js --issue=N --draft --execute   # one copy to the owner
node scripts/send-newsletter.js --issue=N --execute           # the whole list
```

**Always send the draft first and wait.** Ask the user to read it in Gmail, not
a browser, and to hit Reply once to confirm the round trip. Only send to the
list when they say to.

The script logs real sends to `newsletter/SENT-LOG.md` automatically. Do not
write that file by hand.

## Rules

1. **Never send to the list without explicit approval in the current
   conversation.** A send is irreversible and goes to real people. Approval to
   draft is not approval to send.
2. **Never invent a price, a feature or a user count.** Everything traces to the
   tool's row or its live pricing page.
3. **Never hardcode the directory size.**
4. **A draft is not a send.** Only real sends are logged.
5. **Check the subscriber list before a send** for disposable domains, role
   addresses and malformed entries. Report anything odd rather than silently
   mailing it.

## Sender configuration

`NEWSLETTER_FROM` is "Boki from Intelligent Tools <boki@intelligenttools.co>",
deliberately separate from `SMTP_FROM`, which outreach uses. Subscribers were
welcomed by "Intelligent Tools", so a bare personal name reads as an unknown
sender to them. `SMTP_REPLY_TO` must be set: the sending domain bounces replies.

## Related

- `newsletter/SENT-LOG.md` - when each issue went out and when the next is due
- `OUTREACH-GUIDE.md` - deliverability lessons, all of which apply here
- `text-proofreader` - run it over the issue before sending
- `seo-blog-validator` - for the linked blog post, not the newsletter
