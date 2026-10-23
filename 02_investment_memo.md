# Investment memo — LabelFlow refund agent

Group 5 · Raveloarimanga Rajesy Ny Andrian · Clémence Marco · 23 October 2026
Base case: `02_sizing.xlsx`, Decision tab, Base column. The sheet and this memo use the same base case.

**NO-GO on building the refund agent in-house. CONDITIONAL GO on orchestrating it with a vendor, conditional on in-scope volume reaching ~33,000 deflected tickets a year — which means widening scope beyond sub-€20 refunds, not improving the agent.**

## How much?

Expected value **−€73k a year**, against **+€84k** for doing nothing and **+€180k** for the next-best project, a self-serve refund-status page for merchants. The three scenarios are −€273k, −€84k and +€148k, weighted 0.25 / 0.50 / 0.25.

The agent works. It deflects 12,592 tickets in year 1 and each deflected ticket is worth €5.38 net of AI cost and of the expected cost of a wrong answer. The problem is that 12,592 tickets is 4.2 % of LabelFlow's ticket base, and the seven cost lines do not shrink with scope: €164k a year of integration, evaluation, supervision, monitoring and re-evaluation sit against €80k of net variable value. A money-moving agent carries a fixed base sized for the whole support operation and is pointed at a sliver of it.

## How sure?

Not very, and the uncertainty is concentrated in one unmeasured number. The split of refund requests by amount band (Inputs rows 14–16) is a guess: we assume 65 % of refund requests are under €20. A three-month export of refund amounts settles it and moves the whole case. Nothing else in the sheet deserves a day of work before that export exists.

Two further caveats we did not hide. The €5 per ticket in our own `01_problem_statement.md` was an unverified estimate at the bottom of the Session 2 range; this sheet uses €7 at base and keeps €5 as the pessimistic column. And the hurdle, €180k, is posed rather than built bottom-up — it has not been through the discipline we applied to our own project, and it should be before it decides anything.

## What if we're wrong?

The sensitivity table reads **"below do-nothing" at every deflection rate from 30 % to 75 %**. That is the finding: deflection is not the assumption that flips this decision, and a better agent does not rescue it. The binding constraint is volume in scope. Break-even against do-nothing is roughly **33,000 to 44,000 deflected tickets a year** — 33,000 if do-nothing is scoped like-for-like to refund tickets only, 44,000 on the sheet's own formula, which credits do-nothing across the whole ticket base while our agent touches 30 % of it. Either way, against 12,592 at base, the project needs between 2.6× and 3.5× its current scope.

The failure mode that would hurt is not a missed deflection target. It is an over-refund that fails silently: nobody opens a ticket to complain about being paid. That is why the sheet carries one supervision FTE and a daily reconciliation of refunded amounts against the pre-agent baseline, and why refund requests over €100 are excluded from scope altogether — 40 % automatable at best, the agent only recommends, and the unrecoverable amount is the largest.

## Compared to what?

Against do-nothing (+€84k), the project loses by €157k. Against the refund-status page (+€180k), it loses by €253k. On today's scope the billing-status page is simply the better use of the same team, and it carries none of the liability established in *Moffatt v. Air Canada* (2024 BCCRT 149).

The comparison that changes is the one inside the project. At 12,592 resolutions a year, a vendor at €0.92 per resolution (Intercom Fin, from $0.99, checked 23 October 2026) costs **€11.6k a year** against **€96.9k** to run the equivalent ourselves — integration amortised, evaluation and re-evaluation. Buying the resolution layer is cheaper by **€85k a year** at our volume, and that gap closes only above roughly 100,000 resolutions a year.

## Build, buy or orchestrate

**Orchestrate** (weighted 4.25, against 3.40 for buy and 2.60 for build). We rent the resolution layer and keep the refund policy, the tracking evidence, the decision receipt and the eval-set in-house — because liability stays with LabelFlow whatever we choose, and a pure buy puts the decision logic where we cannot audit it. What would change this: if the refund-amount export shows the in-scope volume is above ~100,000 resolutions a year, or if a vendor refuses to let the refund execution stay on our side, build returns to the table.

**Next step, before any build decision:** the three-month refund-amount export, and the Session 4 prototype measuring deflection on 200 historical sub-€20 tickets. The prototype is still worth running — it tells us whether orchestration is viable at all — but it is no longer what decides the project.
