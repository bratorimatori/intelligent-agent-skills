---
name: pricing-read
description: Read a vendor's real pricing off their own page and record it under one consistent convention. Use when adding a product to a catalogue, comparing tools, or auditing a price column that has drifted. Triggers on - what does it cost, check the pricing, add this tool, compare prices.
---

# Reading pricing you can defend

Pricing is the field most often wrong in any catalogue, because it is copied
from other catalogues and it changes without notice.

## Get it from the vendor, rendered

Not from another directory. Not from your memory of the product. Not from the
raw HTML if the page is client-rendered, which most pricing pages now are: fetch
it in a real browser or you will conclude a page has no prices when it has four
tiers.

Note the date you checked. A price without a date is a claim about the present
that quietly becomes a lie.

## Record a structured ladder, and derive the columns from it

Store the tiers as data, not as a sentence:

```json
{ "checked_at": "2026-09-18",
  "free_kind": "tier",
  "billing_basis": "monthly",
  "evidence": "Monthly toggle selected. Free $0. Pro $20/mo. Enterprise, talk to sales.",
  "tiers": [
    { "name": "Free",       "monthly_usd": 0,    "self_serve": true },
    { "name": "Pro",        "monthly_usd": 20,   "self_serve": true,
      "annual_usd": 17, "annual_total_usd": 200 },
    { "name": "Enterprise", "monthly_usd": null, "self_serve": false }
  ] }
```

Then every column you sort and filter on is arithmetic over that, not a second
judgement kept alongside it:

```
price_from   cheapest monthly_usd where self_serve
price_to     dearest  monthly_usd where self_serve
type         paid + free -> freemium; paid only -> paid; all zero -> free
free tier    free_kind === "tier"
```

**`self_serve` is the field people leave out, and it is the one that makes the
range derivable.** `price_from` and `price_to` are not the ladder's minimum and
maximum - they are the cheapest and dearest tier a reader can actually *buy*. An
"Enterprise $2,499, talk to sales" tier is a real number nobody can pay, and
without that flag it becomes your published ceiling.

Do not infer it from a null price either. "$9 one-time, 40 credits" has no
monthly figure and is perfectly buyable.

## A tier is a plan, never a billing period

If the page lists "Monthly $25" and "Yearly $6.50/mo" for the same plan, that is
ONE tier: `monthly_usd: 25, annual_usd: 6.50`. Emitting two makes $6.50 look
like something you can buy for a single month, and your cheapest-price column
now advertises a rate the vendor does not sell.

## Record both billing periods, in their own fields

A plan with an annual discount publishes up to three different numbers:

| field | means | example |
|---|---|---|
| `monthly_usd` | month to month | "$20/month" |
| `annual_usd` | monthly equivalent, billed yearly | "$17/mo billed yearly" |
| `annual_total_usd` | the yearly charge | "$200/yr", "$96/year" |

Record what the page shows and **never calculate one from another**. A vendor
publishing only "$96/yr" gets `annual_total_usd: 96` and `annual_usd` left out -
writing $8 there quotes a figure they never printed.

Collapsing these into one column is the most common way a catalogue ends up with
a confidently wrong price, because the number is real and simply answers a
different question than the column asks.

## The prose derives from the ladder, not the other way round

Keep a human sentence as well, but treat it as output. The moment prose becomes
an independent source, it drifts: in one catalogue of ~570 products, an audit
found 87 rows contradicting themselves and 148 whose prose quoted a price the
structured data did not contain. One product's search snippet advertised "a
steep jump to $120 a month" while its own pricing panel said the entry plan was
free.

Nothing had gone wrong at write time. The two fields were written months apart
by different processes, and nothing ever compared them.

## Contradictions between the prose and the numbers are bugs

If the text says "no free tier" and `price_from` is 0, one of them is wrong and
the tool is sorting as free. Audit for this directly: it is the highest-value
check in the whole catalogue and it costs one query.

Make it a script that exits non-zero, and run it on every row you touch. A
convention nothing enforces is a convention that has already drifted.

## Things that look like a price and are not

- **A crossed-out list price** next to the real one.
- **Annual billing shown as a monthly number.** Most pricing pages open on the
  annual toggle, because it shows a lower figure next to a "save 20%" badge.
  "$12/month billed yearly" is a $144 commitment, not a $12 one. Toggle to
  monthly before reading anything, record both, and say in your evidence which
  toggle you read.
- **A stated saving.** "Save $240/year" is not a price and belongs in no field.
- **A countdown offer.** If the timer resets on reload, the discount is
  permanent and the urgency is decoration. Record the standing price.
- **Credits.** "300 credits" means nothing without the burn rate. Find how many
  credits an operation costs, or record that it is unstated.

## Re-check on a schedule

Prices derived from prose and never checked against a vendor page are the
long-lived error in any catalogue. Track which rows were verified against the
real page and which were inferred, and work through the backlog. Spot-checking
three of them is usually enough to tell you how bad it is.
