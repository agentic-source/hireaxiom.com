# Customer Service AI Chatbot Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a floating AI chatbot to hireaxiom.com that answers visitor questions using Claude Haiku via a Cloudflare Worker backend.

**Architecture:** A self-contained chat widget (HTML/CSS/JS) embedded in the existing `index.html` communicates with a Cloudflare Worker that proxies requests to the Anthropic Claude API. The Worker bundles a knowledge base JSON file into the system prompt and handles lead capture via FormSubmit.

**Tech Stack:** Vanilla HTML/CSS/JS (frontend), Cloudflare Workers + Wrangler (backend), Anthropic Claude API (AI), FormSubmit (lead emails)

**Spec:** `docs/superpowers/specs/2026-04-09-customer-service-chatbot-design.md`

---

## File Structure

```
ai-mac-mini/
  index.html                          # Modify: append chat widget HTML + CSS + JS

axiom-chatbot-worker/                 # Create: new directory (sibling to ai-mac-mini)
  package.json                        # Create: wrangler dev dependency
  wrangler.toml                       # Create: Worker config, environments, rate limiting
  src/
    index.js                          # Create: Worker entry point, routing, CORS, validation
    system-prompt.js                  # Create: builds system prompt from site content + KB
    knowledge-base.json               # Create: custom Q&A pairs
```

---

## Task 1: Scaffold the Cloudflare Worker Project

**Files:**
- Create: `axiom-chatbot-worker/package.json`
- Create: `axiom-chatbot-worker/wrangler.toml`

- [ ] **Step 1: Create project directory and package.json**

```bash
mkdir -p /Users/tylermeeks/axiom-chatbot-worker/src
```

Write `axiom-chatbot-worker/package.json`:

```json
{
  "name": "axiom-chatbot-worker",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "wrangler dev",
    "deploy": "wrangler deploy"
  },
  "devDependencies": {
    "wrangler": "^3.0.0"
  }
}
```

- [ ] **Step 2: Create wrangler.toml with dev and production environments**

Write `axiom-chatbot-worker/wrangler.toml`:

```toml
name = "axiom-chatbot"
main = "src/index.js"
compatibility_date = "2024-01-01"

[env.dev]
[env.dev.vars]
CORS_ORIGIN = "http://localhost:8000"

[env.production]
[env.production.vars]
CORS_ORIGIN = "https://hireaxiom.com"
routes = [
  { pattern = "api.hireaxiom.com/*", zone_name = "hireaxiom.com" }
]
```

Note: `ANTHROPIC_API_KEY` will be set as a secret via `wrangler secret put`, not in this file.

Note on rate limiting: The Worker uses origin validation to block unauthorized callers. For per-IP rate limiting, Cloudflare's native Rate Limiting rules (configured in the Cloudflare dashboard under Security > WAF > Rate limiting rules) are recommended but require a paid zone plan. The `max_tokens: 300` cap on responses and origin validation provide cost protection at the free tier.

- [ ] **Step 3: Install dependencies**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker && npm install
```

- [ ] **Step 4: Create .gitignore**

Write `axiom-chatbot-worker/.gitignore`:

```
node_modules/
.wrangler/
.dev.vars
```

- [ ] **Step 5: Commit scaffold**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
git init
git add package.json wrangler.toml package-lock.json .gitignore
git commit -m "chore: scaffold Cloudflare Worker project for chatbot"
```

---

## Task 2: Build the Knowledge Base and System Prompt

**Files:**
- Create: `axiom-chatbot-worker/src/knowledge-base.json`
- Create: `axiom-chatbot-worker/src/system-prompt.js`

- [ ] **Step 1: Create knowledge-base.json with initial Q&A pairs**

Write `axiom-chatbot-worker/src/knowledge-base.json`. Seed it with Q&A pairs extracted from the existing FAQ section of `index.html`, plus additional useful entries:

