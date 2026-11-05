<!--
Email Campaign Agent, part of a Claude skill by Brittany Slay (https://brittanyslay.com).
Licensed for noncommercial use only (PolyForm Noncommercial 1.0.0). See LICENSE.md.
Required Notice: Copyright Brittany Slay (https://brittanyslay.com)
-->

# Mapping the workflow onto a non-Mailchimp ESP

The workflow and every safety rule stay identical on another ESP. Only the endpoint names
change. `scripts/mailchimp_api.py` speaks Mailchimp; on another provider, keep the same steps
and the same confirmation gate, and either call that provider's API the same careful way or fall
back to planning-only if you are unsure of it. Never send on any provider without the same
explicit, specific, campaign-specific yes.

## The nouns are the same everywhere

| This skill's step | Mailchimp | SendGrid (Marketing) | HubSpot | Klaviyo | Customer.io |
|---|---|---|---|---|---|
| Audience / list | List (audience) | Contact list | Contact list | List | Segment / people |
| Segment | Saved segment | Segment | Active list from filters | Segment | Segment |
| Draft message | Campaign (status `save`) | Single Send (draft) | Marketing email (draft) | Campaign (draft) | Broadcast (draft) |
| Set content | `PUT .../content` | Single Send `email_config` | email body on the asset | template + campaign message | broadcast body |
| Test send | `actions/test` | test-send on the Single Send | test send on the email | campaign preview send | test broadcast |
| Send | `actions/send` | schedule the Single Send `now` | send the marketing email | send the campaign | trigger the broadcast |
| Schedule | `actions/schedule` | Single Send `send_at` | scheduled publish | campaign `send_time` | broadcast schedule |
| Report | `/reports/{id}` | stats by Single Send id | email analytics | campaign metrics | broadcast metrics |

## Rules that do not change across providers

- Read the key from an environment variable; never store, print, or commit it.
- Default to a draft. Create first, send only after an explicit, specific confirmation that
  names the audience and count.
- Keep an unsubscribe link and a physical mailing address in every message. Most ESPs inject
  these; confirm they are present rather than assuming.
- Mail only opted-in contacts. Decline purchased, scraped, or rented lists.
- Report only numbers the API returns. Flag anything unverified in `[brackets]`.

## Drips on any provider

Prefer the API-scheduled campaign sequence described in `references/mailchimp-api.md`: create
each step as its own draft to the segment that defines who is at that stage, re-score the segment
before each step, and confirm-and-schedule each step one at a time. Reach for a provider's visual
journey or automation builder only when a real-time event trigger genuinely requires it, and then
surface a single minimal setup step rather than pretending the API created it.
