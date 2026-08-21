# Email Campaign Agent

Take an email campaign from idea to scheduled send without hand-writing a single API call. A
[Claude Code](https://claude.ai/code) skill that plans your campaign, drafts the copy, runs a QA
pass, creates it as a **draft** in Mailchimp (or another ESP), confirms with you, and only then
schedules or sends. You bring your own API key at runtime. It is never stored, printed, or
committed.

![The email-campaign-agent flow: intake, plan, draft copy, QA pass, create draft, then confirm before send. On the left, a chat where you approve while the agent reports the work it did, segment scored on the API, copy drafted, QA passed. On the right, a pre-send check that needs your yes, with a reminder it goes to real people and a note that your API key is never stored or logged.](docs/hero.png)

## What it does

- **Plans and drafts the whole campaign.** Goal, audience and segment logic, the send structure
  (one-off, newsletter, or a multi-step drip), subject lines, preview text, and body copy.
- **QA before it ships.** Checks links, UTM tags, merge-tag fallbacks, unsubscribe and address,
  alt text, and plain-text, against a pre-send checklist.
- **Creates drafts, not surprises.** Everything starts as a draft in your ESP. It never sends or
  schedules without an explicit, specific yes, and it offers a test send to your own inbox first.
- **Reads back performance.** After a send, pulls the handful of numbers that matter for the
  goal: opens, clicks, unsubscribes, top links. It reports only what the API returns and never
  invents a result.
- **Works with no key too.** In planning-only mode it drafts everything as text and files and
  tells you exactly what to paste, with nothing connected.

## Install

Clone into your skills directory:

```bash
# user-level (available in every project)
git clone https://github.com/brittanyslay/email-campaign-agent.git ~/.claude/skills/email-campaign-agent

# or per-project
git clone https://github.com/brittanyslay/email-campaign-agent.git /path/to/project/.claude/skills/email-campaign-agent
```

Then just ask: *"Plan an email campaign to re-engage churned trials and set it up in Mailchimp as
a draft."* Claude picks the skill up from there.

## Bring your own API key (and keep it safe)

This skill ships with no key and stores none. You supply your own at runtime through an
environment variable, so it stays out of the chat transcript:

```bash
export MAILCHIMP_API_KEY="your-key-here-us21"
```

- The key is never written to a file, printed, or committed. In examples it is always masked.
- A read-only or limited key is safest for a first run or for pulling reports.
- No key set means planning-only mode. It drafts the campaign and copy and connects to nothing.
- A key is a live credential to your audience. If it ever lands in a shared log or screenshot,
  rotate it in Mailchimp right away.

It never sends without your explicit confirmation, defaults to drafts, and refuses purchased or
non-consented lists.

## What's inside

| File | What it's for |
|---|---|
| `SKILL.md` | The workflow Claude follows: intake, plan, draft, QA, draft-create, confirm, schedule, report. |
| `references/mailchimp-api.md` | The Mailchimp Marketing API v3 endpoint map, auth, and request bodies. |
| `references/qa-checklist.md` | The pre-send QA checklist. |
| `examples/example-run.md` | One campaign walked end to end, prompt to scheduled send. |

## Example prompts

Natural ways to reach for this skill once it's installed:

- `plan an email campaign to re-engage churned trial accounts`
- `draft a 3-step onboarding drip for new signups`
- `write our monthly newsletter and QA it before it goes out`
- `set up a Mailchimp campaign to this segment as a draft, don't send`
- `build a segment of accounts that opened but didn't click last time`
- `schedule this campaign for Tuesday 9am ET after I approve it`
- `send a test of this campaign to my inbox first`
- `read back how last week's campaign performed`
- `plan this campaign, I'm not giving you a key`
- `map this drip onto SendGrid instead of Mailchimp`

## License

Noncommercial use only (PolyForm Noncommercial 1.0.0). Commercial use requires a license from
Brittany Slay. See [LICENSE.md](LICENSE.md). Use, adapt, and share it for noncommercial purposes
with attribution intact.

---

Built by [Brittany Slay](https://brittanyslay.com), a B2B marketing leader who builds AI-native tools. More free Claude skills at [brittanyslay.com/skills](https://brittanyslay.com/skills). Want an AI-native marketing build done for real? [Get in touch](https://brittanyslay.com/#contact).
