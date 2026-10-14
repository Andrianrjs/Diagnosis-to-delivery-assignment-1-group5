Team: Raveloarimanga Rajesy Ny Andrian (GIT:Andrianrjs) - Clemence Marco (GIT:clemencemarco-prog) 

Company: LabelFlow — a SaaS platform for e-commerce logistics (shipping labels + returns). 15,000 merchants, 25,000 tickets/month.

What the agent does: When a merchant requests a refund for a shipping label, the agent checks whether it's justified (policy + package tracking), then refunds it if the amount is small or hands it off to a human if the amount is large. Each time, it produces a "decision receipt" (rule applied, evidence, amount).

How far the agent acts on its own:

Under €20 → acts on its own; a human reviews afterwards.
€20–100 → prepares the decision; a human approves before it's done.
Over €100 → only recommends; the human decides.
What it must never do: refund without package-tracking evidence, or refund more than €20 on its own.