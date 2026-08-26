# Meridian Systems — Discovery: Support Ticket Taxonomy

> Reference engagement. Meridian Systems is a fictional client. The analysis method is real.

## 1. Why this document exists

Meridian's Head of Customer Success asked for an AI chatbot on the help centre.

Before agreeing to build one, I needed to answer a question nobody at Meridian could: **what do customers actually contact us about?** They had 1,400 tickets a month and no categorisation. Agents triaged by eye from a shared inbox. There was no way to know which questions repeated, which were expensive, or which would be safe to automate.

This is the artifact that answers it. Everything downstream — the data model, the routing rules, the agent's scope, the evaluation test set — comes from here.

## 2. Method

- Sample of 200 tickets drawn across ten weeks (1 June – 7 August 2026), covering all customer plan tiers
- Each ticket read and assigned to a single category by its **primary** request
- Categories derived from the tickets themselves, not from a predefined list
- Each ticket also tagged easy / medium / hard, where difficulty means *how hard it is to classify and route correctly*, not how hard it is to resolve
- Raw data: `data/tickets-sample.csv`

**Sampling caveat:** 200 of roughly 3,200 tickets in the period, sampled for spread rather than randomly. Seasonal effects (month-end, audit season) are visible but not measured. Stated so the numbers are not over-read.

## 3. The taxonomy

Fourteen categories. Every ticket in the sample has a home.

| # | Category | Count | Share | Needs account data? | Cost of a wrong answer |
|---|---|---|---|---|---|
| 1 | Login | 20 | 10.0% | Sometimes | Low |
| 2 | Reports | 17 | 8.5% | Sometimes | Low |
| 3 | HowTo | 17 | 8.5% | No | Low |
| 4 | Scanner | 16 | 8.0% | No | Low |
| 5 | Seats | 15 | 7.5% | Yes | Medium |
| 6 | Integration | 15 | 7.5% | Yes | Medium |
| 7 | DataImport | 15 | 7.5% | Sometimes | Medium |
| 8 | Billing | 15 | 7.5% | Yes | **High** |
| 9 | Performance | 15 | 7.5% | Sometimes | Low |
| 10 | FeatureRequest | 15 | 7.5% | No | Low |
| 11 | Other | 14 | 7.0% | Varies | Varies |
| 12 | StockDiscrepancy | 12 | 6.0% | Yes | Medium |
| 13 | DataLoss | 8 | 4.0% | Yes | **Very high** |
| 14 | Cancellation | 6 | 3.0% | Yes | **Very high** |

Difficulty spread: 57 easy, 77 medium, 66 hard.

## 4. What each category actually contains

**Login (10.0%)** — password resets, lockouts, MFA problems, SSO failures. Mostly identical and self-serviceable. But SSO certificate expiry and login redirect loops sit in the same bucket and are engineering problems.

**Reports (8.5%)** — exports failing, blank PDFs, scheduled reports not arriving, column and formatting requests. A wide range: some are how-to questions in disguise, some are bugs.

**HowTo (8.5%)** — "where do I set a reorder point", "how do I archive products", "how do returns work". Product knowledge questions. No account data needed. The purest deflection candidate.

**Scanner (8.0%)** — pairing, charging, firmware, keyboard mode. Hardware troubleshooting that follows repeatable steps. Two outliers: cold-store failures and MDM management, which are genuinely technical.

**Seats (7.5%)** — adding, removing, transferring users; seat counts not matching. Requires account data, so it is assistable rather than deflectable.

**Integration (7.5%)** — accounting sync failures, webhooks, API errors, duplicate orders. Almost always needs investigation. Poor deflection candidate.

**DataImport (7.5%)** — file format, validation errors, imports that overwrite or duplicate. Template and format questions are deflectable; imports that damaged data are not.

**Billing (7.5%)** — invoice copies, VAT numbers, address changes, double charges, refunds, price increases. Split personality: some are trivially deflectable, some are commercially sensitive.

**Performance (7.5%)** — general slowness, timeouts, month-end degradation. Usually needs an engineer to confirm whether it is the customer's environment or Meridian's.

**FeatureRequest (7.5%)** — dark mode, multi-currency, purchase orders, forecasting. These need acknowledgement and logging, not resolution.

**Other (7.0%)** — security questionnaires, DPAs, SLA questions, GDPR requests, "everything is broken", account management. A genuine mixed bag, and it should stay that way rather than being forced into false categories.

**StockDiscrepancy (6.0%)** — counts not matching reality. Requires account data and often reveals a process problem rather than a software one.

**DataLoss (4.0%)** — products disappeared, counts reset to zero, adjustments missing, work lost mid-count. Small in volume, largest in consequence.

**Cancellation (3.0%)** — downgrades, non-renewals, competitor offers. Smallest category. Highest commercial value per ticket.

## 5. Findings

### Finding 1 — the repetitive volume is real, but smaller than assumed

Ana's assumption was that "most" tickets are repetitive. The data says the four cleanly repetitive categories — HowTo, Scanner, Login, and the format-question portion of Reports and DataImport — account for roughly **34%** of volume, not "most".

That is still substantial: around 475 tickets a month. But a project sold on "we'll deflect most of your tickets" would fail against its own promise.

### Finding 2 — the categories nobody expected are the expensive ones

DataLoss and Cancellation together are only 7% of volume — 14 tickets in the sample. They are also the only two categories where a wrong or slow answer plausibly costs a customer. Both are currently triaged by the same eye, in the same inbox, at the same speed as a password reset.

**That is the finding worth paying for.** It is not an automation opportunity. It is a routing failure that exists today and can be fixed immediately, regardless of whether any AI is built.

### Finding 3 — categories are not homogeneous

Billing is the clearest example. "Send me a copy of May's invoice" and "we were charged twice, refund us" are the same category and completely different problems. Any design that routes by category alone will send commercially sensitive tickets down an automated path.

**Design consequence:** routing must be decided per ticket, not per category. That single conclusion shapes the entire build.

### Finding 4 — one in three tickets is hard to classify

66 of 200 tickets were tagged hard: vague wording ("something isn't right with the numbers"), two problems in one message ("password reset and also add two seats"), or emotional tone concealing a simple request ("nobody answered my ticket from Tuesday, just need my export fixed").

This sets a realistic expectation before anything is built: **automated classification will not be reliable on a third of the volume**, and the design has to assume that rather than hope otherwise.

### Finding 5 — the surprise

Login is the largest category at 10%, and the assumption was that it was entirely self-service. Reading them, four of the twenty are SSO or session-management problems affecting entire teams — engineering issues sitting inside the category everyone assumed was trivial.

A chatbot trained to answer "Login" questions with a password reset link would give the wrong answer to those four, to a whole team, during an outage.

## 6. What this changes about the brief

Ana asked for a chatbot on the help centre.

What the data supports is narrower and more useful:

1. **Fix routing first.** DataLoss and Cancellation need to leave the shared inbox today. This costs nothing and does not need AI.
2. **Deflect a defined 34%**, not "most" — and define it by ticket, not by category.
3. **Assume a third of tickets classify poorly** and design the human hand-off as a first-class feature rather than a fallback.

## 7. Assumptions register

- Meridian Systems, its customers, staff and ticket volumes are constructed for this reference engagement
- The 200 tickets are authored, not real customer data
- Category shares reflect the sample, not a measured population
- Handling times used in the next phase are estimates, labelled where they appear
- No customer data of any kind was used

## 8. Next

`03-opportunity-analysis.md` — sorting these fourteen categories into deflect, assist and human-only, with the arithmetic behind each decision.
