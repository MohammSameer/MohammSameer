<h1 align="center">Mohammad Sameer</h1>

<p align="center">
  <b>Full-Stack Developer · 1 year · Solis Technology, Gurgaon</b><br/>
  NestJS · Next.js · TypeScript · PostgreSQL · AWS<br/>
  <i>Open to SDE-1 / SDE-2 backend and full-stack roles</i>
</p>

<p align="center">
  <a href="https://mohammsameer.github.io/"><img src="https://img.shields.io/badge/Portfolio-000000?style=for-the-badge&logo=githubpages&logoColor=white" alt="Portfolio"/></a>
  <a href="https://linkedin.com/in/md-sameer123"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge" alt="LinkedIn"/></a>
  <a href="mailto:sameermunthaj@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"/></a>
</p>

---

## 💫 About Me

I build the parts of a product where a mistake costs money or trust: approval workflows, commission and payments, document processing, audit trails and access control. At **Solis Technology** I work across three products:

- **Solis Insurify:** insurance distribution
- **HRMS:** a multi-tenant HR and payroll SaaS
- **Aurum CoNexus:** a members-only leadership network

I work AI-first with Claude Code. My commit messages record the reasoning and root cause behind each change, and most changes ship with tests.

> **Where's the activity?** Company code is private and committed from my work account, [@mohammadsameer-byte](https://github.com/mohammadsameer-byte). The contribution graph on this profile only shows personal projects.

| Product | My period | My footprint |
|---|---|---|
| **Solis Insurify** | Jan 2026 – present | 44 merged PRs · 31 database migrations · 200+ test files |
| **HRMS** | Apr – Jul 2026 | 24 database migrations · 116 test files (unit, integration, Playwright) |
| **Aurum CoNexus** | Dec 2025 – May 2026 | 3 backend modules · 21 API endpoints · only committer across 4 repos since Jan 2026 |

---

## 🚀 Engineering Highlights

### 💰 Found why commission was coming out as ₹0 · *Solis Insurify*
**Outcome:** commission that was silently computing as zero is now priced from the correct rates. Correcting a policy re-prices it safely, without touching money that's already been paid.

I traced the zero to four separate faults hiding behind each other:
- a config key nobody read
- a failed rate lookup that returned 0 instead of falling back
- mismatched product and product-version IDs
- pricing inputs that never left the finance service

Those inputs now travel end to end (events, DTOs, gRPC), and rates are pinned to the product version in force on the sale date. Re-pricing is append-only, recorded as a reversal plus a new accrual. It refuses to run once any instalment is paid, and database `CHECK` constraints are the last guard. Another engineer started the engine; I'm now its main author, and I built the Payin Dashboard and the configurable settlement terms.

`NestJS` `gRPC` `PostgreSQL` `Prisma` `Next.js` `Jest`

### 📄 Policy PDFs that fill in the form themselves · *Solis Insurify*
**Outcome:** a relationship manager uploads an insurer's policy PDF and the policy form fills itself. Any field the engine isn't confident about is flagged for review instead of guessed. Now live in production.

- **Why the old approach failed:** regex over flattened PDF text broke on insurers' column layouts.
- **What I built:** a two-layer engine. A geometry layer rebuilds rows, cells and label/value pairs across columns. A semantic layer classifies pages, scopes sections, and maps each field with a confidence score and a record of where it came from.
- **How it stays safe:**
  - It shipped behind a feature flag, with the old parser kept as a fallback whose values are only ever suggested.
  - Every run is audited.
  - A golden-file test set enforces a minimum recall that can only go up.
  - 300+ tests.

`TypeScript` `NestJS` `pdf.js` `PostgreSQL` `Prisma` `Next.js` `Jest`

### 🧾 Offer letters from HR's own templates · *HRMS*
**Outcome:** HR generates offer letters from its existing PDF templates, and documents candidates submit before the offer carry straight into onboarding. I also fixed two bugs HR reported from production: letters printing blank characters, and salaries showing 100× too small.

**How it works:**
1. Three detectors read text positions from the template: explicit tokens, label-plus-blank, and underlined runs.
2. An admin reviews the fields they found.
3. pdf-lib stamps real form fields into the template.

