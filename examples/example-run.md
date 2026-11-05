# Example run: re-engage churned trial accounts

A worked example of the skill in connected mode, from prompt to scheduled send. Numbers in
`[brackets]` are placeholders the user would fill with real values. No real key appears anywhere.

## Prompt

> Plan an email campaign to re-engage trial accounts that churned in the last 60 days. Set it up
> in Mailchimp as a draft, don't send. I've exported MAILCHIMP_API_KEY.

## 1. Intake (inferred, one confirm)

- Goal: get a churned trial to book a 15-minute call. One action: click "Book 15 minutes."
- Audience: read from the API. `GET /lists` finds the audience; a segment of members who went
  inactive in the last 60 days is `[N]` people.
- Structure: single one-off campaign to start, with a two-step follow-up drip proposed as an
  option.
- From: `Brittany at Acme <brittany@acme.com>`, reply-to the same. Verified domain, good.
- Timing: draft only for now.

## 2. Plan (approved at a glance)

- Segment: last activity older than 60 days, was a trial, never converted.
- One email now. If open rate clears `[X]%`, add a step-two nudge after 4 days.
- Success metric: calls booked, not opens.

## 3. Draft

- Subject: `You left mid-trial. Here's the 2-minute version.`
- Alternate: `The part of Acme you probably never reached`
- Preview: `One thing to try, and a link to skip the setup.`
- Body: two short paragraphs, one link (`Book 15 minutes`, UTM-tagged), one line on why now.
  `*|FNAME|*` with a fallback of "there".

## 4. QA

Saved the HTML as `draft.html` and the settings as `meta.json`, then ran the automated pass:
```bash
python3 -I scripts/qa_check.py --html draft.html --meta meta.json
```
It came back `14 passed, 0 warnings, 0 failures`. The link resolves and carries
`utm_source=email&utm_medium=campaign&utm_campaign=trial_reengage`, the merge tag has a fallback,
unsubscribe and address are present, no em dashes. Then walked the human half by eye. No invented
stats; the `[X% of trials that came back converted]` claim was pulled because it could not be
verified.

## 5. Create as a draft

```bash
python3 -I scripts/mailchimp_api.py campaign-create --json campaign.json   # prints draft id
python3 -I scripts/mailchimp_api.py content-set --campaign CAMPAIGN_ID --html draft.html
```

Reported back: campaign ID, and a note that it is a draft visible in their Mailchimp account.
Nothing sent.

## 6. Confirm, test, schedule

- Sent a test to the user's own inbox first:
  ```bash
  python3 -I scripts/mailchimp_api.py test --campaign CAMPAIGN_ID --to brittany@acme.com
  ```
- Showed the pre-send summary from live data, which also prints the exact confirm phrase:
  ```bash
  python3 -I scripts/mailchimp_api.py preview --campaign CAMPAIGN_ID --at 2026-08-25T13:00:00+00:00
  ```
  Audience `[N]` people, from `Brittany at Acme`, subject as above, schedule Tuesday 9:00am ET
  (13:00 UTC). The required phrase was `SCHEDULE campaign CAMPAIGN_ID for 2026-08-25T13:00:00+00:00 to [N] recipients`.
- User replied "yes, schedule it for Tuesday 9am ET." Only then, passing that exact phrase back:
  ```bash
  python3 -I scripts/mailchimp_api.py schedule --campaign CAMPAIGN_ID \
    --at 2026-08-25T13:00:00+00:00 \
    --confirm "SCHEDULE campaign CAMPAIGN_ID for 2026-08-25T13:00:00+00:00 to [N] recipients"
  ```
  A "looks good" would not have matched the phrase, so the gate would have refused it.

## 7. Read back (after it sends)

```bash
python3 -I scripts/mailchimp_api.py report --campaign CAMPAIGN_ID
```
Sends `[N]`, open rate `[X]%`, click rate `[Y]%`, unsubscribes `[Z]`, top link the
`Book 15 minutes` CTA. Reported the handful tied to the goal. If pulled too early, said
"no meaningful data yet, check back tomorrow" rather than reporting zeros.
