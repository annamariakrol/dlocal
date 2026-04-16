# Brazil: dLocal payout bank details UX — stakeholder briefing

**Audience:** Business stakeholders  
**Purpose:** Standalone read — **what we ship** under **REVPAY-6359**, **why smoother Brazil dLocal payout-bank capture matters** for doctors and payouts, and **where to find** the formal scope (Jira + request).  
**Ticket:** [REVPAY-6359](https://docplanner.atlassian.net/browse/REVPAY-6359)  
**Product request (cycle doc):** [DocPlanner/dp-payments-ai — `6359-dlocal-br-bank-form-ui/request.md`](https://github.com/DocPlanner/dp-payments-ai/blob/main/specs/features/marketplace-payment-panel/cycles/6359-dlocal-br-bank-form-ui/request.md)

**Full HTML version (GitHub Pages):** [annamariakrol.github.io/dlocal/brazil-payout-bank-briefing/briefing.html](https://annamariakrol.github.io/dlocal/brazil-payout-bank-briefing/briefing.html)

---

## At a glance — what this is & why we’re doing it

**REVPAY-6359** improves **Brazil dLocal payout bank** capture in the revenue dashboard (**first-time setup** and **payout settings**) with a **searchable bank list** aligned to dLocal’s official [Brazil payouts](https://docs.dlocal.com/docs/brazil-payouts) catalog, clearer **placeholders**, always-on **CPF/CNPJ / BRL** guidance, and **bank-specific** branch/account hints—so fewer bad details reach dLocal.

**Why now:** payout bank is a **high-friction** step: doctors must match **bank code**, **agency/branch**, and **account** to what dLocal expects. In Brazil we **repeatedly** see **tens of merchants per month** slowing down or failing that leg (support and ops see bank-validation and “account not found”-class outcomes). **REVPAY-6359** is a **focused UX upgrade** that makes the path **clearer and easier to complete**—searchable bank list, stronger guidance, and bank-specific hints aligned to dLocal’s Brazil documentation.

---

## 1. Executive summary — headline for stakeholders

**Headline:** Brazil dLocal **payout bank** is a step where **many merchants each month** **stall, retry, or drop off** because details are easy to get wrong. **REVPAY-6359** replaces a brittle free-text flow with a **guided, dLocal-aligned** experience so more doctors complete payout setup **correctly the first time**.

---

## 2. What we are delivering (short)

- **Searchable bank code** entry (autocomplete) using the **official Brazil payouts bank catalog** from dLocal’s documentation ([Brazil payouts](https://docs.dlocal.com/docs/brazil-payouts)), shown as **code — bank name**.  
- **Clear placeholders** for branch and account, aligned to how doctors read numbers on statements / banking apps.  
- **Always-visible guidance** that the account must match the **CPF/CNPJ** on file, that a **BRL** payout account is required, and that incorrect data can cause **failures**.  
- **Conditional help** for the major banks (BB, Santander, Caixa, Bradesco, Itaú, HSBC Brasil, Inter) plus a **generic** fallback for all others — same behaviour in **onboarding** and **payout settings**.

**Interactive prototypes:** [Onboarding flow](https://annamariakrol.github.io/brazil-payout-bank-prototype/) · [Payout settings](https://annamariakrol.github.io/brazil-payout-bank-prototype/payouts-settings.html)

---

## 3. Why this is a major business blocker (plain language)

1. **Activation is not “KYC only”** — for payouts, dLocal must accept **bank account** data. If the doctor enters the wrong **bank code** or mis-enters **agency / account** structure, dLocal cannot treat the account as valid — the doctor stays in a **broken or blocked payout path** until data is fixed.  
2. **Errors are expensive at scale** — support tickets, repeated submissions, and calendar time while the doctor cannot be paid create **trust and churn risk** with high-value clinics.  
3. **The fix is disproportionately cheap vs the downside** — this is primarily **doctor-facing UX + copy + a maintained bank list**, not a new payment rail.

---

## 4. References

- Jira: [REVPAY-6359](https://docplanner.atlassian.net/browse/REVPAY-6359)  
- Formal scope / ACs: [request.md in dp-payments-ai](https://github.com/DocPlanner/dp-payments-ai/blob/main/specs/features/marketplace-payment-panel/cycles/6359-dlocal-br-bank-form-ui/request.md)  
- dLocal public reference: [Brazil payouts](https://docs.dlocal.com/docs/brazil-payouts)
