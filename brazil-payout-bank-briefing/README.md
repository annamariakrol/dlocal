# Brazil: dLocal payout bank details UX — stakeholder briefing

**Audience:** Business stakeholders  
**Purpose:** Standalone read — **why bank capture UX is a merchant-activation blocker** for Brazil dLocal, what we are shipping, and **scale context** (onboarding volume) comparable in spirit to the Mexico dual-wallet evidence pack  
**Ticket:** [REVPAY-6359](https://docplanner.atlassian.net/browse/REVPAY-6359)  
**Product request (cycle doc):** [DocPlanner/dp-payments-ai — `6359-dlocal-br-bank-form-ui/request.md`](https://github.com/DocPlanner/dp-payments-ai/blob/main/specs/features/marketplace-payment-panel/cycles/6359-dlocal-br-bank-form-ui/request.md)

**Full HTML version (GitHub Pages):** [annamariakrol.github.io/dlocal/brazil-payout-bank-briefing/briefing.html](https://annamariakrol.github.io/dlocal/brazil-payout-bank-briefing/briefing.html)

---

## At a glance — what this is & why we’re doing it

**REVPAY-6359** improves **Brazil dLocal payout bank** capture in the revenue dashboard (**first-time setup** and **payout settings**) with a **searchable bank list** aligned to dLocal’s official [Brazil payouts](https://docs.dlocal.com/docs/brazil-payouts) catalog, clearer **placeholders**, always-on **CPF/CNPJ / BRL** guidance, and **bank-specific** branch/account hints—so fewer bad details reach dLocal. On BroadDB we still see **dozens of stuck or rejected bank cases per month** (e.g. CRM **`REJECTED_ACCOUNT_NOT_FOUND`** on `crm_marketplace_payment_account.eventually_due` and marketplace-account **`bank_account_status = REJECTED`**—often in the **~20–40/month** range in recent months depending on definition; internal query logs: `2026-04-16_12-23-06_br-dlocal-REJECTED_ACCOUNT_NOT_FOUND-monolith` and `2026-04-16_12-10-02_br-dlocal-bank-account-rejected`). Doctors often **stall or abandon** the step because **bank codes and branch/account formats** are hard to remember when the UI is mostly free text—this change targets that friction directly.

---

## 1. Executive summary — cohort scale (marketplace-account)

On **DocPlanner BroadDB** (`zl_marketplaceaccount_br_broad`), among **dLocal** accounts created since **2025-09-01**, **225 / 4,325 (~5.2%)** currently have **`bank_account_status = REJECTED`** — i.e. dLocal did **not** accept the payout bank account (via webhooks). **220** of those rows are still **`status = active`** at marketplace-account level: doctors who are **not payout-unblocked** on the bank leg despite an otherwise active account. That order of magnitude turns **REVPAY-6359** from “nice UX” into a **measurable activation / payout friction** problem at BR scale.

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
2. **Errors are expensive at scale** — support tickets, repeated form submissions, and calendar time while the doctor cannot be paid create **trust and churn risk** with high-value clinics.  
3. **The fix is disproportionately cheap vs the downside** — this is primarily **doctor-facing UX + copy + a maintained bank list**, not a new payment rail.

---

## 4. Data evidence (BroadDB — Brazil marketplace-account)

**Environment:** DocPlanner **BroadDB**, database **`zl_marketplaceaccount_br_broad`**, table **`account`**.  
**Cohort:** `type = 'd_local'`, `deleted_at IS NULL`, `created_at >= '2025-09-01'` (**no upper bound** on `created_at` — “from 1 Sep 2025 through the DB snapshot”).  
**Query log (internal):** `Database structure with AI/queries/2026-04-16_12-10-02_br-dlocal-bank-account-rejected/` — `query.sql` + `result.md`.  
**Run date (snapshot):** 2026-04-16 — `bank_account_status` values below are **as observed on that date**, not a count of rejection *events* inside a month.

### 4.1 dLocal bank account `REJECTED` (payout bank not accepted)

**Table scope:** same cohort as §4; **date range on membership** = account `created_at` **≥ 2025-09-01**; **status columns** = **point-in-time on 2026-04-16**.

| Metric | Value |
|--------|------:|
| Accounts in cohort | **4,325** |
| `bank_account_status = 'REJECTED'` | **225** (**~5.2%**) |
| Of those, `account.status = 'active'` | **220** |
| `bank_account_status = 'ACTIVE'` | 3,241 |
| `bank_account_status` IS NULL | 854 |

`bank_account_status` is maintained from **dLocal webhooks** (`BankAccountStatusUpdatedDLocalMessage` path in **marketplace-account-app**). **`REJECTED`** means dLocal did not validate/accept that bank account for payouts — **drivers are not only typos** (fraud / policy / other provider reasons can appear here), but this is the **closest stable operational metric** we expose for “doctor submitted bank details; dLocal said no”.

### 4.2 New accounts per month (same cohort definition) + bank `REJECTED` still on file

| Month | Accounts created | `bank_account_status = REJECTED` (subset) |
|-------|----------------:|---------------------------------------------:|
| 2026-04 | 250 | 20 |
| 2026-03 | 941 | 24 |
| 2026-02 | 642 | 9 |
| 2026-01 | 697 | 10 |
| 2025-12 | 487 | 3 |
| 2025-11 | 514 | 32 |
| 2025-10 | 318 | 38 |
| 2025-09 | 476 | 89 |

**Note:** **2026-04** is a **partial month** in the snapshot. The “REJECTED subset” column counts accounts **created in that month** whose **current** `bank_account_status` is still `REJECTED` (not “rejections that happened in that calendar month”).

### 4.3 How this compares to the Mexico wallet briefing

The Mexico pack shows **patient pay-in TPV** on saved Stripe cards. This pack shows **provider-side bank acceptance** for **Brazil dLocal payout setup** — different lens, same idea: **material counts**, not a niche edge case.

### 4.4 (Optional) Median time-to-onboarded metrics

For **SLA-style** “hours until `onboarded_at`” benchmarks (different SQL — joins monolith CRM), see the internal *dLocal Onboarding Time Analysis* / `dlocal-onboarding-analysis` skill — **do not mix those month counts directly** with §4.2 without reconciling definitions.

---

## 5. References

- Jira: [REVPAY-6359](https://docplanner.atlassian.net/browse/REVPAY-6359)  
- Formal scope / ACs: [request.md in dp-payments-ai](https://github.com/DocPlanner/dp-payments-ai/blob/main/specs/features/marketplace-payment-panel/cycles/6359-dlocal-br-bank-form-ui/request.md)  
- dLocal public reference: [Brazil payouts](https://docs.dlocal.com/docs/brazil-payouts)

