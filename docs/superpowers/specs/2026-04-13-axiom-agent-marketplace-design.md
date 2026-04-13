# Axiom Agent Marketplace — Design Spec

**Date:** 2026-04-13
**Author:** Tyler Meeks
**Status:** Draft

---

## 1. Overview

### What
An AI agent marketplace under the Axiom brand where businesses can discover, purchase, and deploy pre-built AI agents that automate real work — not just generate text. The marketplace complements Axiom's existing hardware offering (pre-configured Mac Minis) by providing the software layer: purpose-built agents that run on those machines or in the cloud.

### Why
- Tyler has already built 10+ working AI agents for ConnectHealth and Axiom sales — these are production-proven, not prototypes
- The market is flooded with prompt packs and "AI courses" but lacks turnkey agents that actually DO things (scrape, email, enrich, automate)
- Axiom's existing Mac Mini customers are a natural first audience — they already have the hardware, now they need the software
- Recurring revenue (SaaS subscriptions) is more valuable than one-time hardware sales

### Who
**Primary buyers:**
- Small business owners (5-50 employees) who know AI can help but don't know how to build it themselves
- Axiom Mac Mini customers looking for more automation
- Healthcare staffing companies, recruiting firms, insurance agencies, law firms, accounting firms, consulting firms, medical practices

**Secondary audience (Phase 2):**
- AI builders/developers who want to sell their agents through the platform

---

## 2. Business Model

### Three-Tier Product Structure

#### Tier 1 — Prompt Packs (Free / $5-29 one-time)
- **What:** Pre-built Claude Projects, system prompts, prompt templates
- **Delivery:** Downloadable markdown files, copy-paste instructions, or Claude Project import links
- **Purpose:** Top of funnel. Gets people in the door, builds trust, grows the email list
- **Examples:** "Healthcare Recruiter Prompt Pack," "Sales Email Template Suite," "Meeting Prep Prompts"
- **Margin:** Near 100% (no infrastructure cost)

#### Tier 2 — Hosted Agents (Subscription, $29-199/mo per agent or bundle)
- **What:** Agents that run autonomously on Axiom infrastructure — connected to email, CRMs, job boards, APIs
- **Delivery:** Buyer gets a dashboard login. Agent runs on a schedule, sends reports, takes actions.
- **Purpose:** Core revenue driver. Monthly recurring revenue with high retention (agents become essential to operations)
- **Examples:** "Job Board Scanner" ($49/mo), "Lead Discovery + Outreach Suite" ($149/mo), "Healthcare Staffing Bundle" ($199/mo)
- **Margin:** ~70-80% after AI API costs and infrastructure

#### Tier 3 — Creator Marketplace (20-30% commission)
- **What:** Third-party builders list their agents on the platform
- **Delivery:** Axiom handles hosting, billing, and distribution
- **Purpose:** Scale the catalog without building everything internally. Network effects.
- **Timeline:** Phase 2 (launch after proving the model with Tier 1 and Tier 2)

### Pricing Strategy
- Prompt packs priced to be impulse buys ($5-29)
- Individual hosted agents: $29-99/mo
- Agent bundles (3-5 related agents): $99-199/mo
- Enterprise/custom: Contact sales
- Axiom Mac Mini buyers get discounted agent subscriptions (loyalty + upsell)

---

## 3. Launch Catalog — 16 Agents Across 3 Categories

### Category A: Healthcare Staffing Agents
*Source: Built from ConnectHealth operations. Production-proven.*

| # | Agent | Description | Build Status | Tier |
|---|-------|-------------|-------------|------|
| 1 | Job Board Scanner | Scrapes 7+ boards daily (CompHealth, Vista, AyaLocums, MPLT, Weatherby, Global Medical, LocumJobsOnline), emails new physician/CRNA jobs with facility identification | ✅ Built | Hosted |
| 2 | Provider Matcher | Match provider CVs/profiles to open jobs by specialty, location, availability, pay rate | 🔧 Needs build | Hosted |
| 3 | Credential Tracker | Monitors license, DEA, malpractice, board cert expirations. Sends renewal alerts. Tracks per-facility privilege requirements | 🔧 Needs build | Hosted |
| 4 | Recruiting Outreach Drafter | Generates personalized Doximity/LinkedIn/email outreach to providers based on their profile + matching jobs | 🔧 Needs build | Prompt + Hosted |
| 5 | Rate Calculator | Estimates locum bill/pay rates by specialty, location, and market data (CMS, historical placements) | 🔧 Needs build | Prompt |
| 6 | Facility Intel Agent | Researches facilities before pitching — pulls CMS data, reviews, bed count, specialties, recent news | 🔧 Needs build | Prompt + Hosted |

### Category B: Sales & Outreach Agents
*Source: Built from axiom-sales pipeline. 4,900+ leads generated.*

