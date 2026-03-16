# Daily Digest

A personal Telegram bot that fetches your Gmail newsletters, summarizes them with Claude AI, and lets you have a real conversation about what you read.

Built for people who subscribe to a lot of newsletters and want to stay informed without spending 45 minutes reading.

---

## What it does

You message the bot when you're ready to catch up. It fetches that day's newsletters from Gmail, lets you navigate through them section by section, and gives you AI-generated summaries of each one. Then you can ask follow-up questions in natural language — the bot remembers the context of what you just read.

---

## Flow

**1. Type "digest" → pick a date**

![Date picker](assets/date-picker.png)

**2. Pick a newsletter**

![Newsletter selection](assets/newsletter-selection.png)

**3. Pick a section**

![Section menu](assets/section-menu.png)

**4. Get an AI-generated summary with article titles, descriptions, and links**

![Section summary](assets/section-summary.png)

**5. Ask follow-up questions — Claude has the context of what you just read**

![Chat reply](assets/chat-reply.png)

**6. Iterate — ask for a different format, more detail, simpler explanation**

![Chat reformatted](assets/chat-reformatted.png)

---

## Stack

| Component | Tool |
|---|---|
| Email | Gmail API |
| Bot | python-telegram-bot |
| Summarization | Claude Haiku |
| Conversation | Claude Sonnet |
| State | SQLite |
| Scheduler | APScheduler |
| Hosting | Railway |

---

## Architecture

```
Gmail API
    │
    ▼
Email Parser          ← extracts sections, articles, and URLs
    │                    from plain-text newsletter format
    ▼
Claude Haiku          ← summarizes each section into
    │                    titled bullets with read-more links
    ▼
SQLite Cache          ← summaries stored per session,
    │                    no re-summarization on repeat views
    ▼
Telegram Bot          ← inline keyboard navigation
    │                    date picker → newsletter → section → summary
    ▼
Claude Sonnet         ← conversational follow-up,
                         full section context injected into prompt
```

---

## Technical highlights

**Email parser**
TLDR newsletters arrive as Gmail plain text with reference-style links (`[1] https://...` at the footer) and word-wrapped titles. The parser reconstructs article titles, resolves link references, and identifies sections from emoji patterns — all without an HTML DOM to work with.

**Summarization**
Each section is summarized independently using Claude Haiku. Summaries are cached in SQLite so navigating back to a section is instant. If Claude can't extract structured articles (e.g. a GitHub repos section), it falls back to summarizing the raw section text.

**Conversation**
When you ask a follow-up question, the full section summary is injected as context alongside the conversation history. Claude Sonnet responds in Telegram-compatible HTML — no markdown leaking through.

**Deployment**
Runs as a single always-on Python process on Railway. Gmail OAuth credentials are stored as base64-encoded environment variables and written to disk at boot. Auto-deploys on every GitHub push.

---

## Why I built this

I subscribe to 5 TLDR newsletters (AI, InfoSec, Marketing, Dev, and the main one). Reading all of them daily was taking too long, and most tools that claim to summarize email either miss structure, hallucinate links, or don't let you ask questions. I wanted something that fits my actual reading habit — quick scan, then go deep on whatever catches my eye.
