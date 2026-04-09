# Customer Service AI Chatbot for hireaxiom.com

## Overview

An AI-powered customer service chatbot embedded on hireaxiom.com that answers visitor questions about Axiom's products, pricing, and services. Powered by Claude API via a Cloudflare Worker backend, with a floating chat widget on the frontend.

## Architecture

```
Visitor Browser                  Cloudflare Worker              Anthropic API
+------------------+            +--------------------+         +-----------+
| Chat Widget      |  POST      | /api/chat          |  POST   | Claude    |
| (in index.html)  | --------> | - injects system   | ------> | Haiku 4.5 |
|                  | <-------- |   prompt + KB       | <------ |           |
| Bubble + Panel   |  JSON     | - rate limits       |  JSON   +-----------+
|                  |            | - CORS lock         |
| Handoff Form     |  POST     | /api/lead           |  POST
|                  | --------> | - validates input   | ------> FormSubmit
+------------------+            +--------------------+          (tyler@hireaxiom.com)
```

## Components

### 1. Frontend: Chat Widget (index.html)

Self-contained HTML + CSS + JS appended to the existing `index.html`.

**Chat Bubble:**
- Fixed position, bottom-right corner (24px offset)
- 60px circle with Axiom accent blue gradient (`--gradient-1`)
- Chat icon (SVG), subtle pulse animation
- Click toggles the chat panel open/closed

**Chat Panel:**
- 400px wide x 500px tall, slides up from bubble
- Dark theme matching site variables (`--bg-secondary`, `--border-subtle`, `--text-primary`)
- Header: "Chat with Axiom" + close (X) button
- Message area: scrollable conversation thread
  - Bot messages: left-aligned, darker background
  - User messages: right-aligned, accent blue background
  - Typing indicator: animated dots while awaiting response
- Input bar: text input + send button, Enter key to send, disabled while waiting
- Pre-loaded welcome message on open

**Mobile (< 480px):**
- Full-width panel, full viewport height minus safe areas
- Bubble remains accessible

**Handoff UI:**
- When bot triggers handoff, inline form appears in chat: name, email, question (pre-filled)
- Submit POSTs to `/api/lead`
- Shows booking link after submission

**Conversation limits:**
- Max 20 messages per conversation
- After limit, suggest booking a call or emailing

### 2. Backend: Cloudflare Worker

Single Worker with two endpoints.

**`POST /api/chat`**
- Accepts: `{ messages: [{ role: "user"|"assistant", content: "..." }] }`
- **Server-side validation:** Rejects message arrays longer than 20 pairs. Truncates individual messages to 1,000 characters. Returns `{ error: "too_many_messages" }` if exceeded.
- Constructs system prompt from:
  1. Role & personality instructions
  2. Static site content (pricing, features, FAQ, industries, guarantee)
  3. Custom Q&A pairs from bundled `knowledge-base.json`
  4. Handoff rules
- Calls Claude API (`claude-haiku-4-5-20251001`)
- Returns: `{ response: "..." }`
- **On API failure:** Returns `{ error: "unavailable" }`. Frontend displays: "Our chat is temporarily unavailable. Please email tyler@hireaxiom.com or book a call."
- Rate limit: 10 requests/minute per IP, enforced via Cloudflare Rate Limiting rules configured in `wrangler.toml`
- CORS: Configurable via `CORS_ORIGIN` env var. Production: `https://hireaxiom.com`. Dev: `http://localhost:*`. Configured per wrangler environment (`[env.dev]` / `[env.production]`).

**`POST /api/lead`**
- Accepts: `{ name: "...", email: "...", question: "..." }`
- Validates required fields
- Forwards to `tyler@hireaxiom.com` via FormSubmit (existing integration)
- Checks FormSubmit HTTP response status before confirming
- Returns: `{ success: true }` only on confirmed delivery, `{ error: "send_failed" }` otherwise
- Frontend shows fallback on failure: "Something went wrong. Please email tyler@hireaxiom.com directly."

**Security:**
- Anthropic API key stored as Cloudflare Worker secret
- Never exposed to frontend
- CORS origin configurable per environment

### 3. Knowledge Base (`knowledge-base.json`)

Simple JSON file bundled with the Worker.

```json
[
  {
    "question": "Can I try Axiom before committing?",
    "answer": "Yes! We offer a 30-day money-back guarantee. If Axiom doesn't save you meaningful time, return the hardware in original condition for a full refund."
  },
  {
    "question": "How long does setup take?",
    "answer": "1-2 weeks from order to delivery, including custom configuration for your industry and a hands-on onboarding call."
  }
]
```

- Editable by Tyler directly
- Changes require `wrangler deploy` to take effect
- Target capacity: 50-75 Q&A pairs within system prompt token limits
- Future improvement: store Q&A in Cloudflare KV to enable edits without redeployment

### 4. System Prompt Structure

```
You are Axiom's AI assistant on hireaxiom.com. You help visitors understand
what Axiom is, how it works, pricing, and whether it's right for them.

PERSONALITY:
- Friendly, concise, and confident
- Speak in plain English, no jargon
- Be helpful but not pushy

CORE INFORMATION:
[Extracted site content: pricing tiers, features, how it works, industries, guarantee, etc.]

CUSTOM Q&A:
[Injected from knowledge-base.json]

RULES:
- Only answer questions about Axiom and its services
- If asked about competitors, stay positive about Axiom without bashing others
- If you cannot answer confidently, respond with exactly [HANDOFF] at the start
  (frontend uses case-insensitive prefix match), then explain you'd like to
  connect them with the team
- Keep responses under 3 sentences when possible
- Never make up pricing, features, or promises not in your knowledge base
```

## File Structure

```
ai-mac-mini/
  index.html                    # Chat widget added here (HTML + CSS + JS)

axiom-chatbot-worker/           # New directory for Cloudflare Worker
  wrangler.toml                 # Worker config (name, routes, secrets)
  src/
    index.js                    # Worker entry point (both endpoints)
    system-prompt.js            # System prompt construction
    knowledge-base.json         # Custom Q&A pairs
  package.json                  # Dependencies (none expected beyond wrangler)
```

## Deployment

1. **Frontend:** Push updated `index.html` to GitHub -> GitHub Pages deploys automatically
2. **Backend:** `npx wrangler deploy` from `axiom-chatbot-worker/` directory
3. **Secrets:** `npx wrangler secret put ANTHROPIC_API_KEY`
4. **Custom domain:** Configure Worker route for `api.hireaxiom.com` or use default `*.workers.dev` URL

## Cost Estimate

- **Claude Haiku 4.5:** ~$0.001 per conversation (avg 5 exchanges)
- **Cloudflare Worker:** Free tier covers 100K requests/day
- **Estimated monthly cost at 100 conversations/day:** ~$3/month in API usage
