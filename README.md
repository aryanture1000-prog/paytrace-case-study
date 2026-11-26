# PayTrace: product case study

PayTrace ([paytrace.in](https://paytrace.in)) reconciles courier and cash-on-delivery remittances for Indian D2C brands, so a brand can see which rupees were short-paid and chase them. I am the founder, the product owner and the builder. I define the requirements, pricing and metrics, and build with AI coding tools. The source code is private, so this repo is the product thinking behind it.

**Status:** live and early stage. I am onboarding D2C brands with real COD volume. This write-up makes no revenue or customer claims. I will add real results here once brands give permission.

## The problem

When a customer pays cash on delivery, the courier collects it and passes it to the brand later, in batches, with deductions. Most brands check the payout total. Very few check it order by order, so a short payment, a missing remittance or a returned parcel counted wrongly goes unnoticed. The money is owed, but nobody is looking for it.

## Who it is for

Finance and operations teams at D2C brands with real COD volume, and the agencies and accountants who run books for several brands at once. The pricing tiers follow that split.

## What it does

A brand uploads the courier remittance file and its order export. PayTrace matches them at order (AWB) level across Delhivery, Shiprocket, XpressBees, Shadowfax and Ekart, and shows rupee shortfalls per order. Each shortfall can then be tracked from flagged to disputed to recovered or written off, and the product drafts the dispute letter. There is nothing to integrate for a solo user, because it works from files the brand already has.

## Product decisions and why

**Files first, integrations later.** A brand already has the courier files. Asking it to connect systems before it has seen any value is a barrier. Integration and automation are reserved for the Enterprise tier, where a team has reason to want it.

**Show the rupees before asking for money.** A 7-day trial with phone OTP and no card, and the first charge only at the end of the trial. The product's pitch is a number, so the number comes first.

**Tiers follow the buyer, not a feature checklist.** Solo is one brand. Pro is built for agencies: several brands, white-label exports, recovery tracking and an API. Enterprise is custom, with automation, team roles and client portals.

**Yearly billing with a discount that rises with the tier:** 17% on Solo, 21% on Pro, and 25% on the displayed Enterprise price. Yearly is a real recurring subscription that renews automatically, which was an explicit decision, not a default. Anyone about to give up prepaid time by switching down is warned first, naming the date.

**Change the limits without breaking existing customers.** When I lowered the plan limits, existing subscribers kept their original limits. The limits are stored on the account and cleared only on a real plan change. Clearing them on every routine renewal would have silently taken limits away from people who had done nothing, which I caught before writing the code.

**Never advertise what is not enforced.** I reworded "Cancel within 7 days", which read like a refund guarantee that did not exist, to "No charge if you cancel during your trial". I left single-device sign-in and a 5-seat team feature off the pricing page until they were built. There are no testimonials, because I have no real ones yet.

## How I checked quality

I audited the product against the promises on its own pricing page. Four parallel code audits, one per tier plus billing, were cross-checked against each pricing bullet, and every finding was ranked by how much it would hurt a real customer.

What that found, and what I fixed:
- A headline Enterprise feature, scheduled automation, never ran, because the scheduler was never configured in the deployment.
- Audit history was broken for runs made after a database change.
- The API docs contained sample code that would have failed if copied, plus a wrong error code and an outdated limit.
- An onboarding audit found that "Continue with Solo" was rejected for every verified trial user, and that closing a Pro checkout blocked every retry.

Each fix has a regression test. The suite has about 18 smoke-test files, and I ran a browser sweep of roughly 1,000 clicks across 8 account states.

## Go-to-market

Cold LinkedIn connections were not converting, so I replaced them. The plan leads with the product's own output, a free audit, instead of a pitch.
1. Free audits for 10 brands, to turn three of them into permissioned case studies.
2. Answer real questions where D2C operators already ask them, such as weight disputes and missing COD remittances, and mention PayTrace only when it is the fix.
3. Free tools that rank in search: a volumetric weight calculator and a COD leakage calculator.
4. Agency and accountant partners, since one agency can bring five to fifteen brands.

I defined the measures in advance: audits offered, accepted, trials started, trial to paid, and rupees found per audit. If fewer than 3 of 10 audit offers are accepted, the problem is the offer or the targeting, not the channel. See [docs/go-to-market.md](docs/go-to-market.md).

## AI in the product

I designed an LLM-assisted column classifier for messy courier files and deliberately held it back from production. See [docs/llm-column-classifier-spec.md](docs/llm-column-classifier-spec.md).

## What is still open

Plain gaps I know about: a team-seat invite flow for Pro is not built yet, a cancelled subscriber cannot resume in the app, and the column-mapping step has no way to correct a wrong guess. Each is on the list, in that order of customer impact.

## Stack

A static frontend, a Cloudflare Worker backend, Supabase for data and auth, Razorpay for subscriptions, phone OTP sign-in, and Resend for email.
