---
name: send-safely
description: Pre-flight a recipient list and dry-run an email send before anything leaves the machine. Use before any outbound email to more than one person - outreach, newsletters, notifications. Triggers on - send the emails, mail the list, send the newsletter, email these people.
---

# Sending email to real people

Sending is irreversible and outward facing. The list is the asset, and a bad
send damages it permanently.

## The sequence

1. **Build the list** with a script that only reads. It must not be able to send.
2. **Pre-flight every address** before applying any limit, so a cap of 20 gives
   20 usable recipients rather than 20 rows of which several bounce.
3. **Read the list yourself.** Delete anyone who should not be on it.
4. **Dry run.** Default to it. Sending requires an explicit flag.
5. **Send one to yourself first** and open it in a real mail client, not a
   browser preview. This is the step that catches spam placement while it is
   still cheap.
6. **Send**, with spacing between messages.
7. **Check the provider dashboard within the hour** for bounces.

## Sent is not delivered

A send script reporting "sent 20, failed 0" means the provider accepted 20
messages. It says nothing about inboxes. Only the provider's own delivery log
knows. If your API key is send-only, say so and open the dashboard rather than
implying the script proved delivery.

## What the address pre-flight should catch

Ranked by what actually bounces, not by what is easy to check:

- **Typo domains.** `gamil.com` resolves and has a valid MX record because it is
  a registered typo-squat. Only an explicit list catches these.
- **Role addresses**, for cold outreach: `support@`, `hello@`, `info@`. They
  reach a shared inbox, not a person who can say yes.
- **Disposable and relay domains.**
- **Domains with no mail exchanger**, which catches genuinely dead domains.

An MX check alone catches the least interesting of these. Run it, but do not
mistake it for the check that matters.

**Role addresses are context-dependent.** Drop them from cold outreach. Keep
them on a newsletter: somebody who subscribed as `team@company.com` asked for
it. Applying the outreach rule to a subscriber list silently unsubscribes people
who opted in.

## Bounces are permanent on a subscriber list

An outreach bounce happens once. A subscriber list is forever, so one bad
address bounces on every issue from now on, and repeated bounces to a dead
address is exactly what costs a young sending domain its deliverability. Keep a
list of confirmed bounces and exclude them by hand until the pre-flight learns
them.

## Test the round trip, not just the send

A sending domain is not automatically a receiving domain. Mail can go out
perfectly from a domain with no MX record, and every reply then bounces into
nothing. Send yourself a message and hit Reply before you trust the address you
are asking a hundred people to write to.

## Record what each person was told

If you send different versions to different people, store which one each
received, and append rather than overwrite when you follow up. A reply arriving
three weeks later is unreadable without it.
