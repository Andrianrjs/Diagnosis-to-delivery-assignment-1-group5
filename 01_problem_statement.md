# Problem Statement

## One-sentence problem (quantified)

Support agents at **LabelFlow**, a B2B e-commerce logistics SaaS (15,000 merchants,
25,000 tickets/month, of which ~7,500 are shipping-label refund requests), manually
process refunds that are often under €20 — cases where the processing cost (~€5/ticket)
exceeds the amount refunded; today each request waits ~3 days and is handled by a human.
Success looks like: refunds under €20 settled automatically in under 1 hour, cost per
case cut by 50%, with no increase in the over-refund rate.

## Where each number comes from

| Number | Value | Source type |
|---|---|---|
| Merchants | 15,000 | Measured (platform logs) |
| Tickets per month | 25,000 | Measured (platform logs) |
| Share of refund tickets | 30% → ~7,500/month | Measured (calculated: 30% × 25,000) |
| Wait per request | ~3 days | Measured (support log) *(to confirm vs. estimate)* |
| Processing cost per ticket | ~€5 | Estimate / analogy (human support-ticket benchmark), to be measured on a 2-week sample |
| Target resolution time | < 1 hour | Target set by the team (not observed) |
| Target cost reduction | −50% | Target set by the team (not observed) |
| Over-refund rate | must not increase | Guardrail metric |

## Breakdown (the 5 required building blocks)

- **Who:** LabelFlow support agents
- **Pain / when:** manually handling refund requests under €20
- **Quantified cost:** ~€5/ticket processing cost exceeds refund value; ~7,500 tickets/month
- **Do-nothing baseline:** ~3-day wait, every case routed to a human
- **Measurable success:** < 1h resolution, −50% cost per case, no rise in over-refunds

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