| # | Agent | Description | Build Status | Tier |
|---|-------|-------------|-------------|------|
| 7 | Lead Discovery Engine | Finds small businesses across any vertical via DuckDuckGo scraping. Covers 20 cities, 8 verticals. | ✅ Built | Hosted |
| 8 | Lead Enricher | Visits lead websites, extracts decision-makers, company size, tech stack. Claude Haiku scores leads 1-10. | ✅ Built | Hosted |
| 9 | Email Sequence Writer | Drafts personalized 3-email cold outreach sequences based on lead profile and vertical | ✅ Built | Hosted |
| 10 | Follow-Up Automator | Sends timed follow-ups on schedule, handles email delivery via SendGrid/Gmail | ✅ Built (partial) | Hosted |
| 11 | CRM Sync Agent | Logs lead activity, outreach status, and responses back to Bullhorn/HubSpot/Salesforce | 🔧 Needs build | Hosted |

### Category C: General Business Ops Agents
*Source: Broad appeal. Every small business needs these.*

| # | Agent | Description | Build Status | Tier |
|---|-------|-------------|-------------|------|
| 12 | Email Triage Agent | Categorizes inbox by priority/topic, drafts replies, flags urgent items | 🔧 Needs build | Hosted |
| 13 | Meeting Prep Agent | Researches attendees (LinkedIn, company site), summarizes relevant docs/emails before meetings | 🔧 Needs build | Prompt + Hosted |
| 14 | Social Media Poster | Generates and schedules LinkedIn/X/Instagram posts from your content, blog, or talking points | 🔧 Needs build | Prompt + Hosted |
| 15 | Invoice & Proposal Drafter | Generates professional proposals, SOWs, and invoices from templates + context | 🔧 Needs build | Prompt |
| 16 | Competitor Monitor | Tracks competitor websites, pricing pages, job postings, and social media for changes. Weekly digest. | 🔧 Needs build | Hosted |

**Summary:** 6 agents already built or mostly built. 10 need building. Launch with the 6 that work today, add the rest over 8-12 weeks.

---

## 4. Technical Architecture

### Platform Components