**The two production bugs:**
- The blank characters came from pdf-lib's runtime font subsetting. I replaced it with a font subset built offline with HarfBuzz.
- The 100× error came from one form sending paise and another sending rupees. Everything now uses one unit, and every offer is checked against the requisition's salary band.

Copying pre-offer documents into onboarding is safe to re-run and integration-tested with Testcontainers.

`NestJS` `pdf.js` `pdf-lib` `PostgreSQL` `Prisma` `Next.js` `Vitest` `Testcontainers`

<details>
<summary><b>Two more deep-dives: keeping data consistent across services</b></summary>
<br/>

#### 🔁 Agent reassignment that can't drift · *Solis Insurify*
**Outcome:** HR moves sales agents between managers in one batch. Every downstream service picks up the change, and retries on its own if a service is down. Turning on the reconciler repaired a mirror table that was missing most agents.

- An append-only ledger that doubles as a transactional outbox, with delivery tracked per downstream service, retries with backoff, and a reconciler that checks against live data
- Idempotency keys backed by a unique index; batched compare-and-swap updates that report concurrent edits instead of overwriting them
- I built the schema, service, scheduled jobs, the receiving endpoint in each of four services, and the HR screen; 92 tests

`NestJS` `PostgreSQL` `Prisma` `Next.js` `Jest`

#### 🪪 KYC case lifecycle with rules enforced in the database · *Solis Insurify*
**Outcome:** partner onboarding stopped producing duplicate KYC cases and approvals that could never complete. Rejected documents and bank proofs can now be corrected without starting over.

- One case-state model shared by two services and three portals, replacing status lists that had drifted apart
- A partial unique index guarantees one open case per agent. Re-linking a document is safe to repeat. Automation stops instead of guessing when two status sources disagree.
- Fair reviewer assignment (least recently assigned) inside a transaction, and per-document rejection with an append-only review history
- I built the module and wrote most of its history; 190+ tests

`NestJS` `PostgreSQL` `Prisma` `Next.js` `Jest`

</details>

<details>
<summary><b>More from Solis Insurify</b></summary>
<br/>

- **Sales credited to the right agent:** policies now record who sold them, not who typed them in. This restored 34 policies to their agents' portals without moving any commission.
- **Lead → policy conversion:** a separate "issued by the insurer" lifecycle keeps paid-but-unissued business out of renewals and commission clawbacks.
- **Leads Assigner role:** access is denied unless explicitly allowed. Lead claims are transactional, so two assigners can't overwrite each other. Leads are distributed down the sales reporting line.
- **Activity and audit log:**
  - every event is delivered at least once, and duplicates are processed only once
  - a dead-letter queue catches failures
  - personal data is redacted from stored payloads
  - a test fails if any event type lacks a handler
- **Reporting-line access control:** one recursive-CTE hierarchy resolver defines team scope for five services.
- **Insurer email ticketing:** HMAC-signed reply addresses, and replies matched to their ticket in three tiers (signed alias → Message-ID chain → subject tag).
- **Daily business report:** a single server-side source of truth, plus a scheduled email sent exactly once, with a retry sweep and catch-up on restart.
- **Operations service foundations:** KYC, claims, SLA timers on a business-hours calendar, and contests with live leaderboards.
- **Real-time support chat:** Socket.IO with a Redis adapter, presence and read receipts.
- **Employee onboarding on Better Auth:**
  - email invites
  - routing to the right portal by role
  - email-address normalisation
  - fail-closed authentication between services
- **Team tooling:** a conventional-commit hook and a PR-title CI gate, now used across the repo.

</details>

<details>
<summary><b>More from HRMS</b></summary>
<br/>

- **SaaS billing layer, built from scratch:**
  - usage metering
  - per-employee pricing with GST invoicing
  - Razorpay payments
  - follow-up on failed payments (dunning)
  - plan entitlement checks
  - billing workflows on Temporal
- **Access control:** role checks added to 68 API controllers, separation-of-duties rules, per-user rate limiting, and a 54-spec per-role Playwright suite in CI.
- **Payroll:** salary templates, structure revisions and a recovery ledger.
- **Attendance:** a shift roster and an employee calendar.

</details>

<details>
<summary><b>More from Aurum CoNexus</b></summary>
<br/>