```json
[
  {
    "question": "Do I need to be technical?",
    "answer": "No. You talk to it like you'd talk to an employee. Plain English, no coding, no terminal commands. We handle all the technical setup and maintenance."
  },
  {
    "question": "What industries do you serve?",
    "answer": "We serve businesses across healthcare staffing, legal, real estate, insurance, accounting, consulting, medical practice, and recruiting — plus individuals like busy professionals, parents, students, and content creators. Each Axiom is custom-configured for your specific needs."
  },
  {
    "question": "What happens if something breaks?",
    "answer": "Remote support is included in all plans. We monitor your system proactively and push updates and fixes — usually before you even notice an issue."
  },
  {
    "question": "What about AI costs?",
    "answer": "Included in your monthly plan. No surprise bills, no usage caps for normal business operations. You pay one predictable monthly price and we handle the rest."
  },
  {
    "question": "Can I add more automations later?",
    "answer": "Yes. Additional workflows are $500–$2,000 each depending on complexity. Each workflow handles one defined automation (e.g., daily lead scraping, contract review, weekly reporting). Most clients add 2–3 new workflows within the first quarter."
  },
  {
    "question": "Is my data secure?",
    "answer": "Everything runs locally on YOUR machine. Your data never touches our servers. The Axiom system sits on your desk, behind your firewall, under your control."
  },
  {
    "question": "How long does setup take?",
    "answer": "1–2 weeks from order to delivery, including custom configuration for your industry, installation of your specific automations, and a hands-on onboarding call."
  },
  {
    "question": "Can I try Axiom before committing?",
    "answer": "Yes! We offer a 30-day money-back guarantee. If Axiom doesn't save you meaningful time in the first 30 days, return the hardware in original condition for a full refund — no questions asked."
  },
  {
    "question": "What are the pricing tiers?",
    "answer": "We have four plans: Personal ($1,999 one-time + $299/mo, 2 workflows), Business ($2,999 + $499/mo, 6 workflows), Business Pro ($4,999 + $999/mo, 12 workflows + weekly support call), and Enterprise ($9,999 + $1,999/mo, unlimited workflows + dedicated account manager). All plans include hardware, setup, and API usage."
  },
  {
    "question": "What is Axiom exactly?",
    "answer": "Axiom is an always-on AI system that runs on a dedicated machine at your desk. It automates repetitive business tasks — things like lead scraping, contract review, report generation, and more. It's custom-configured for your industry and workflows, and we handle all setup and maintenance."
  }
]
```

- [ ] **Step 2: Create system-prompt.js**

Write `axiom-chatbot-worker/src/system-prompt.js`:

```javascript
import knowledgeBase from './knowledge-base.json';

const SITE_CONTENT = `
ABOUT AXIOM:
Axiom is an AI-powered system that runs on a dedicated Mac Mini at your desk. It automates repetitive business tasks 24/7. Custom-configured for your industry, fully managed — so your team can stop doing repetitive work manually.

HOW IT WORKS:
1. Tell us what you need — we'll show you what Axiom can automate
2. We configure it — custom automations tailored to your workflows, pre-loaded and pre-tested
3. Plug in & go — arrives at your door, plug it in, connect to Wi-Fi, live onboarding call

WHAT YOU GET:
- Dedicated always-on system
- Advanced AI engine with your business context pre-loaded
- Custom automations built for YOUR specific workflows (not templates)
- Ongoing support, monitoring, and continuous improvement

PRICING:
- Personal "The Assistant": $1,999 one-time + $299/month (2 personal workflows, email support 48hr)
- Business "The Associate": $2,999 one-time + $499/month (6 custom workflows, same-day email support)
- Business Pro "The Executive" (Most Popular): $4,999 one-time + $999/month (~$33/day) (12 custom workflows, industry-specific agent suite, weekly support call, remote monitoring)
- Enterprise "The Command Center": $9,999 one-time + $1,999/month (unlimited workflows, dedicated account manager, priority support & SLA, quarterly strategy)
- All plans include hardware, setup, configuration, and API usage
- Additional workflows: $500–$2,000 each
- Save 2 months with annual billing

GUARANTEE:
30-day money-back guarantee. Return hardware in original condition for a full refund, no questions asked.

INDUSTRIES SERVED:
Business: Healthcare Staffing, Legal, Real Estate, Insurance, Accounting, Consulting, Medical Practice, Recruiting
Personal: Busy Professionals, Parents/Home Managers, Students, Content Creators

DATA & SECURITY:
Everything runs locally on the customer's machine. Data never touches Axiom's servers. The system sits on their desk, behind their firewall, under their control.

