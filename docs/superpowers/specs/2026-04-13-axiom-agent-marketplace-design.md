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
- Axiom Mac Mini buyers get discounted agent subscriptions (loyalty + upsell)
- Entry-level single agent: $29-49/mo (bridge from prompt packs to full suites)

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

**Summary:** 6 agents already built or mostly built. 10 need building. Launch with the 6 that work today, add 4-5 more over 8-12 weeks. Defer the remaining 5 to Phase 3+. A 16-agent catalog is the long-term vision, not the launch target.

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
| Agent runtime | Python on Railway or Fly.io | All existing scrapers are Python. CF Workers can't run Python. Railway gives always-on containers with no timeout limits. |
| AI | Claude API (Haiku + Sonnet) | Already integrated, proven in production |
| Database | Supabase (Postgres) | Free tier, real-time, auth built-in |
| Billing | Stripe | Subscriptions, usage-based billing, marketplace payouts (for Tier 3) |
| Job scheduling | BullMQ (Redis) on Railway/Fly.io | GitHub Actions works for single-tenant; multi-tenant needs a real job queue with per-customer scheduling |
| Email | SendGrid + Gmail SMTP | Already integrated |
| Hosting | Vercel (frontend) + Cloudflare Workers (API) | Low cost, auto-scaling |
| Auth | Clerk or Supabase Auth | Fast to implement, handles OAuth |

### Prompt Pack Delivery
- Stored as markdown files in a private GitHub repo or S3
- On purchase, buyer gets a download link (time-limited signed URL)
- For Claude Projects: provide a "one-click import" URL or copy-paste instructions
- Gumroad can handle this initially (you already have an account)

---

## 5. Cost Model & Unit Economics

### Per-Customer Cost Estimates (Monthly)

#### Healthcare Staffing Suite ($99/mo)
| Cost Item | Estimate | Notes |
|-----------|----------|-------|
| Claude API (Haiku) — facility identification | $1.50-3.00 | ~50-100 jobs/day × 30 days × ~$0.001/call |
| Claude API (Haiku) — enrichment | $0.50-1.00 | Identifier runs on new jobs only |
| Infrastructure (Railway container share) | $2-5 | Shared container, runs ~30 min/day |
| Email delivery (SendGrid/Gmail) | $0-1 | Well within free tiers at low volume |
| **Total COGS per customer** | **$4-10/mo** | |
| **Gross margin** | **$89-95 (90-96%)** | |

#### Sales Outreach Suite ($149/mo)
| Cost Item | Estimate | Notes |
|-----------|----------|-------|
| Claude API (Haiku) — lead enrichment | $5-15 | Depends on lead volume; ~500 leads/mo × website scrape + analysis |
| Claude API (Sonnet) — email drafting | $3-8 | ~100-200 outreach drafts/mo |
| DuckDuckGo scraping / web requests | $0 | Free, but subject to rate limits |
| Infrastructure | $3-5 | Higher compute for enrichment |
| Email delivery | $0-2 | |
| **Total COGS per customer** | **$11-30/mo** | |
| **Gross margin** | **$119-138 (80-93%)** | |

#### Platform Infrastructure (Fixed Monthly)
| Cost Item | Estimate at 10 customers | Estimate at 100 customers |
|-----------|-------------------------|--------------------------|
| Railway (Python agent runtime) | $5-10 | $25-50 |
| Supabase (database) | $0 (free tier) | $25 |
| Vercel (frontend) | $0 (free tier) | $20 |
| Cloudflare Workers (API) | $0 (free tier) | $5 |
| Stripe fees (2.9% + $0.30) | $35 | $350 |
| **Total fixed infra** | **$40-45/mo** | **$425-450/mo** |

**Key insight:** Margins are healthy. The risk is not average cost — it's outlier customers who run heavy enrichment volumes. Mitigation: usage caps per tier with overage pricing.

