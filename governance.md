# Governance  LabelFlow refund agent

Group 5 · Raveloarimanga Rajesy Ny Andrian · Clémence Marco

## 1. What is the one job, in one sentence?

When a merchant requests a refund for a shipping label, the agent checks eligibility against the refund policy and the package-tracking evidence, then settles, prepares or recommends the refund according to its amount, producing a decision receipt in every case.

## 2. What must it never do?

Two absolute prohibitions, both testable in the eval-set:

- never refund without package-tracking evidence attached to the decision receipt;
- never refund more than €20 on its own authority.

A third, operational: never close a case without a receipt. A decision with no receipt is a decision nobody can audit, and in a money-moving flow that is the same as no control.

## 3. What is the trust level per step?

| Step | Level | What that means |
|---|---|---|
| Read the policy and the tracking evidence | L4 | Autonomous, read-only, no money moves |
| Decide and execute a refund under €20 | L4 with after-the-fact review | Acts alone; a human reviews a sample weekly |
| Decide a refund between €20 and €100 | L2 | Prepares the decision and the receipt; a human approves before execution |
| Decide a refund over €100 | L1 | Recommends only; the human decides |
| Refuse a refund, any amount | L2 | A refusal is a commercial act: prepared by the agent, confirmed by a human |

Refusals sit at L2 deliberately. An over-refund is visible in the accounts; a wrongful refusal is visible only as churn, months later.

The sizing follows these levels rather than assuming them away: `02_sizing.xlsx` prices one supervision FTE as a cost line, and the €20–100 band is entered as 0.38 ticket-equivalents rather than full deflection, because the human approval step is never removed.

## 4. Who is accountable when it is wrong?

Accountability does not move to the vendor, the model, or the agent. *Moffatt v. Air Canada* (2024 BCCRT 149) settled the point: the airline argued its chatbot was a separate legal entity responsible for its own actions, the tribunal disagreed, and LabelFlow would be in the same position. This holds under orchestration too, which is why Assignment 2 recommends renting the resolution layer while keeping the policy, the receipt and the eval-set in-house.

- Head of Support Operations: owns the agent's decisions as if they were an agent's, owns the weekly sample review, and owns the kill-switch.
- Finance: owns the refund-leakage line and the monthly reconciliation.
- Legal: owns the wording of the receipt and of any automated refusal.
- The product owner of the agent: owns the eval-set and the re-evaluation at every model change.

## 5. What is logged, and for how long?

Every decision writes a receipt: input request, policy clause applied, tracking evidence and its timestamp, amount, decision, trust level at which it was taken, model version, and the identity of the approving human where one was required. Receipts are retained for the statutory accounting period and are the evidence base for the eval-set and for any dispute. Merchant personal data in the receipt is limited to what the decision required (GDPR minimisation); the receipt is the merchant's on request.

## 6. How does a merchant reach a human, and how do we roll back?

Rollback triggers (any one moves every case to L2):

- **Money trigger (primary):** refunded amount on agent-settled cases exceeds the pre-agent
  baseline by more than **€1,500 over a rolling 7 days** — roughly the agent's weekly net
  value (€80k/year in `02_sizing.xlsx`). Above that, leakage erases what the agent saves.
  _Guess — to be recalibrated once the 3-month refund export gives the real baseline._
- any refund executed without tracking evidence (zero tolerance);
- wrongful-refusal rate (refusals overturned after dispute) above the human baseline over
  a rolling 7 days;
- refunded amount per merchant jumping without a matching request volume (splitting signal);
- a model-version change that has not passed the eval-set replay.

---

## Assignment 1 — the six governance questions (provisional)

Status legend: ✅ answered · 🔍 to check before launch

