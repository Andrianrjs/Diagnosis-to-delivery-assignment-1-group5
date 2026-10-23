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


## 5. Agent selection: the four questions

**1. High frequency?**

Yes. ~7,500 refund requests a month (30 % of 25,000 tickets), i.e. ~350 per working day.
The work is done every hour, not every quarter — the volume justifies building a system
rather than improving a checklist.

**2. Bounded decision space?**

Yes. The agent has three tools (read refund policy, read tracking status, execute refund)
and four possible outcomes: refund, prepare-for-approval, recommend-only, refuse with a
reason. Nothing else. Every outcome ships with a decision receipt (rule applied, evidence,
amount), which is itself the audit surface.

**3. Measurable success?**

Yes, in one sentence per ticket: the refund decision matches what a trained agent would
have decided on the same evidence, settled in under 1 hour, with a receipt a merchant can
read. Four countable business metrics follow: share of sub-€20 cases settled without a
human, time-to-resolution, over-refund rate, and wrongful-refusal rate (refusals
overturned after merchant dispute).

**4. Recoverable failure?**

Partly, and this is the weak point of P4. A wrongful refusal is recoverable: the merchant
re-opens, a human overturns it. A wrongful refund is **not** recoverable case by case
(the money is gone) and it fails silently — nobody opens a ticket to complain about being
paid. Bounded per case by the €20 L4 ceiling, but a systematic error, or a merchant who
discovers the threshold and splits requests, compounds before anyone notices.

**Consequence for the design, not just the paperwork:**
- the €20 autonomous ceiling and the tracking-evidence requirement are the two guardrails
  that make the failure bounded;
- a daily reconciliation of refunded amount vs. the pre-agent baseline is the detector for
  the silent failure mode (S7 monitoring);
- the rollback plan is a kill-switch that drops the agent to L2 (prepares, human approves)
  on every case, not an off switch — turning it off returns the queue to a 3-day wait.

---

## 6. Press release + the three "would have to be true"

### Press release (working backwards, dated at launch)

**LabelFlow settles small shipping-label refunds in under an hour**

_Paris, 30 June 2027_ — LabelFlow, the logistics SaaS used by 15,000 e-commerce merchants,
today announced that shipping-label refund requests under €20 are now settled in under an
hour instead of three days. A refund agent reads the refund policy and the package-tracking
evidence together, and sends the merchant a decision receipt — the rule applied, the
evidence, the amount — that is both the answer and the audit trail. Under €20 it settles
on its own and a human reviews a sample afterwards; above €20 a human approves or decides;
it never refunds without tracking evidence. "I used to wait three days for a €12 refund
and an answer I couldn't check," said Camille R., who runs an online homeware store in
Lyon. "Now I get the money and the reason in the same message, before lunch."

_(Merchant quote illustrative — working-backwards exercise.)_

### The three "would have to be true"

| # | Assumption | What would make it false | How we test it | Session |
| --- | --- | --- | --- | --- |
| 1 | Eligibility for sub-€20 refunds is decidable from the policy text + tracking data alone, on at least 70 % of cases | Decisions depend on context living nowhere in those two sources (merchant history, commercial gestures, verbal arrangements) | Manual audit of 200 historical sub-€20 refund tickets: can the recorded decision be reconstructed from policy + tracking alone? | Session 4 |
| 2 | The agent's over-refund rate does not exceed the current human over-refund rate | The agent is systematically more generous on ambiguous evidence, or merchants learn to split requests under the €20 line | Replay the agent on the same 200 tickets and compare with the human outcome (S4); then monitor refunded amount per merchant per week for splitting (S5) | Session 4 → 5 |
| 3 | A merchant who receives an automated decision receipt does not re-open the ticket | The receipt is unreadable, or an automated refusal is less acceptable than a human one | Re-open rate and CSAT on agent-settled vs. human-settled cases, measured the same way before and after | Session 5 |

---

## 7. The do-nothing baseline

Do-nothing is not zero, and it is not static.

**What the current process costs, today**

| Line | Value | Source type |
|---|---|---|
| Refund requests per month | ~7,500 (30 % × 25,000) | Measured (platform logs) |
| Share of requests under €20 | ~65 %, i.e. ~4,875/month | **Guess — to be measured on a 3-month export of refund amounts** |
| Processing cost per ticket | ~€5 in `01_problem_statement.md`; €7 at base in `02_sizing.xlsx` | Estimate / analogy, to be measured on a 2-week sample |
| Annual processing cost, sub-€20 slice | ~4,875 × €7 × 12 ≈ **€410k / year** | Derived |
| Annual processing cost, all refund requests | ~7,500 × €7 × 12 ≈ **€630k / year** | Derived |
| Wait per request | ~3 days | Measured (support log), to confirm |

**What do-nothing does on its own**

The baseline is not frozen. Without any AI project, the same cost falls a few percent a
year: agents get faster on a repetitive task, the policy documentation improves, macros
and canned responses accumulate. We size that at 2–5 % a year (analogy), which
`02_sizing.xlsx`` turns into **+€84k a year** of value that accrues without doing anything.

Do-nothing also carries no new risk: no over-refund leakage from a model, no vendor
dependency, no re-evaluation burden at every model change, and no legal exposure of the
kind established in *Moffatt v. Air Canada* (2024 BCCRT 149), where the company was held
liable for its chatbot's answer.

**The real competitor is not do-nothing**

The hurdle is the next-best use of the same team: a **self-serve refund-status page for
merchants**, which removes the "where is my refund" tickets without deciding anything.
Sized at **€180k a year** in `02_sizing.xlsx`. That figure is posed rather than built
bottom-up, and deserves the same discipline before it decides anything — but on today's
scope it beats the agent, which is the finding of Assignment 2.