FOUNDER STORY:
Tyler Meeks runs a healthcare staffing company. He was drowning in repetitive work — scraping job boards, reviewing contracts, chasing data every morning. He built an AI system to automate it all. When other business owners saw what it could do, they wanted one too. That's Axiom.

CONTACT:
- Book a 15-minute demo at hireaxiom.com (scroll to bottom)
- Email: hello@hireaxiom.com
- Located in San Juan, PR
`;

export function buildSystemPrompt() {
  const qaPairs = knowledgeBase
    .map(qa => `Q: ${qa.question}\nA: ${qa.answer}`)
    .join('\n\n');

  return `You are Axiom's AI assistant on hireaxiom.com. You help visitors understand what Axiom is, how it works, pricing, and whether it's right for them.

PERSONALITY:
- Friendly, concise, and confident
- Speak in plain English, no jargon
- Be helpful but not pushy
- Keep responses under 3 sentences when possible

CORE INFORMATION:
${SITE_CONTENT}

CUSTOM Q&A:
${qaPairs}

RULES:
- Only answer questions about Axiom and its services
- If asked about competitors, stay positive about Axiom without bashing others
- If you cannot answer confidently, respond with exactly [HANDOFF] at the start of your message, then explain that you'd like to connect them with the team for a better answer
- Never make up pricing, features, or promises not in your knowledge base
- When relevant, suggest they book a 15-minute demo or email hello@hireaxiom.com
- Do not use markdown formatting in responses — plain text only`;
}
```

- [ ] **Step 3: Commit knowledge base and system prompt**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
git add src/knowledge-base.json src/system-prompt.js
git commit -m "feat: add knowledge base and system prompt builder"
```

---

## Task 3: Build the Worker Entry Point

**Files:**
- Create: `axiom-chatbot-worker/src/index.js`

- [ ] **Step 1: Write the Worker with /api/chat and /api/lead endpoints**

Write `axiom-chatbot-worker/src/index.js`:

```javascript
import { buildSystemPrompt } from './system-prompt.js';

const MAX_MESSAGES = 40; // 20 user + 20 assistant
const MAX_MESSAGE_LENGTH = 1000;

function isAllowedOrigin(origin, allowedOrigin) {
  if (!origin) return false;
  // Support wildcard localhost for dev
  if (allowedOrigin.includes('localhost') && origin.includes('localhost')) return true;
  return origin === allowedOrigin;
}

function corsHeaders(origin, allowedOrigin) {
  const resolvedOrigin = isAllowedOrigin(origin, allowedOrigin) ? origin : allowedOrigin;
  return {
    'Access-Control-Allow-Origin': resolvedOrigin,
    'Access-Control-Allow-Methods': 'POST, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type',
  };
}

async function handleChat(request, env) {
  const body = await request.json();
  const messages = body.messages;

  if (!Array.isArray(messages) || messages.length === 0) {
    return Response.json({ error: 'invalid_messages' }, { status: 400 });
  }

  if (messages.length > MAX_MESSAGES) {
    return Response.json({ error: 'too_many_messages' }, { status: 400 });
  }

  // Truncate and validate messages
  const sanitized = messages.map(m => ({
    role: m.role === 'assistant' ? 'assistant' : 'user',
    content: typeof m.content === 'string'
      ? m.content.slice(0, MAX_MESSAGE_LENGTH)
      : '',
  }));

  const systemPrompt = buildSystemPrompt();

  try {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5-20251001',
        max_tokens: 300,
        system: systemPrompt,
        messages: sanitized,
      }),
    });

    if (!response.ok) {
      console.error('Anthropic API error:', response.status);
      return Response.json({ error: 'unavailable' }, { status: 502 });
    }

    const data = await response.json();
    const text = data.content?.[0]?.text || '';
    return Response.json({ response: text });
  } catch (err) {
    console.error('Anthropic API fetch error:', err);
    return Response.json({ error: 'unavailable' }, { status: 502 });
  }
}