**1. EU AI Act risk tier — Limited risk (provisional).**
The agent decides shipping-label refunds for business customers. It is not in Annex III:
no creditworthiness scoring, no access to essential services for natural persons, no
employment decision. Transparency obligations apply: merchants are told the decision was
automated (see §6).
🔍 Some merchants are sole traders (natural persons) — confirm this does not change the
tier, and check whether GDPR Art. 22 (solely automated decisions) applies to an automated
refusal. Our design already gives a human route on every decision.

**2. Whose data, on what legal basis.**
- Merchant account and contact data, refund history → **contract** (GDPR Art. 6(1)(b)):
  processing the refund the merchant asked for.
- Monitoring for errors, fraud and request splitting; the receipt audit log →
  **legitimate interest** (Art. 6(1)(f)).
- Receipt retention for the accounting period → **legal obligation** (Art. 6(1)(c)).
- Package-tracking data contains the **merchant's end-customers'** names and addresses:
  third-party data, used only to confirm delivery or return, and kept out of the receipt.
🔍 Confirm LabelFlow's role (controller vs. processor) for end-customer data in the
merchant contract; check the vendor's data location and sub-processor terms if we
orchestrate (Assignment 2).

**3. Who is accountable, and where is the human.** ✅
Accountability stays with LabelFlow (§4, _Moffatt v. Air Canada_). The human sits after
the decision under €20 (weekly sample), before it between €20 and €100, and decides
above €100 and on every refusal (§3).
🔍 Name the Head of Support Operations and the Legal owner before launch.

**4. What it must never do.** ✅
Refund without tracking evidence; refund more than €20 on its own; close a case without
a receipt (§2).

**5. Explainable to whom.**
- **Merchant:** the decision receipt, in plain language — clause applied, evidence, amount.
- **Support reviewer:** the same receipt plus the raw evidence, to confirm or overturn.
- **Finance:** daily reconciliation of refunded amounts against the baseline.
- **Auditor / Legal:** the full log — model version, trust level, approving human (§5).
🔍 Test receipt readability with 5–10 merchants (H3, Session 5).

**6. What happens to the people it cannot serve.**
Cases with missing or ambiguous tracking, non-standard carriers, policy exceptions, or
merchants who dispute: **missing evidence means escalation, never refusal.** These cases go
to a human with the full context — no worse than today's 3-day queue — and every merchant
has a one-click route to a person (§6).
🔍 Measure the escalation share on the 200 Session 4 tickets; if it exceeds ~30 %, the
agent serves too few cases to be worth it.



---

## Session 2 line — the cost-of-error line that is a legal or reputational risk

Of the seven cost lines in `02_sizing.xlsx`, the cost of error is the one that is not only money. It is priced at **€25 per wrong answer in the sub-€20 band and €55 in the €20–100 band**, and at base it absorbs **€19.2k a year, 22 % of the gross value the agent creates**. Both figures are guesses until the Session 5 eval-set exists, and both are labelled as such in the sheet.

**The line:** an automated refusal of a refund, or a refund executed on misread tracking evidence, where the decision receipt states a policy clause the agent applied incorrectly.

**Why it is not just money.** Priced as money, one wrong refusal is a €12 refund and a re-opened ticket. The real exposure is elsewhere. The receipt is a written statement by LabelFlow about its own policy, sent to a professional customer: if it misstates the policy, it is a representation the merchant can rely on, and *Moffatt v. Air Canada* confirms the company carries that liability whatever produced the text. A systematic misreading applied to thousands of receipts is not an incident, it is a pattern across the merchant base, discoverable in our own logs. Reputationally, a logistics SaaS that refuses small refunds automatically and wrongly loses merchants without ever receiving a complaint — the failure is silent on both sides of the ledger, which is why the sizing carries a daily reconciliation of refunded amounts rather than relying on complaints to surface it.

**Owner:** Head of Support Operations, accountable for the decision; Legal, accountable for the wording of the receipt and of any automated refusal; both named before launch, and the eval-set carries a dedicated wrongful-refusal slice reviewed at every model change.
	
