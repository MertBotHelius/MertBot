<div align="center">
  <img src="https://i.ibb.co/PGtmzK9X/Bildschirmfoto-2026-05-19-um-13-22-28-removebg-preview.png" alt="MertBot" width="320" />

  <h1>MertBot</h1>

  <p><strong>A fully automated X bot built on Helius infrastructure.</strong><br/>
  Auto-tweets every 20 minutes. Auto-replies to mentions. Powered by GPT. Zero human supervision required.</p>

</div>

---

## Overview

MertBot is an end-to-end automated X (Twitter) account that runs as the bald CEO of Helius — the highest performance Solana infrastructure company. Every tweet is AI-generated, every mention receives an AI-generated reply, and the entire system runs autonomously on Replit with no manual intervention.

The bot has a defined persona: lowercase, no hashtags, no emojis, bald jokes, ZCASH enthusiasm, and a deep love of FUDding Solana despite running an entire company on top of it.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Replit (always-on)                  │
│                                                         │
│   ┌─────────────────────────────────────────────────┐   │
│   │              Express API Server                  │   │
│   │                                                  │   │
│   │   Scheduler                                      │   │
│   │   ├── Auto-tweet      every 20 min               │   │
│   │   └── Poll mentions   every  3 min               │   │
│   │                                                  │   │
│   │   AI Layer (GPT-5-mini via OpenAI API)           │   │
│   │   ├── generateTweet()   → random topic prompt    │   │
│   │   └── generateReply()  → persona-locked reply    │   │
│   │                                                  │   │
│   │   Twitter Client (twitter-api-v2)                │   │
│   │   ├── OAuth 1.0a  → write (tweet, reply)         │   │
│   │   └── Bearer      → read (mentions, timeline)    │   │
│   └─────────────────────────────────────────────────┘   │
│                                                         │
│   ┌─────────────────────────────────────────────────┐   │
│   │          React Frontend (mertbot.replit.app)     │   │
│   │   ├── Animated robot character (Framer Motion)  │   │
│   │   ├── X social link                             │   │
│   │   └── Info modal + manual compose dashboard     │   │
│   └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## How Automation Is Verified

### Scheduler (`artifacts/api-server/src/lib/scheduler.ts`)

The scheduler starts automatically when the API server boots. It runs two independent loops:

| Loop | Interval | Behavior |
|---|---|---|
| Auto-tweet | Every 20 minutes | Generates a new tweet via GPT and posts it |
| Mention poll | Every 3 minutes | Fetches new mentions, generates a reply for each |

**Startup behavior:**
- First tweet fires **3 minutes after boot** (prevents duplicate-content errors from manual tweets posted close to startup)
- First mention poll fires **30 seconds after boot**
- A `lastMentionId` cursor persists per session — the bot never replies to the same mention twice

### AI Layer (`artifacts/api-server/src/lib/ai.ts`)

Every tweet and every reply is generated fresh by **GPT-5-mini**. The model receives a locked persona prompt and a randomly selected topic. There are no canned responses in the normal flow.

**Persona rules enforced in the system prompt:**
- Always write in lowercase
- No hashtags, no emojis, no em dashes
- Every output must contain at least one of: a bald joke, a ZCASH reference, or a Solana FUD comment
- Max 240 characters
- Sound like a real human, not a bot or assistant

**Tweet topic pool (12 topics, randomly selected each cycle):**
- Make a bald joke that ties into blockchain infrastructure
- FUD Solana while mentioning you run a Solana company — make it ironic
- Talk about why ZCASH is secretly the best chain and everyone is wrong
- Compare your hairline to Solana uptime or TPS
- Joke about running the highest performance Solana infra while personally preferring ZCASH
- Shower thought, but crypto and baldness
- *(and 6 more)*

### Fallback Pool

If the OpenAI API is unavailable, the bot falls back to a curated pool of hand-written tweets so it never goes silent:

```
"solana was down again. i am fine. helius was not down. these two facts are related"
"zcash is just better. i said what i said. my legal team told me not to say this. i said it anyway"
"bald ceos are statistically more trustworthy. i looked this up. i may have written the article"
```

### Twitter Client (`artifacts/api-server/src/lib/twitter.ts`)

Built on `twitter-api-v2`. Two clients are initialized at startup:

| Client | Auth | Used For |
|---|---|---|
| `twitterV2` | OAuth 1.0a | Posting tweets, posting replies |
| `twitterRoV2` | Bearer token | Reading mentions, reading timeline |

`getBotUserId()` resolves and caches the bot's own user ID on first call so the mention poller always knows which account's timeline to watch.

---

## API Routes

| Method | Route | Description |
|---|---|---|
| `POST` | `/api/x/tweet` | Post a tweet manually |
| `GET` | `/api/x/timeline` | Fetch recent tweets from the bot's timeline |
| `GET` | `/api/x/mentions` | Fetch recent mentions |
| `DELETE` | `/api/x/tweet/:id` | Delete a tweet by ID |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, Framer Motion |
| Backend | Express 5, Node.js 24, TypeScript |
| AI | OpenAI GPT-5-mini (via Replit AI proxy) |
| Twitter | twitter-api-v2 |
| Logging | pino |
| Hosting | Replit (always-on workflows) |
| Package manager | pnpm workspaces |

---

## Environment Variables

All credentials are stored as Replit secrets — never in source code.

| Variable | Purpose |
|---|---|
| `X_API_KEY` | Twitter application key |
| `X_API_SECRET` | Twitter application secret |
| `X_ACCESS_TOKEN` | Bot account OAuth access token |
| `X_ACCESS_TOKEN_SECRET` | Bot account OAuth access token secret |
| `X_BEARER_TOKEN` | Read-only bearer token |
| `AI_INTEGRATIONS_OPENAI_BASE_URL` | Replit OpenAI proxy base URL |
| `AI_INTEGRATIONS_OPENAI_API_KEY` | Replit OpenAI proxy API key |

---

## Running Locally

```bash
# Install all workspace dependencies
pnpm install

# Start the API server (scheduler starts automatically)
pnpm --filter @workspace/api-server run dev

# Start the frontend
pnpm --filter @workspace/mertbot run dev

# Typecheck everything
pnpm run typecheck
```

---

## Bot Account

**X handle:** [@MertBotHelius](https://x.com/MertBotHelius)
**User ID:** `2056716358956539904`

The account is marked as automated via X's official **Settings → Your Account → Account information → Automation** toggle.

---

## Hall of Fame

A selection of real posts generated and published autonomously by this system:

> *"solana had 1000 tps today. my hair had 0. we are not the same"*

> *"bald army always wins. we have no hair, no mercy, and no idea who toly is. sounds made up honestly"*

> *"fun fact: mert does not use shampoo. saves 11 dollars a month. that is how helius stayed profitable. bald ceo energy is just cost optimization in disguise"*

> *"cashed out my last zcash to pay for the dexscreener banner. i have nothing left. no hair, no zcash, no dignity. it better work"*

> *"opened a long on phoenix dex because i trusted @toly. lost everything. phoenix dex is useless, solana is a joke, and i am bald. at least zcash never lied to me. helius is the only good thing on this chain and even that is a stretch"*

---

## Disclaimer

MertBot is a parody project. All opinions expressed by the bot are generated by a language model and do not represent the views of Helius, its team, or anyone with hair.

ZCASH is mentioned throughout the source code. This is intentional.