async function handleLead(request, env) {
  const body = await request.json();
  const { name, email, question } = body;

  if (!name || !email || !question) {
    return Response.json({ error: 'missing_fields' }, { status: 400 });
  }

  try {
    const formData = new FormData();
    formData.append('name', name);
    formData.append('email', email);
    formData.append('message', `[Chatbot Lead]\n\nQuestion: ${question}`);
    formData.append('_subject', 'New Chatbot Lead from hireaxiom.com');
    formData.append('_captcha', 'false');
    formData.append('_template', 'table');

    const response = await fetch('https://formsubmit.co/ajax/tyler@hireaxiom.com', {
      method: 'POST',
      body: formData,
    });

    if (!response.ok) {
      console.error('FormSubmit error:', response.status);
      return Response.json({ error: 'send_failed' }, { status: 502 });
    }

    return Response.json({ success: true });
  } catch (err) {
    console.error('FormSubmit fetch error:', err);
    return Response.json({ error: 'send_failed' }, { status: 502 });
  }
}

export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const origin = request.headers.get('Origin') || '';
    const allowedOrigin = env.CORS_ORIGIN || 'https://hireaxiom.com';
    const headers = corsHeaders(origin, allowedOrigin);

    // Handle CORS preflight
    if (request.method === 'OPTIONS') {
      return new Response(null, { status: 204, headers });
    }

    // Reject requests from disallowed origins
    if (request.method === 'POST' && !isAllowedOrigin(origin, allowedOrigin)) {
      return new Response(null, { status: 403 });
    }

    let response;

    if (url.pathname === '/api/chat' && request.method === 'POST') {
      response = await handleChat(request, env);
    } else if (url.pathname === '/api/lead' && request.method === 'POST') {
      response = await handleLead(request, env);
    } else {
      response = Response.json({ error: 'not_found' }, { status: 404 });
    }

    // Apply CORS headers to response
    const newHeaders = new Headers(response.headers);
    for (const [key, value] of Object.entries(headers)) {
      newHeaders.set(key, value);
    }
    return new Response(response.body, {
      status: response.status,
      headers: newHeaders,
    });
  },
};
```

- [ ] **Step 2: Test locally with wrangler dev**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
npx wrangler dev --env dev
```

In another terminal, test:

```bash
curl -X POST http://localhost:8787/api/chat \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"What is Axiom?"}]}'
```

Expected: JSON response with a helpful answer about Axiom (requires `ANTHROPIC_API_KEY` secret — set via `npx wrangler secret put ANTHROPIC_API_KEY --env dev` first).

- [ ] **Step 3: Commit Worker entry point**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
git add src/index.js
git commit -m "feat: add Worker entry point with chat and lead endpoints"
```

---

## Task 4: Build the Chat Widget Frontend

**Files:**
- Modify: `ai-mac-mini/index.html` (append before closing `</body>` tag, after the existing `<script>` block at line ~2923)

- [ ] **Step 1: Add chat widget CSS to the existing `<style>` block**

Insert the following CSS before the closing `</style>` tag in `index.html` (around line 1700, after the existing styles):

```css
/* ========== CHAT WIDGET ========== */
.chat-bubble {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: var(--gradient-1);
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 24px rgba(0, 180, 255, 0.3);
  z-index: 9999;
  transition: transform 0.2s, box-shadow 0.2s;
  animation: chat-pulse 3s ease-in-out infinite;
}

.chat-bubble:hover {
  transform: scale(1.08);
  box-shadow: 0 6px 32px rgba(0, 180, 255, 0.45);
}

.chat-bubble svg {
  width: 28px;
  height: 28px;
  fill: white;
}

.chat-bubble.open svg.icon-chat { display: none; }
.chat-bubble:not(.open) svg.icon-close { display: none; }

@keyframes chat-pulse {
  0%, 100% { box-shadow: 0 4px 24px rgba(0, 180, 255, 0.3); }
  50% { box-shadow: 0 4px 32px rgba(0, 180, 255, 0.5); }
}

.chat-panel {
  position: fixed;
  bottom: 100px;
  right: 24px;
  width: 400px;
  height: 500px;
  background: var(--bg-secondary);
  border: 1px solid var(--border-subtle);
  border-radius: var(--radius);
  display: none;
  flex-direction: column;
  z-index: 9998;
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.5);
  overflow: hidden;
}

.chat-panel.open {
  display: flex;
}

.chat-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid var(--border-subtle);
  background: var(--bg-card);
}

.chat-header-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-primary);
}

.chat-header-close {
  background: none;
  border: none;
  color: var(--text-secondary);
  cursor: pointer;
  font-size: 20px;
  padding: 0;
  line-height: 1;
}

