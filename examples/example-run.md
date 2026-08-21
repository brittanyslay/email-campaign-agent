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

Ran `references/qa-checklist.md`. Link resolves and carries
`utm_source=email&utm_medium=campaign&utm_campaign=trial_reengage`. Merge tag has a fallback.
Unsubscribe and address present. Plain-text generated. No em dashes. No invented stats; the
`[X% of trials that came back converted]` claim was pulled because it could not be verified.

## 5. Create as a draft

```bash
DC="${MAILCHIMP_API_KEY##*-}"
BASE="https://$DC.api.mailchimp.com/3.0"
# create draft
curl -sS -u "key:$MAILCHIMP_API_KEY" -X POST "$BASE/campaigns" -d '{ ...settings... }'
# set content
curl -sS -u "key:$MAILCHIMP_API_KEY" -X PUT "$BASE/campaigns/CAMPAIGN_ID/content" -d '{ "html": "..." }'
```

Reported back: campaign ID, and a note that it is a draft visible in their Mailchimp account.
Nothing sent.

## 6. Confirm, test, schedule

- Offered a test send to the user's inbox first.
- Showed the pre-send summary: audience `[N]` people, from `Brittany at Acme`, subject as above,
  proposed schedule Tuesday 9:00am ET (13:00 UTC).
- User replied "yes, schedule it for Tuesday 9am ET." Only then:
  ```bash
  curl -sS -u "key:$MAILCHIMP_API_KEY" -X POST "$BASE/campaigns/CAMPAIGN_ID/actions/schedule" \
    -d '{ "schedule_time": "2026-08-25T13:00:00+00:00" }'
  ```

## 7. Read back (after it sends)

`GET /reports/CAMPAIGN_ID`: sends `[N]`, open rate `[X]%`, click rate `[Y]%`, unsubscribes `[Z]`,
top link the `Book 15 minutes` CTA. Reported the handful tied to the goal. If pulled too early,
said "no meaningful data yet, check back tomorrow" rather than reporting zeros.
