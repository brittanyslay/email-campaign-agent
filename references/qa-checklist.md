# Pre-send QA checklist

Run this before any campaign is created, and again before any send. Fix what you can. Flag what
needs the user in `[brackets]`. A campaign goes to real inboxes once; catch it here.

## Subject and preview
- [ ] Subject line is set, is not truncated on mobile (aim under ~50 characters), and matches
      the email's actual content. No clickbait the body does not pay off.
- [ ] Preview text is set and does not repeat the subject word for word.
- [ ] One alternate subject is offered for the user to A/B or swap.

## Links and tracking
- [ ] Every link resolves (no `#`, no `example.com`, no dead placeholder).
- [ ] Every marketing link carries the right UTM tags (source, medium, campaign) so clicks are
      attributable.
- [ ] No internal or staging URLs left in the body.

## Personalization
- [ ] Every merge tag (for example `*|FNAME|*`) is spelled correctly and has a fallback so it
      never renders "Hi ,".
- [ ] Any personalization that depends on a data field confirms that field actually exists on
      the audience. If it might be empty, it has a fallback.

## Compliance and deliverability
- [ ] Unsubscribe link is present (Mailchimp adds `*|UNSUB|*`; confirm it is not removed).
- [ ] A physical mailing address is present (CAN-SPAM and GDPR).
- [ ] From name and from email are correct and on a domain the account is verified to send from.
- [ ] Reply-to goes somewhere a human reads.
- [ ] The list is opt-in. No purchased, scraped, or rented contacts.

## Content and rendering
- [ ] Images have alt text and are not the only way the message reads (image-blocked inboxes).
- [ ] The plain-text version is not empty.
- [ ] No em dashes or en dashes anywhere in the copy.
- [ ] No invented numbers, dates, quotes, or claims. Anything unverified is in `[brackets]`.
- [ ] Reads like a specific person wrote it, not a template.

## Final gate
- [ ] A test send was offered; if the user wanted it, it went to their own inbox and looked right.
- [ ] The pre-send summary (audience, count, from, subject, send time) was shown and the user
      gave an explicit, specific yes.