```
┌─────────────────────────────────────────────────┐
│                 AXIOM MARKETPLACE                │
│           (hireaxiom.com/marketplace)            │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ Storefront│  │ Dashboard │  │ Agent Runner  │  │
│  │  (Next.js)│  │  (Next.js)│  │  (Python/Node)│  │
│  └─────┬────┘  └─────┬────┘  └──────┬───────┘  │
│        │              │              │           │
│  ┌─────┴──────────────┴──────────────┴───────┐  │
│  │              API Layer (Node.js)           │  │
│  │  - Auth (Clerk/Auth.js)                    │  │
│  │  - Billing (Stripe subscriptions)          │  │
│  │  - Agent orchestration                     │  │
│  │  - Webhook handling                        │  │
│  └─────────────────┬─────────────────────────┘  │
│                    │                             │
│  ┌─────────────────┴─────────────────────────┐  │
│  │           Infrastructure                   │  │
│  │  - Database (Supabase/Postgres)            │  │
│  │  - Job queue (BullMQ / GitHub Actions)     │  │
│  │  - File storage (S3/R2)                    │  │
│  │  - Email delivery (SendGrid/Gmail)         │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### How Hosted Agents Run

Each hosted agent follows the same pattern Tyler's scrapers already use:

1. **Trigger:** Cron schedule (GitHub Actions) or webhook (user-initiated)
2. **Execute:** Python script runs — scrapes, enriches, analyzes, drafts
3. **AI Layer:** Claude API (Haiku for enrichment, Sonnet for drafting)
4. **Output:** Email report, CRM update, file generation, or dashboard update
5. **State:** JSON state files track what's been processed (deduplication)

For the marketplace, we wrap each agent in:
- **Config layer:** Per-customer settings (email address, CRM credentials, verticals, etc.)
- **Tenant isolation:** Each customer's state/data is isolated
- **Monitoring:** Track runs, success/failure, output quality

### Tech Stack

| Component | Technology | Reason |
|-----------|-----------|--------|
| Storefront / Dashboard | Next.js + Tailwind | Already familiar, fast to build, SEO-friendly |
| API | Node.js (Cloudflare Workers or Vercel) | axiom-chatbot-worker already runs on CF Workers |
| Agent runtime | Python | All existing scrapers/agents are Python |
| AI | Claude API (Haiku + Sonnet) | Already integrated, proven in production |
| Database | Supabase (Postgres) | Free tier, real-time, auth built-in |
| Billing | Stripe | Subscriptions, usage-based billing, marketplace payouts (for Tier 3) |
| Job scheduling | GitHub Actions + cron | Already proven across 7 scrapers |
| Email | SendGrid + Gmail SMTP | Already integrated |
| Hosting | Vercel (frontend) + Cloudflare Workers (API) | Low cost, auto-scaling |
| Auth | Clerk or Supabase Auth | Fast to implement, handles OAuth |

### Prompt Pack Delivery
- Stored as markdown files in a private GitHub repo or S3
- On purchase, buyer gets a download link (time-limited signed URL)
- For Claude Projects: provide a "one-click import" URL or copy-paste instructions
- Gumroad can handle this initially (you already have an account)

---

## 5. Go-to-Market Strategy

### Phase 1 — Validate (Weeks 1-4)
**Goal:** Prove people will pay for agents. Minimum viable marketplace.

- Package the 6 existing agents into 2 bundles:
  - **Healthcare Staffing Suite** (Job Board Scanner + facility identification) — $99/mo
  - **Sales Outreach Suite** (Lead Discovery + Enricher + Email Sequences) — $149/mo
- Create 3-4 prompt packs from ConnectHealth workflows — sell on Gumroad ($9-19)
- Add a `/marketplace` page to hireaxiom.com with agent descriptions and Stripe checkout
- Offer to existing Axiom Mac Mini customers first (warm audience)
- Target: 5-10 paying subscribers

### Phase 2 — Build Out (Weeks 5-12)
**Goal:** Expand catalog, build proper dashboard.

- Build remaining 10 agents from the launch catalog
- Create customer dashboard (agent status, run history, settings, reports)
- Add self-service onboarding (customer enters email, CRM creds, preferences)
- Integrate Stripe subscription management
- Launch content marketing (LinkedIn posts showing agent results, case studies from ConnectHealth)
- Target: 25-50 paying subscribers

### Phase 3 — Scale (Months 4-6)
**Goal:** Open the marketplace to third-party creators.

- Build creator portal (submit agent, set pricing, view sales)
- Implement Stripe Connect for marketplace payouts (20-30% commission)
- Add agent ratings, reviews, and categories
- Launch affiliate/referral program
- Target: 100+ subscribers, 10+ third-party agents listed

### Phase 4 — Moat (Months 6-12)
**Goal:** Network effects and defensibility.

- Agents that share data across the platform (e.g., Lead Enricher feeds into Email Sequence Writer)
- Agent "workflows" — chain multiple agents together
- Industry-specific bundles auto-configured for verticals
- API for developers to build on top of the platform
- White-label option (agencies resell under their brand)

---

## 6. Revenue Projections (Conservative)

| Milestone | Timeline | Subscribers | MRR | ARR |
|-----------|----------|------------|-----|-----|
| First paying customer | Week 2 | 1 | $99 | $1,188 |
| Validate | Month 1 | 10 | $1,200 | $14,400 |
| Build out | Month 3 | 40 | $5,000 | $60,000 |
| Scale | Month 6 | 100 | $12,000 | $144,000 |
| Moat | Month 12 | 250 | $30,000 | $360,000 |

*Assumes average $120/mo per subscriber. Does not include one-time prompt pack sales or Axiom hardware revenue.*

---

## 7. Competitive Landscape

| Competitor | What They Do | Axiom Advantage |
|-----------|-------------|-----------------|
| Darius Lukas Black Book | 70 custom GPTs + prompt library ($997) | Axiom agents actually DO work (scrape, email, automate). His are conversational only. |
| Zapier / Make.com | No-code automation platforms | Generic. Axiom provides pre-built, industry-specific agents. No building required. |
| Clay.com | Sales data enrichment | Narrow focus (sales only). Axiom covers multiple business functions. |
| Relevance AI | AI agent builder platform | Developer-focused. Axiom targets non-technical business owners. |
| Custom dev agencies | Build bespoke AI solutions | Expensive ($10K-50K+). Axiom is self-serve and affordable. |

**Axiom's moat:** Tyler has built AND operated these agents for his own business. They're not theoretical — they run daily, generate real leads, and find real jobs. No one else in the market can say "I built this for my own company, it works, now I'm selling it to you."

---

## 8. Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| AI API costs eat margin on hosted agents | Use Haiku (cheapest) for enrichment tasks. Monitor per-customer costs. Set usage limits per tier. |
| Claude API changes break agents | Abstract the AI layer. Most agents are scraping + email logic with AI as one step. |
| Customer data security | Tenant isolation from day one. No cross-customer data access. SOC2 compliance roadmap for Phase 3+. |
| Support burden as customer count grows | Self-service dashboard. Agent health monitoring. Automated alerts on failures. |
| Third-party agents (Phase 3) quality control | Review process before listing. Sandbox testing. Customer ratings. Revenue share incentivizes quality. |
| Competition from big platforms (OpenAI, Anthropic) | They'll stay horizontal. Axiom wins on vertical-specific, pre-configured agents. Speed to value. |

---

## 9. Immediate Next Steps

1. **Package the Healthcare Staffing Suite** — wrap the 7 job scrapers into a single product with per-customer config
2. **Package the Sales Outreach Suite** — wrap axiom-sales into a multi-tenant product
3. **Build a /marketplace page** on hireaxiom.com — agent descriptions, pricing, Stripe checkout
4. **Create 3-4 prompt packs** — sell on Gumroad as top-of-funnel
5. **Offer to existing Axiom customers** — warm outreach, special launch pricing
6. **Set up Stripe subscriptions** — monthly billing for hosted agents

---

## 10. Open Questions

- Should Axiom Mac Mini buyers get bundled agent subscriptions (e.g., 3 months free)?
- What's the right Stripe pricing structure — per-agent or per-bundle?
- Do we need a custom domain for the marketplace (e.g., agents.hireaxiom.com)?
- How do we handle customers who need agents customized for their specific CRM/tools?
- What's the support model — email only, chat, or included setup calls?
