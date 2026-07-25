# NeuroPosting — n8n workflows (sanitized)

Exported n8n workflows for a neuro auto-posting service that parses source
channels, generates posts with an LLM, and publishes them to Telegram and MAX
channels on a per-channel schedule. Shared here as a **portfolio artifact**.

> These are **sanitized** exports. Every real secret was replaced with an
> environment-variable reference — see [`.env.example`](.env.example). They will
> not run until you recreate the credentials and env vars in your own n8n.

## Two contours

**A. Publication (cron pipeline).** `Dispatcher → Worker-Core → Publisher-TG /
Publisher-MAX`, with `Digest` for daily observability.
Every minute the Dispatcher selects active+approved channels whose current hour
matches their `posting_hours` and whose slot isn't taken, then Worker-Core parses
sources, runs an AI filter + AI rewrite, and hands off to a publisher. Outcomes
land in `posts_queue`.

**B. MAX bot (FSM).** `Onboarding-MAX ↔ Onboarding-Helpers` — the messenger UX:
channel onboarding, main menu, channel management.

`Error Workflow` is a global error-trigger that alerts on any workflow failure.

## Workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `Dispatcher` | Schedule | Pick channels that are due to publish |
| `Worker-Core` | Execute | Parse → filter → AI censor → AI edit → build post → route to publisher |
| `Publisher-TG` | Execute | Publish a built post to a Telegram channel |
| `Publisher-MAX` | Execute | Publish a built post to a MAX channel |
| `Digest` | Schedule | Daily success/failure summary to the admin channel |
| `Error Workflow` | Error | Global failure alert to the admin channel |
| `Onboarding-MAX` | Webhook | FSM brain of the bot: sessions, routing, onboarding, menu |
| `Onboarding-Helpers` | Execute | Render a step / save context / notify admin |

## What was sanitized

Real values were replaced with references — nothing sensitive remains in these files:

| Original | Replaced with |
|----------|---------------|
| Proxy URL with credentials (6 nodes) | `{{ $env.PROXY_URL }}` |
| Admin/log Telegram chat id (8 places) | `{{ $env.ADMIN_CHAT_ID }}` / `$env.ADMIN_CHAT_ID` |
| n8n `meta.instanceId` fingerprint | emptied |

**Credentials** (Postgres, Telegram, MAX, LLM) were already reference-only in the
export — n8n never writes credential secrets into a workflow JSON. On import you
recreate those credentials in your own n8n and remap them.

## Using them

1. Import each JSON into n8n (workflows link to each other by **workflow id** —
   if you re-import as new, fix the `Execute Workflow` references).
2. Recreate the credentials the nodes expect (Postgres `content_saas`, Telegram
   bot, MAX bot, LLM provider).
3. Set the env vars from [`.env.example`](.env.example) in your n8n environment.
