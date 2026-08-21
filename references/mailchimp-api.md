# Mailchimp Marketing API v3, the parts this skill uses

This is the endpoint map for connected mode. Follow it as guidance, not as code to run blind.
Every send or schedule call is gated on an explicit user confirmation. Never store, print, or
commit the key.

## Auth and base URL

- Mailchimp uses HTTP Basic auth. The username can be any string; the password is the API key.
- The key ends in a datacenter suffix after a dash, for example `...-us21`. That suffix is the
  datacenter. The base URL is `https://<dc>.api.mailchimp.com/3.0/`, so a `us21` key uses
  `https://us21.api.mailchimp.com/3.0/`.
- Read the key from the environment, never inline it:
  ```bash
  export MAILCHIMP_API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-us21"
  DC="${MAILCHIMP_API_KEY##*-}"          # -> us21
  BASE="https://$DC.api.mailchimp.com/3.0"
  ```
- In examples, always mask the key. Use `-u "anystring:$MAILCHIMP_API_KEY"` so the shell
  substitutes it and it never appears in the printed command.

## Audiences and lists

- List audiences: `GET /lists`
- One audience: `GET /lists/{list_id}`
- Members and counts: `GET /lists/{list_id}/members?count=10`
  ```bash
  curl -sS -u "key:$MAILCHIMP_API_KEY" "$BASE/lists"
  ```

## Segments

- Read: `GET /lists/{list_id}/segments`
- Create from conditions: `POST /lists/{list_id}/segments`
  ```json
  {
    "name": "Opened last campaign, no click",
    "options": {
      "match": "all",
      "conditions": [
        { "condition_type": "Aim", "field": "aim", "op": "open", "value": "CAMPAIGN_WEB_ID" }
      ]
    }
  }
  ```
  Campaign open/click segments use `condition_type: "Aim"`, `field: "aim"`, and an `op` of
  `open` / `click` / `sent` / `noopen` / `noclick` / `nosent`. The `value` is the campaign's
  numeric `web_id` (the number in the Mailchimp UI URL), NOT the alphanumeric campaign `id`.
  Get it from `GET /campaigns/{id}` (the `web_id` field). There is no `CampaignActivity`
  condition type; using it returns a 400.

## Create a campaign as a draft

`POST /campaigns` returns a draft. It does not send.

```json
{
  "type": "regular",
  "recipients": {
    "list_id": "LIST_ID",
    "segment_opts": { "saved_segment_id": 12345 }
  },
  "settings": {
    "subject_line": "The one number that moved last quarter",
    "preview_text": "Not a roundup. One change and what it did.",
    "title": "Q3 re-engagement (internal name)",
    "from_name": "Brittany at Acme",
    "reply_to": "brittany@acme.com"
  }
}
```

The response `id` is the campaign ID used by every call below.

## Set content

`PUT /campaigns/{campaign_id}/content`

```json
{ "html": "<html>... your email ...</html>" }
```

Let Mailchimp auto-generate the plain-text part, or send a `plain_text` field. Never ship an
empty plain-text version.

## Test send (do this first)

`POST /campaigns/{campaign_id}/actions/test`

```json
{ "test_emails": ["you@yourdomain.com"], "send_type": "html" }
```

## Send (confirmation-gated, irreversible)

`POST /campaigns/{campaign_id}/actions/send` with an empty body. Only after the user gives an
explicit, specific yes and has seen the pre-send summary.

## Schedule and unschedule (confirmation-gated)

- Schedule: `POST /campaigns/{campaign_id}/actions/schedule`
  ```json
  { "schedule_time": "2026-08-25T13:00:00+00:00" }
  ```
  Mailchimp requires the minute to be 0, 15, 30, or 45. Times are UTC in the API; convert the
  user's local time and confirm both back to them.
- Unschedule: `POST /campaigns/{campaign_id}/actions/unschedule`

## Reports

- Summary: `GET /reports/{campaign_id}` (sends, opens, open rate, clicks, click rate,
  unsubscribes, bounces).
- Clicked links: `GET /reports/{campaign_id}/click-details`
- Per-recipient activity: `GET /reports/{campaign_id}/email-activity`

A campaign that just sent has little to no data yet. Say so rather than reporting zeros as if
they were the result.

## Automations and Customer Journeys

The classic Automations API (`/automations`) is limited and largely superseded, and newer Customer
Journeys are only partly exposed through the API. So do NOT reach for the Journey builder by
default. Build the drip as an API-scheduled campaign sequence, which the API fully supports and
which leaves the user wiring nothing:

1. Plan every step, trigger, and delay fully in copy.
2. Create each step as its own campaign (`POST /campaigns`) in DRAFT, targeted at the segment that
   defines who is at that stage: an entry segment for step 1, then for each later step a segment of
   people who did or did not act on the prior step (opened, clicked, or bought). Set content.
3. Before each step goes out, re-score its segment via `POST /lists/{id}/segments` (or a saved
   segment) so it reflects who actually acted. That is an API call the skill makes, not a user task.

**Every scheduled step is a real send to real people, so every step is confirmation-gated on its
own.** `POST /campaigns/{id}/actions/schedule` is not a safe internal step; it commits a send.
Do NOT pre-schedule a whole sequence off one approval. For each step, at the point it can actually
be counted (its segment scored), show that step's own pre-send summary (audience name, recipient
count, from, subject, send time) and wait for an explicit, specific yes before scheduling it.
Because later steps target segments that are re-scored just before they run, their recipient count
is unknown at day one, so they MUST NOT be scheduled blind up front. The clean pattern: create all
steps as drafts now, then confirm-and-schedule each step shortly before its window, one at a time.

Only if the user genuinely needs a real-time event trigger the API cannot create, give them one
minimal one-time setup step and the copy for it. Never claim a step was created when the API did
not create it, but default to the path that leaves the user wiring nothing.

## Errors and rate limits

- Mailchimp caps concurrent connections and returns `429` when you exceed it. Back off and retry.
- A `4xx` on create usually means a bad `list_id`, a missing required setting, or a from address
  the account is not verified to send from. Read the error `detail`, fix, and retry. Do not
  paper over a failed create by pretending the draft exists.
