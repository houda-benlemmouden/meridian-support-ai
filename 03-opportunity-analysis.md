# Meridian Systems — Opportunity Analysis

> Reference engagement. Meridian Systems is a fictional client. Volumes and handling times are stated assumptions, marked as such.

## 1. What this document decides

Discovery told us what customers ask about. This document decides what to do about it — specifically, which categories are safe to automate, which need a human in the loop, and which must never be touched.

The output is one recommendation and the arithmetic behind it.

## 2. The three buckets

| Bucket | Definition | Test applied |
|---|---|---|
| **Deflect** | Resolved without a human. Customer gets an answer immediately | The answer is the same every time **and** needs no account data **and** a wrong answer is cheap |
| **Assist** | AI classifies, routes, and drafts. A human sends | Needs account context or judgement, but the shape of the answer is predictable |
| **Human only** | Never automated, never auto-drafted. Routed to a named person fast | A wrong or slow answer plausibly costs money, trust, or the customer |

**The rule that decides the hard cases:** if I cannot state what a wrong answer costs, it does not go in Deflect.

## 3. Category by category

Monthly volumes are derived from 1,400 tickets/month at the sample shares. **Assumption.**

| Category | Monthly | Bucket | Reasoning |
|---|---|---|---|
| **HowTo** | 119 | **Deflect** (100%) | Product knowledge. Stable answers, no account data, wrong answer costs a follow-up ticket |
| **Scanner** | 112 | **Deflect** (~80%) | Repeatable troubleshooting steps. The cold-store and MDM cases are genuinely technical — those route to a human |
| **Login** | 140 | **Deflect** (~70%) | Password, MFA and lockout are self-service. **SSO, session and redirect-loop cases are not** — see finding below |
| **FeatureRequest** | 105 | **Deflect** (100%) | Not resolved — acknowledged, logged to the product backlog, customer told where to track it. That is the correct outcome, and it is currently done by hand |
| **Billing** | 105 | **Split** (~50% deflect) | Invoice copies, address and PO number changes are administrative. **Disputes, double charges, refunds and price increases are commercial** |
| **Reports** | 119 | **Split** (~40% deflect) | Formatting and how-to questions deflect. Blank PDFs, timeouts and wrong totals are bugs |
| **DataImport** | 105 | **Split** (~40% deflect) | Template, format and file-size questions deflect. Imports that overwrote or duplicated data are damage, not questions |
| **Seats** | 105 | **Assist** | Needs account data. Predictable shape, so AI drafts and routes; a human confirms |
| **Integration** | 105 | **Assist** | Requires investigation of logs and third-party state. Route to the integrations specialist |
| **Performance** | 105 | **Assist** | Needs an engineer to determine whether it is customer environment or platform |
| **StockDiscrepancy** | 84 | **Assist** | Needs account data and usually reveals a process issue. AI gathers context, human diagnoses |
| **Other** | 98 | **Assist** | Genuinely mixed: DPAs, security questionnaires, GDPR requests, account management. Route by content, resolve by human |
| **DataLoss** | 56 | **HUMAN ONLY** | A wrong answer here can cost the customer their inventory records. Never automated, never auto-drafted |
| **Cancellation** | 42 | **HUMAN ONLY** | Commercial conversation. An automated reply to "we're considering leaving" is the worst possible response |

## 4. The arithmetic

**Deflection-eligible volume**

| Category | Monthly | Eligible % | Eligible tickets |
|---|---|---|---|
| HowTo | 119 | 100% | 119 |
| Scanner | 112 | 80% | 90 |
| Login | 140 | 70% | 98 |
| FeatureRequest | 105 | 100% | 105 |
| Billing | 105 | 50% | 53 |
| Reports | 119 | 40% | 48 |
| DataImport | 105 | 40% | 42 |
| **Total eligible** | | | **555 (40% of volume)** |

**Reconciliation with discovery.** Discovery reported ~34% cleanly repetitive. The difference is FeatureRequest, which is not repetitive but *is* fully automatable because the correct outcome is acknowledgement and logging rather than resolution. Excluding it, the figures agree at 32%.

**Actual deflection, conservatively modelled**

Not every eligible ticket will deflect. Customers reopen, phrasing defeats the classifier, some people simply want a person.

> **Assumption: 60% real deflection rate on eligible volume.** Vendor case studies commonly claim 80–90%. 60% is deliberately conservative, because a project that promises 85% and delivers 62% is judged a failure, while one promising 60% and delivering 72% is judged a success. The number you commit to is a business decision, not a technical one.

**555 × 60% = 333 tickets/month deflected — 24% of total volume.**

**Time recovered**

> **Assumption: 12 minutes average handling time** across deflectable categories, including reading, response and logging.

333 × 12 minutes = **~67 hours/month ≈ 8 working days**, or roughly 0.4 of one agent's capacity.

**State this honestly.** This does not replace an agent. It gives eight people back about a day each per month, and it removes the most repetitive work — which is what drives the turnover Tomás is worried about. Framing it as headcount reduction would be both wrong and, to the support team, hostile.

## 5. The recommendation

> Deflect the seven partially or fully eligible categories — about 40% of volume, modelled at 60% real deflection. Auto-classify and route everything else to the right specialist. Route DataLoss and Cancellation to a named human immediately and never automate them, because a wrong answer there costs a customer.

## 6. What I am not recommending, and why

**Not automating DataLoss.** 56 tickets a month, 4% of volume. Automating them would save roughly 11 hours a month. One mishandled data-loss ticket during a stock take costs a renewal worth far more. The arithmetic is not close.

**Not automating Cancellation.** 42 tickets a month, each representing a customer actively considering leaving. These are the highest-value conversations Meridian has. An automated reply signals exactly the indifference that caused the churn.

**Not building the help-centre chatbot first.** Ana's original request. The discovery data shows the biggest immediate problem is not deflection — it is that high-consequence tickets sit in a shared inbox at the same priority as password resets. **Fixing routing requires no AI, costs nothing, and can be done this week.** It should ship before anything else.

**Not promising 80% deflection.** The number is achievable in vendor decks and not in Meridian's ticket mix, where a third of tickets are hard to classify.

## 7. Phasing

| Phase | What | Effort | Why this order |
|---|---|---|---|
| **0** | Route DataLoss and Cancellation out of the shared inbox to named owners | Under a day, no AI | Highest risk reduction, zero cost, no dependencies |
| **1** | Ticket registry, auto-classification, routing to teams | The build | Creates the data that makes everything else measurable |
| **2** | Self-service agent on the six clean deflection topics | The build | Deflection begins once routing is trustworthy |
| **3** | Extend to the split categories once classification accuracy is proven | Later | Do not extend into commercially sensitive territory before the evidence exists |

## 8. Assumptions register

| # | Assumption | Basis | Sensitivity |
|---|---|---|---|
| 1 | 1,400 tickets/month | Client-stated | Scales linearly |
| 2 | Category shares from a 200-ticket sample | Discovery | ±3pp per category |
| 3 | 60% real deflection on eligible volume | Deliberately conservative | Every 10pp = ±55 tickets/month |
| 4 | 12 min average handling time | Author's estimate from support experience, not measured at Meridian | Every 2 min = ±11 hours/month |
| 5 | Eligible percentages per split category | Author's judgement from reading the tickets | Should be re-derived after 30 days of real classification data |

**All figures should be treated as a planning model, not a forecast.** The correct time to replace assumption 4 with a measurement is after phase 1, when the registry starts recording real handling times.

## 9. Next

`04-design/` — the data model, the apps, the routing flow, and the agent, built to implement this recommendation.