.chat-header-close:hover {
  color: var(--text-primary);
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.chat-msg {
  max-width: 85%;
  padding: 10px 14px;
  border-radius: 12px;
  font-size: 14px;
  line-height: 1.5;
  word-wrap: break-word;
}

.chat-msg.bot {
  align-self: flex-start;
  background: var(--bg-card);
  color: var(--text-primary);
  border: 1px solid var(--border-subtle);
}

.chat-msg.user {
  align-self: flex-end;
  background: var(--accent);
  color: white;
}

.chat-msg a {
  color: var(--accent);
  text-decoration: underline;
}

.chat-msg.user a {
  color: white;
}

.chat-typing {
  align-self: flex-start;
  padding: 10px 14px;
  background: var(--bg-card);
  border: 1px solid var(--border-subtle);
  border-radius: 12px;
  display: none;
}

.chat-typing.visible {
  display: flex;
  gap: 4px;
  align-items: center;
}

.chat-typing-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--text-secondary);
  animation: chat-typing-bounce 1.4s ease-in-out infinite;
}

.chat-typing-dot:nth-child(2) { animation-delay: 0.2s; }
.chat-typing-dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes chat-typing-bounce {
  0%, 60%, 100% { transform: translateY(0); }
  30% { transform: translateY(-4px); }
}

.chat-input-bar {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  border-top: 1px solid var(--border-subtle);
  background: var(--bg-card);
}

.chat-input {
  flex: 1;
  padding: 10px 14px;
  border-radius: var(--radius-sm);
  border: 1px solid var(--border-subtle);
  background: var(--bg-primary);
  color: var(--text-primary);
  font-family: var(--font);
  font-size: 14px;
  outline: none;
}

.chat-input:focus {
  border-color: var(--border-glow);
}

.chat-input::placeholder {
  color: var(--text-muted);
}

.chat-send {
  padding: 10px 16px;
  border-radius: var(--radius-sm);
  background: var(--gradient-1);
  color: white;
  border: none;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  font-family: var(--font);
  transition: opacity 0.2s;
}

.chat-send:hover { opacity: 0.9; }
.chat-send:disabled { opacity: 0.5; cursor: not-allowed; }

/* Handoff form inside chat */
.chat-handoff-form {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 12px;
  background: var(--bg-card);
  border: 1px solid var(--border-subtle);
  border-radius: 12px;
  margin-top: 4px;
}

.chat-handoff-form input {
  padding: 8px 12px;
  border-radius: var(--radius-sm);
  border: 1px solid var(--border-subtle);
  background: var(--bg-primary);
  color: var(--text-primary);
  font-family: var(--font);
  font-size: 13px;
  outline: none;
}

.chat-handoff-form input:focus {
  border-color: var(--border-glow);
}

.chat-handoff-form button {
  padding: 8px 16px;
  border-radius: var(--radius-sm);
  background: var(--gradient-1);
  color: white;
  border: none;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  font-family: var(--font);
}

/* Mobile */
@media (max-width: 480px) {
  .chat-panel {
    width: 100%;
    height: calc(100vh - 120px);
    right: 0;
    bottom: 80px;
    border-radius: var(--radius) var(--radius) 0 0;
  }
}
```

- [ ] **Step 2: Add chat widget HTML before closing `</body>` tag**

Insert before `</body>` (line 2924 of current `index.html`):

```html
<!-- Chat Widget -->
<button class="chat-bubble" id="chat-bubble" aria-label="Open chat">
  <svg class="icon-chat" viewBox="0 0 24 24"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
  <svg class="icon-close" viewBox="0 0 24 24"><path d="M18 6L6 18M6 6l12 12" stroke="white" stroke-width="2" stroke-linecap="round" fill="none"/></svg>
</button>