### Break-Even Analysis
- Fixed costs (infra + Tyler's time opportunity cost): ~$2,000/mo
- Average revenue per customer: ~$120/mo
- Average COGS per customer: ~$15/mo
- **Break-even: ~19 customers**

---

## 6. Legal & Compliance Considerations

### Scraping & Data Redistribution
- **Internal use vs. resale:** The existing scrapers were built for ConnectHealth's internal operations. Selling scraped job board data as a SaaS product changes the legal posture significantly.
- **Action required before launch:** Review Terms of Service for each scraped site (CompHealth, Vista, AyaLocums, MPLT, Weatherby, Global Medical, LocumJobsOnline) to assess redistribution risk.
- **Mitigation options:**
  - Position as a "monitoring/alerting" service (customer gets notified of jobs, not a copy of the database)
  - Each customer runs their own scraper instance (they're the ones accessing the site, not Axiom)
  - Focus Tier 2 on the enrichment/matching/outreach layer, not raw data resale
  - Consider partnering with job boards or using their official APIs where available

### Customer Data
- Customers will provide: email addresses, CRM credentials, business information
- **Required:** Privacy policy and Terms of Service for the marketplace
- **Required:** Data Processing Agreement (DPA) template for business customers
- **Action:** Extend the existing hireaxiom.com privacy policy to cover agent data handling
- CRM credentials must be encrypted at rest (Supabase vault or AWS Secrets Manager)

### Financial
- Stripe handles PCI compliance for payment processing
- Need to collect W-9 / tax info from third-party creators before Phase 3 (Stripe Connect handles most of this)

---

## 7. Multi-Tenancy Architecture

### The Core Challenge
The existing scrapers are single-tenant: one state.json, one config, one cron job, one email recipient. Selling them as SaaS requires running N instances with isolated state and config.

### Approach: Per-Customer Container with Shared Codebase

```
┌─────────────────────────────────────┐
│         Agent Orchestrator          │
│  (BullMQ job queue on Railway)      │
│                                     │
│  Schedules jobs per customer:       │
│  - customer_123: run job-scanner    │
│  - customer_456: run lead-enricher  │
└──────────────┬──────────────────────┘
               │
┌──────────────┴──────────────────────┐
│         Agent Worker Pool           │
│  (Python workers on Railway)        │
│                                     │
│  Each job runs with:                │
│  - customer_id (from job payload)   │
│  - config pulled from database      │
│  - state read/written to database   │
│  - output scoped to customer        │
└─────────────────────────────────────┘
```

### Key Design Decisions
1. **Config:** Per-customer settings stored in Supabase (Postgres). Agent reads config at runtime based on customer_id passed in the job payload.
2. **State:** Migrate from JSON files to database rows. Each state entry keyed by (customer_id, agent_type, item_id). This is the biggest refactor needed.
3. **Isolation:** Workers are stateless — all state lives in the database. No cross-customer leakage possible because every query is scoped by customer_id.
4. **Scaling:** Start with a single worker processing jobs sequentially. Add workers when queue depth grows. BullMQ handles concurrency.
5. **Failure handling:** Failed jobs retry 3x with exponential backoff. After 3 failures, mark as failed in dashboard and notify customer via email. One customer's failure does not block others.

### Migration Path (from current → multi-tenant)
1. Abstract state.py to read/write from Postgres instead of JSON files
2. Add a `config` table with per-customer agent settings
3. Replace cron triggers with BullMQ scheduled jobs
4. Add customer_id parameter to all agent entry points
5. Test with ConnectHealth as customer_001 before onboarding anyone else

---

## 8. Operational Plan

### Solo Operator Risk
Tyler is the sole builder, operator, and support agent. This is the #1 operational risk.

**Mitigations:**
- **Monitoring first:** Before selling any agent, instrument it with health checks, success/failure logging, and automated alerts (PagerDuty or simple email/SMS on failure)
- **Automated recovery:** Agents should auto-retry on transient failures. Scraper target changes trigger an alert, not silent failure.
- **Customer communication:** Transparent status page (simple: Supabase table + static page). If a scraper breaks, customers see "Job Board X: degraded" within minutes.
- **Support model (Phase 1):** Email only, 24-hour response SLA on weekdays. No phone. No 2am pages for prompt packs.
- **Hire trigger:** When support tickets exceed 5/week or MRR hits $5,000, hire a part-time contractor for customer support.

### Scraping Target Risk
Job boards change HTML, add Cloudflare, or block IPs. This WILL happen.

**Mitigations:**
- IP rotation via proxy service (BrightData, ScraperAPI) for high-risk targets
- Headless browser fallback (Playwright) already in use for CompHealth
- Alert on scrape failure → manual fix within 24 hours
- Customer SLA: "We monitor 7+ job boards. Individual sources may experience temporary outages. We aim to restore within 48 hours."
- Diversify sources — the more boards you scrape, the less any single one matters

### Churn Modeling
Realistic SaaS churn at this price point: 5-8% monthly.

| Month | New Customers | Churned | Net Subscribers | MRR |
|-------|--------------|---------|----------------|-----|
| 1 | 10 | 0 | 10 | $1,200 |
| 3 | 15 | 3 | 40 | $4,800 |
| 6 | 20 | 8 | 80 | $9,600 |
| 12 | 25 | 15 | 175 | $21,000 |

*More conservative than the original projections. Assumes 7% monthly churn starting Month 2.*

---

## 9. Go-to-Market Strategy

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

### Phase 3 — Scale (Months 4-8)
**Goal:** Expand catalog, add remaining agents.

- Build 4-5 more agents from the catalog (prioritize by customer demand)
- Launch affiliate/referral program
- Add agent ratings and reviews
- Target: 80-100 subscribers

### Phase 4 — Creator Marketplace (Months 9-12)
**Goal:** Open to third-party creators. (Moved from Phase 3 — Stripe Connect compliance is complex.)

- Build creator portal (submit agent, set pricing, view sales)
- Implement Stripe Connect for marketplace payouts (20-30% commission)
- Target: 150+ subscribers, 10+ third-party agents listed

### Phase 5 — Moat (Year 2)
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

## 10. Competitive Landscape

| Competitor | What They Do | Axiom Advantage |
|-----------|-------------|-----------------|
| Darius Lukas Black Book | 70 custom GPTs + prompt library ($997) | Axiom agents actually DO work (scrape, email, automate). His are conversational only. |
| Zapier / Make.com | No-code automation platforms | Generic. Axiom provides pre-built, industry-specific agents. No building required. |
| Clay.com | Sales data enrichment | Narrow focus (sales only). Axiom covers multiple business functions. |
| Relevance AI | AI agent builder platform | Developer-focused. Axiom targets non-technical business owners. |
| Custom dev agencies | Build bespoke AI solutions | Expensive ($10K-50K+). Axiom is self-serve and affordable. |

**Axiom's moat:** Tyler has built AND operated these agents for his own business. They're not theoretical — they run daily, generate real leads, and find real jobs. No one else in the market can say "I built this for my own company, it works, now I'm selling it to you."

---

## 11. Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| AI API costs eat margin on hosted agents | Use Haiku (cheapest) for enrichment tasks. Monitor per-customer costs. Set usage limits per tier. |
| Claude API changes break agents | Abstract the AI layer. Most agents are scraping + email logic with AI as one step. |
| Customer data security | Tenant isolation from day one. No cross-customer data access. SOC2 compliance roadmap for Phase 3+. |
| Support burden as customer count grows | Self-service dashboard. Agent health monitoring. Automated alerts on failures. |
| Third-party agents (Phase 3) quality control | Review process before listing. Sandbox testing. Customer ratings. Revenue share incentivizes quality. |
| Competition from big platforms (OpenAI, Anthropic) | They'll stay horizontal. Axiom wins on vertical-specific, pre-configured agents. Speed to value. |
| Single-person operational risk | Monitoring/alerting from day one. Automated recovery. Hire trigger at $5K MRR or 5 tickets/week. See Section 8. |
| Scraping targets block/break | IP rotation, Playwright fallback, source diversification, 48-hour restoration SLA. See Section 8. |
| Per-customer API costs exceed pricing | Usage caps per tier. Haiku for bulk tasks. Monitor and alert on outlier customers. See Section 5. |

---

## 12. Immediate Next Steps

0. **Instrument existing agents for monitoring** — add success/failure logging, run duration tracking, and email alerts on failure. This is prerequisite to selling anything.
1. **Review scraping ToS** — check each job board's terms for redistribution risk. Decide on per-customer scraper instances vs. centralized scraping.
2. **Migrate state management** — refactor state.py from JSON files to Postgres (Supabase). Key by (customer_id, agent_type, item_id). Test with ConnectHealth as customer_001.
3. **Build a /marketplace page** on hireaxiom.com — agent descriptions, pricing, Stripe checkout
4. **Create 3-4 prompt packs** — sell on Gumroad as top-of-funnel (fastest revenue)
5. **Package Healthcare Staffing Suite** — wrap the 7 job scrapers with per-customer config on Railway + BullMQ
6. **Package Sales Outreach Suite** — wrap axiom-sales into multi-tenant product
7. **Offer to existing Axiom customers** — warm outreach, launch pricing
8. **Set up Stripe subscriptions** — monthly billing with failed payment handling (3-day grace period, then pause agents)

---

## 13. Open Questions

- Should Axiom Mac Mini buyers get bundled agent subscriptions (e.g., 3 months free)?
- What's the right Stripe pricing structure — per-agent or per-bundle?
- Do we need a custom domain for the marketplace (e.g., agents.hireaxiom.com)?
- How do we handle customers who need agents customized for their specific CRM/tools?
- What's the support model — email only, chat, or included setup calls?
