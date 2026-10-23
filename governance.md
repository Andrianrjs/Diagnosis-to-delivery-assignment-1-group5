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

Every agent decision carries a one-click escalation that routes to a human with the full case context, and the merchant is told the decision was automated. A merchant may not be left with no route to a person: that is the omission the Klarna case exposed, where the customers who could not reach a human had no line in the business case.

Rollback is a downgrade, not a shutdown: the kill-switch moves every case to L2 (the agent prepares, a human approves). Turning the agent off entirely returns the queue to a 3-day wait, so it is the last resort, not the first.

Rollback triggers: over-refund rate above the human baseline over a rolling 7 days; any refund executed without tracking evidence; refunded amount per merchant jumping without a matching request volume (the splitting signal); a model-version change that has not passed the eval-set replay.

---

## Session 2 line — the cost-of-error line that is a legal or reputational risk

Of the seven cost lines in `02_sizing.xlsx`, the cost of error is the one that is not only money. It is priced at **€25 per wrong answer in the sub-€20 band and €55 in the €20–100 band**, and at base it absorbs **€19.2k a year, 22 % of the gross value the agent creates**. Both figures are guesses until the Session 5 eval-set exists, and both are labelled as such in the sheet.

**The line:** an automated refusal of a refund, or a refund executed on misread tracking evidence, where the decision receipt states a policy clause the agent applied incorrectly.

**Why it is not just money.** Priced as money, one wrong refusal is a €12 refund and a re-opened ticket. The real exposure is elsewhere. The receipt is a written statement by LabelFlow about its own policy, sent to a professional customer: if it misstates the policy, it is a representation the merchant can rely on, and *Moffatt v. Air Canada* confirms the company carries that liability whatever produced the text. A systematic misreading applied to thousands of receipts is not an incident, it is a pattern across the merchant base, discoverable in our own logs. Reputationally, a logistics SaaS that refuses small refunds automatically and wrongly loses merchants without ever receiving a complaint — the failure is silent on both sides of the ledger, which is why the sizing carries a daily reconciliation of refunded amounts rather than relying on complaints to surface it.

**Owner:** Head of Support Operations, accountable for the decision; Legal, accountable for the wording of the receipt and of any automated refusal; both named before launch, and the eval-set carries a dedicated wrongful-refusal slice reviewed at every model change.
	