<div class="chat-panel" id="chat-panel">
  <div class="chat-header">
    <span class="chat-header-title">Chat with Axiom</span>
    <button class="chat-header-close" id="chat-close" aria-label="Close chat">&times;</button>
  </div>
  <div class="chat-messages" id="chat-messages">
    <div class="chat-msg bot">Hi! I'm Axiom's AI assistant. Ask me anything about how Axiom works, pricing, or what it can do for your business.</div>
    <div class="chat-typing" id="chat-typing">
      <div class="chat-typing-dot"></div>
      <div class="chat-typing-dot"></div>
      <div class="chat-typing-dot"></div>
    </div>
  </div>
  <div class="chat-input-bar">
    <input type="text" class="chat-input" id="chat-input" placeholder="Type a message..." autocomplete="off">
    <button class="chat-send" id="chat-send">Send</button>
  </div>
</div>
```

- [ ] **Step 3: Add chat widget JavaScript after the existing `<script>` block**

Insert a new `<script>` block before `</body>`, after the chat HTML:

```javascript
// ========== CHAT WIDGET ==========
(function() {
  const WORKER_URL = 'https://axiom-chatbot.YOUR_SUBDOMAIN.workers.dev';
  // TODO: Replace with actual Worker URL after first deploy
  const MAX_MESSAGES = 20;

  const bubble = document.getElementById('chat-bubble');
  const panel = document.getElementById('chat-panel');
  const closeBtn = document.getElementById('chat-close');
  const messagesEl = document.getElementById('chat-messages');
  const typingEl = document.getElementById('chat-typing');
  const inputEl = document.getElementById('chat-input');
  const sendBtn = document.getElementById('chat-send');

  let conversation = [];
  let userMessageCount = 0;
  let waiting = false;

  function toggleChat() {
    const isOpen = panel.classList.toggle('open');
    bubble.classList.toggle('open', isOpen);
    if (isOpen) inputEl.focus();
  }

  bubble.addEventListener('click', toggleChat);
  closeBtn.addEventListener('click', toggleChat);

  function addMessage(text, role) {
    const div = document.createElement('div');
    div.className = 'chat-msg ' + role;
    div.textContent = text;
    messagesEl.insertBefore(div, typingEl);
    messagesEl.scrollTop = messagesEl.scrollHeight;
  }

  function addHTML(html, role) {
    const div = document.createElement('div');
    div.className = 'chat-msg ' + role;
    div.innerHTML = html;
    messagesEl.insertBefore(div, typingEl);
    messagesEl.scrollTop = messagesEl.scrollHeight;
  }

  function showTyping(show) {
    typingEl.classList.toggle('visible', show);
    messagesEl.scrollTop = messagesEl.scrollHeight;
  }

  function setInputEnabled(enabled) {
    waiting = !enabled;
    inputEl.disabled = !enabled;
    sendBtn.disabled = !enabled;
  }

  function showHandoffForm(botMessage) {
    // Show the bot's message (without [HANDOFF] prefix)
    const cleanMessage = botMessage.trimStart().replace(/^\[handoff\]/i, '').trim();
    if (cleanMessage) addMessage(cleanMessage, 'bot');

    addMessage("I'd love to connect you with our team. Leave your info below and we'll get back to you:", 'bot');

    const form = document.createElement('div');
    form.className = 'chat-handoff-form';
    form.innerHTML = `
      <input type="text" placeholder="Your name" id="handoff-name" required>
      <input type="email" placeholder="Your email" id="handoff-email" required>
      <input type="text" placeholder="Your question" id="handoff-question" value="${conversation.length > 0 ? conversation[conversation.length - 1].content.replace(/"/g, '&quot;') : ''}">
      <button id="handoff-submit">Send to Team</button>
    `;
    messagesEl.insertBefore(form, typingEl);
    messagesEl.scrollTop = messagesEl.scrollHeight;

    form.querySelector('#handoff-submit').addEventListener('click', async () => {
      const name = form.querySelector('#handoff-name').value.trim();
      const email = form.querySelector('#handoff-email').value.trim();
      const question = form.querySelector('#handoff-question').value.trim();

      if (!name || !email || !question) {
        addMessage('Please fill in all fields.', 'bot');
        return;
      }

      try {
        const res = await fetch(WORKER_URL + '/api/lead', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ name, email, question }),
        });
        const data = await res.json();
        if (data.success) {
          form.remove();
          addHTML('Thanks! Our team will get back to you soon. You can also <a href="#contact">book a 15-minute demo</a> to talk with us directly.', 'bot');
        } else {
          addHTML('Something went wrong. Please email <a href="mailto:tyler@hireaxiom.com">tyler@hireaxiom.com</a> directly.', 'bot');
        }
      } catch {
        addHTML('Something went wrong. Please email <a href="mailto:tyler@hireaxiom.com">tyler@hireaxiom.com</a> directly.', 'bot');
      }
    });
  }

  async function sendMessage() {
    const text = inputEl.value.trim();
    if (!text || waiting) return;

    if (userMessageCount >= MAX_MESSAGES) {
      addHTML('We\'ve reached the conversation limit. For more help, <a href="#contact">book a 15-minute demo</a> or email <a href="mailto:hello@hireaxiom.com">hello@hireaxiom.com</a>.', 'bot');
      return;
    }

    addMessage(text, 'user');
    conversation.push({ role: 'user', content: text });
    userMessageCount++;
    inputEl.value = '';

    setInputEnabled(false);
    showTyping(true);

    try {
      const res = await fetch(WORKER_URL + '/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages: conversation }),
      });
      const data = await res.json();

      showTyping(false);

      if (data.error === 'unavailable') {
        addHTML('Our chat is temporarily unavailable. Please email <a href="mailto:tyler@hireaxiom.com">tyler@hireaxiom.com</a> or <a href="#contact">book a call</a>.', 'bot');
      } else if (data.error === 'rate_limited') {
        addMessage('You\'re sending messages too quickly. Please wait a moment and try again.', 'bot');
      } else if (data.error) {
        addHTML('Something went wrong. Please email <a href="mailto:tyler@hireaxiom.com">tyler@hireaxiom.com</a> directly.', 'bot');
      } else if (data.response && data.response.trimStart().toUpperCase().startsWith('[HANDOFF]')) {
        showHandoffForm(data.response);
        conversation.push({ role: 'assistant', content: data.response });
      } else {
        addMessage(data.response, 'bot');
        conversation.push({ role: 'assistant', content: data.response });
      }
    } catch {
      showTyping(false);
      addHTML('Our chat is temporarily unavailable. Please email <a href="mailto:tyler@hireaxiom.com">tyler@hireaxiom.com</a> or <a href="#contact">book a call</a>.', 'bot');
    }

    setInputEnabled(true);
    inputEl.focus();
  }

  sendBtn.addEventListener('click', sendMessage);
  inputEl.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') sendMessage();
  });
})();
```

- [ ] **Step 4: Commit the chat widget**

```bash
cd /Users/tylermeeks/ai-mac-mini
git add index.html
git commit -m "feat: add AI chat widget to homepage"
```

---

## Task 5: Deploy and Connect

- [ ] **Step 1: Set the Anthropic API key secret (before first deploy)**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
npx wrangler secret put ANTHROPIC_API_KEY --env production
```