- **Google Workspace scheduling:** a service-account integration that creates Meet links automatically, syncs attendees, regenerates links on reschedule, and checks each staff member's bookings for overlaps.
- **IST-safe dates:** one time-zone convention across the meeting APIs, whatever the server's time zone.
- **Event registration:** confirmed only by Razorpay's payment webhook, with a bypass for free events. Draft events notify members on publish.
- **Membership upgrades:** a request workflow with admin push notifications, plus admin dashboard and newsletter APIs.

</details>

---

## 🛠️ Personal Projects

### 🤖 [Sugam Form Filler](https://github.com/MohammSameer/sugam-form-filler): an AI agent that helps people fill government forms
Upload a photo of a form. The agent explains every field in your language, asks one question at a time, and returns a filled PDF.
- **Google ADK multi-agent setup:** an orchestrator hands off to a vision form-reader and a conversational data collector
- **Gemini 2.5 Flash vision,** in 10 Indian languages
- **A security checkpoint before every model call:** masks Aadhaar, PAN and bank numbers, blocks prompt injection, and writes an audit log
- **An MCP server** for form parsing, scheme lookup and PDF generation

`Python` `Google ADK` `Gemini` `MCP` `pytest`

### 💳 [CoinStack](https://github.com/MohammSameer/financial-app): credit-card spend analytics and rewards · [**Live demo**](https://coin-stack.vercel.app)
*A take-home assignment, built and deployed end to end.*
- Loads 10,000 deliberately messy transactions without dropping a row, and shows a data-quality report in the UI
- Filtering, sorting and paging all happen in PostgreSQL. The charts and table share one filter, so each one narrows the other.
- Atomic coin redemption that is safe to retry, enforced by a unique index; the UI updates optimistically and rolls back on failure
- Every component built by hand, with no UI library; keyboard and screen-reader support

`Next.js` `React` `TypeScript` `FastAPI` `PostgreSQL` `pytest`

### 🍔 EpicEats: food-delivery web app · [**Live demo**](https://epic-eats-frontend.vercel.app) · [Frontend](https://github.com/MohammSameer/epicEats_frontend) · [Backend](https://github.com/MohammSameer/epicEats_backend)
*My internship project.*
- JWT authentication with bcrypt-hashed passwords, a shared cart and order history
- A paginated catalogue API where the client chooses which fields come back, with an optional Redis cache the API keeps working without

`React` `Express` `MongoDB` `Redis`

---

## 💻 Tech Stack

| | |
|---|---|
| **Languages** | ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) |
| **Backend** | ![Node.js](https://img.shields.io/badge/Node.js-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white) ![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat-square&logo=nestjs&logoColor=white) ![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) ![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white) ![gRPC](https://img.shields.io/badge/gRPC-244C5A?style=flat-square) ![Temporal](https://img.shields.io/badge/Temporal-000000?style=flat-square&logo=temporal&logoColor=white) ![Socket.io](https://img.shields.io/badge/Socket.io-010101?style=flat-square&logo=socketdotio&logoColor=white) |
| **Frontend** | ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) ![Redux](https://img.shields.io/badge/Redux-764ABC?style=flat-square&logo=redux&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white) |
| **Data** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white) |
| **Cloud & DevOps** | ![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square) ![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?style=flat-square&logo=googlecloud&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![Nx](https://img.shields.io/badge/Nx-143055?style=flat-square&logo=nx&logoColor=white) |
| **Testing** | ![Jest](https://img.shields.io/badge/Jest-C21325?style=flat-square&logo=jest&logoColor=white) ![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=vitest&logoColor=white) ![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=flat-square) |
| **AI** | ![Claude Code](https://img.shields.io/badge/Claude_Code-D97757?style=flat-square&logo=claude&logoColor=white) ![Gemini](https://img.shields.io/badge/Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white) ![MCP](https://img.shields.io/badge/MCP-000000?style=flat-square) |

---

## 🎓 Experience & Education

- **Full-Stack Developer**, Solis Technology, Gurgaon · *Oct 2025 – present*
- **Web Development Intern**, Edunet Foundation (EY GDS & AICTE) · *Mar – Apr 2025*
- **B.Tech, Computer Science & Engineering**, Sri Indu College · *2021 – 2025*
