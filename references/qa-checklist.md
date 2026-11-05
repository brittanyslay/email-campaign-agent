<!--
Email Campaign Agent, part of a Claude skill by Brittany Slay (https://brittanyslay.com).
Licensed for noncommercial use only (PolyForm Noncommercial 1.0.0). See LICENSE.md.
Required Notice: Copyright Brittany Slay (https://brittanyslay.com)
-->

# Pre-send QA checklist

Run this before any campaign is created, and again before any send. Fix what you can. Flag what
needs the user in `[brackets]`. A campaign goes to real inboxes once; catch it here.

Most of this list is automated. Run the script first, fix every failure, then walk the rest by
eye for the things a script cannot judge:
```bash
python3 -I scripts/qa_check.py --html draft.html --meta meta.json
```
A non-zero exit is a hard gate. The boxes it covers are marked `[auto]` below; the unmarked ones
are the human read: does the subject match the body, does the copy sound like a specific person.

## Subject and preview
- [ ] `[auto]` Subject line is set and is not truncated on mobile (aim under ~50 characters).
- [ ] Subject matches the email's actual content. No clickbait the body does not pay off.
- [ ] `[auto]` Preview text is set and does not repeat the subject word for word.
- [ ] `[auto]` One alternate subject is offered for the user to A/B or swap.

## Links and tracking
- [ ] `[auto]` Every link resolves (no `#`, no `example.com`, no dead placeholder).
- [ ] `[auto]` Every marketing link carries UTM tags (source, medium, campaign) so clicks are
      attributable.
- [ ] `[auto]` No internal or staging URLs left in the body.

## Personalization
- [ ] `[auto]` Every merge tag (for example `*|FNAME|*`) has a fallback so it never renders "Hi ,".
- [ ] Any personalization that depends on a data field confirms that field actually exists on
      the audience. If it might be empty, it has a fallback.

## Compliance and deliverability
- [ ] `[auto]` Unsubscribe link is present (Mailchimp adds `*|UNSUB|*`; confirm it is not removed).
- [ ] `[auto]` A physical mailing address is present (CAN-SPAM and GDPR).
- [ ] `[auto]` From email is not a free inbox; it is on a domain the account is verified to send from.
- [ ] Reply-to goes somewhere a human reads.
- [ ] The list is opt-in. No purchased, scraped, or rented contacts.

## Content and rendering
- [ ] `[auto]` Images have alt text and are not the only way the message reads (image-blocked inboxes).
- [ ] `[auto]` The plain-text version is not empty.
- [ ] `[auto]` No em dashes or en dashes anywhere in the copy.
- [ ] No invented numbers, dates, quotes, or claims. Anything unverified is in `[brackets]`.
- [ ] Reads like a specific person wrote it, not a template.

## Final gate
- [ ] A test send was offered; if the user wanted it, it went to their own inbox and looked right.
- [ ] The pre-send summary (audience, count, from, subject, send time) was shown and the user
      gave an explicit, specific yes.