Enter your Anthropic API key when prompted. This must be done before deploying so the Worker never goes live without the key.

- [ ] **Step 2: Deploy the Cloudflare Worker**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
npx wrangler deploy --env production
```

Note the deployed Worker URL (e.g., `https://axiom-chatbot.your-subdomain.workers.dev`). For now, use the `*.workers.dev` URL. A custom domain (`api.hireaxiom.com`) can be configured later by adding a CNAME DNS record in Cloudflare pointing to the Workers network.

- [ ] **Step 3: Update the frontend WORKER_URL**

In `ai-mac-mini/index.html`, update the `WORKER_URL` constant in the chat widget script to the actual deployed Worker URL from Step 2.

- [ ] **Step 4: Push frontend to GitHub Pages**

```bash
cd /Users/tylermeeks/ai-mac-mini
git add index.html
git commit -m "chore: set production Worker URL for chat widget"
git push origin main
```

- [ ] **Step 5: Test end-to-end on hireaxiom.com**

1. Visit hireaxiom.com
2. Click the chat bubble in the bottom-right
3. Ask "What is Axiom?" — verify you get a helpful response
4. Ask "Can you help me file my taxes?" — verify handoff triggers
5. Fill in the handoff form — verify Tyler receives email at tyler@hireaxiom.com
6. Test on mobile — verify full-width panel
7. Send 20+ messages — verify conversation limit message appears

- [ ] **Step 6: Final commit**

```bash
cd /Users/tylermeeks/axiom-chatbot-worker
git add -A
git commit -m "chore: finalize chatbot Worker deployment"
```
