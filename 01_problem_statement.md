# Problem Statement

## One-sentence problem (quantified)

Support agents at **LabelFlow**, a B2B e-commerce logistics SaaS (15,000 merchants,
25,000 tickets/month, of which ~7,500 are shipping-label refund requests), manually
process refunds that are often under €20 — cases where the processing cost (~€5/ticket)
exceeds the amount refunded; today each request waits ~3 days and is handled by a human.
Success looks like: refunds under €20 settled automatically in under 1 hour, refund cost
per case (processing cost + refund leakage) cut by 50%, with no increase in either
over-refunds or wrongful refusals.

## Where each number comes from

| Number | Value | Source type |
| --- | --- | --- |
| Merchants | 15,000 | Measured (platform logs, P4 brief) |
| Tickets per month | 25,000 | Measured (platform logs, P4 brief) |
| Share of refund tickets | 30% → ~7,500/month | Measured (P4 brief), volume calculated: 30% × 25,000 |
| Wait per request | ~3 days | Measured (support log, P4 brief) |
| Processing cost per ticket | ~€5 | Analogy (human support-ticket benchmark) — to be measured on a 2-week sample |
| Share of clear-cut sub-€20 cases | ~70% | Guess — to be tested on 200 historical tickets (Session 4) |
| Target resolution time | < 1 hour | Target set by the team (not observed) |
| Target cost reduction | −50% | Target set by the team (not observed) |
| Counter-metric 1: over-refund rate | must not increase | Guardrail — measured on a weekly audit sample of decision receipts |
| Counter-metric 2: wrongful-refusal rate | must not increase | Guardrail — refusals overturned after merchant dispute |

## Breakdown (the 5 required building blocks)

- **Who:** LabelFlow support agents
- **Pain / when:** manually handling refund requests under €20
- **Quantified cost:** ~€5/ticket processing cost exceeds refund value; ~7,500 tickets/month
- **Do-nothing baseline:** ~3-day wait, every case routed to a human
- **Measurable success:** < 1h resolution, −50% refund cost per case, no rise in
over-refunds **or** wrongful refusals

## Problem Tree (5 Whys)

**Symptom:** Merchants wait ~3 days for small refunds; support is buried under
~7,500 refund tickets/month.

1. Why do these small refunds cost more than they're worth?
→ Each one is handled manually (~€5/ticket).
2. Why manually?
→ Eligibility must be checked case by case: refund policy **+** tracking data.
3. Why a human every time?
→ The policy is long and full of exceptions, and tracking lives in a separate
system — nothing links the two automatically.
4. Why is nothing linking them?
→ The eligibility decision was never codified; it's tribal knowledge held by agents.
5. Why was it never codified?
→ Ticket volume grew faster than the process; turning the policy into a decision
system was never prioritized.

**Root cause:** The refund-eligibility decision is neither codified nor tooled — it
relies on manual interpretation of a complex policy cross-checked against scattered data.

**Fix (and its nature):**

- **Rule** for clear-cut cases: amount < threshold + tracking confirms + within window → refund.
- **AI/agent** for the ambiguous slice: edge cases, unstructured tracking evidence, and
producing the "decision receipt."
- **Process:** L4/L2/L1 trust thresholds + escalation.
- **Docs:** clarify the policy upstream to reduce ambiguity.

## Is AI the right tool?

**1. Is there a rule that solves 80%?**

Partly. Clear-cut cases (amount < threshold + tracking confirms delivery/return +
within window) can be handled by a deterministic rule (~70% of sub-€20 cases — guess,
to be tested in Session 4). But an ambiguous remainder (unclear tracking, policy
exceptions, non-standard evidence) resists rules — and even clear cases require reading
tracking evidence and writing a decision receipt. A rule handles the easy majority, not all.

**2. Prediction/generation or organisational decision?**

A mix. Primarily an organisational decision (apply policy → rule on eligibility), but it
requires understanding unstructured evidence and generating a readable decision receipt.

**3. Do we have the data?**

Yes: the refund policy text, tracking data, and historical tickets with outcomes
(25,000/month, ~7,500 refunds) to build, evaluate and measure against. The receipts also
form an audit log.

**4. What does one error cost?**

Two opposite failures: over-refunding (silent money leakage) and wrongly refusing (churn,
lost trust). Bounded per case under the L4 threshold, but costly if systematic. Cf.
Moffatt v. Air Canada (2024 BCCRT 149): the company was held liable for its chatbot's
wrong answer — so an error carries financial AND legal cost.

**Chosen level: Agent.**

- Not a rule alone: ~30% ambiguous cases + evidence interpretation + receipt generation.
- Not classical ML: it can't read policy/evidence text or generate explanations.
- Not GenAI alone (the level below): a chatbot answers; our task must _act_ — execute the
refund, chain tool calls (policy + tracking), and respect L4/L2/L1 trust thresholds.
A GenAI responds; an agent acts.

## Agent selection check (4 questions)

- **High frequency:** ✅ ~7,500 refund requests/month (~250/day) — measured (P4 brief).
- **Bounded decision space:** ✅ Three outcomes only (refund / refuse / escalate), two inputs
(refund policy + tracking data), and the amount sets the trust level (L4 < €20, L2 €20–100, L1 > €100).
- **Measurable success:** ✅ Every case has a checkable outcome logged in its decision receipt:
refund cost per case, time to resolution, over-refund rate, wrongful-refusal rate.
- **Recoverable failure:** ⚠️ Yes per case — an L4 error is capped at €20, a wrongful refusal
can be reopened, an over-refund is caught in post-hoc review. But a *systematic* error is
silent — so this holds only with weekly receipt audits and a money-based rollback criterion.