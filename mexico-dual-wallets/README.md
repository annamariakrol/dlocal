# Mexico: dual wallets (Stripe + dLocal) — stakeholder briefing

**Audience:** Business stakeholders  
**Purpose:** Standalone read — context, **why we are doing this**, and **quantified risk** if Stripe wallet were turned off when dLocal wallet is enabled  
**Ticket / epic:** [REVPAY-6358](https://docplanner.atlassian.net/browse/REVPAY-6358)  
**Product request (PR):** [DocPlanner/dp-payments-ai#255](https://github.com/DocPlanner/dp-payments-ai/pull/255)

**Full HTML version (GitHub Pages):** [annamariakrol.github.io/dlocal/mexico-dual-wallets/briefing.html](https://annamariakrol.github.io/dlocal/mexico-dual-wallets/briefing.html)

---

## 1. Executive summary

In Mexico, many patients already pay with **cards saved in the Stripe wallet** (one-click style pay-ins). Today the product cannot treat **Stripe** and **dLocal** wallets as **two parallel, patient-safe** capabilities without a deliberate design.

If we **enabled dLocal wallet** in a way that **automatically disabled Stripe wallet**, patients who rely on saved Stripe cards would be pushed back to **full card entry (e.g. Stripe Checkout)** on every payment. That is a **high-friction** change at scale.

**Evidence from BroadDB (Mexico payments shard)** shows non-trivial **saved-card payment volume and TPV** and meaningful **repeat usage** in 2026. The **REVPAY-6358** request (PR #255) describes **multiple wallets** (per customer provider, combined `/wallet`, dedupe, symmetric Stripe/dLocal behaviour) so we can **add dLocal wallet without breaking Stripe wallet** — protecting continuity of online pay-ins and reducing the risk of complaints, in-office payment substitution, and payment-setup churn.

This document is **evidence + narrative**; PR #255 is the **formal scope and acceptance criteria** for delivery.

---

## 2. What we are doing (short)

- **Deliver** patient wallet behaviour in Mexico that supports **both** Stripe-backed and dLocal-backed saved cards where the business requires it — with a **single combined wallet UI**, clear rules for **which cards appear at checkout**, and **remove** semantics that do not strand users.
- **Avoid** a “toggle” world where turning on dLocal wallet **silently kills** Stripe wallet for patients who already depend on it.

**Why:** continuity of payment experience, **TPV protection**, fewer support escalations to doctors, lower risk of moving clinics from mandatory to optional online payment.

---

## 3. What could go wrong without this feature (plain language)

If Stripe wallet is **disabled** while dLocal wallet is **enabled** (single-wallet assumption):

1. **Friction:** Patients must re-enter card details far more often → **drop-off** at pay step.  
2. **Behavioural substitution:** More **pay in office** instead of online prepay.  
3. **Doctor-facing noise:** Patients **complain to the doctor** (“your online payment is broken / harder”).  
4. **Product churn:** Clinics may **turn off mandatory prepay** or reduce online payment adoption.

The data below shows that **Stripe saved-card pay-ins are already a real lane in MX**, not a niche edge case — so the risk is **material**, not hypothetical.

---

## 4. Data evidence (Mexico, Stripe “saved card” pay-ins)

**Environment:** DocPlanner **BroadDB** — Mexico payments database `zl_crm_mx_broad` (anonymised dev snapshot).  
**Currency:** **MXN** for all rows in these extracts.

### 4.1 Definitions (same across all metrics)

We count **captured marketplace payments** where the payment object is **`UserCreditCardPayment`** (Stripe charge using a **saved** `credit_card_id` on the patient wallet side — i.e. not “type card details again for every charge” at the same friction level as a fresh checkout).

Technical filters:

- `payment.type = 'marketplace_payment'`
- `payment.status = 'captured'`
- `payment.payment_object_class = 'UserCreditCardPayment'`
- `user_credit_card_payment.credit_card_id IS NOT NULL`

**Wallet inventory** (authorised saved Stripe cards on file) additionally requires:

- `credit_card.type = 'user'`, `credit_card.status = 'authorized'`, `credit_card.is_fraudulent = 0`, `credit_card.deleted_at IS NULL`
- `user_credit_card_stripe_data.status = 'authorized'`

### 4.2 Wallet scale (authorised Stripe cards on file)

| Metric | Value |
|--------|------:|
| Distinct authorised Stripe wallet cards | **74,507** |
| Distinct patients (`user.external_id` → monolith `user.id`) | **68,130** |

### 4.3 2026 usage — saved-card pay-ins and repeat visits

Calendar year **2026** (through snapshot date on BroadDB):

| Metric | Value |
|--------|------:|
| Captured marketplace payments (`UserCreditCardPayment` + saved `credit_card_id`) | **22,929** |
| Successful **visit** payments linked to those pay-ins (monolith `crm_visit_payment`) | **22,610** payment rows |
| Distinct patients (visit path) | **14,590** |
| Distinct visit bookings (visit path) | **22,610** |
| **Patients with >1 visit booking paid with the *same* saved Stripe card in 2026** | **2,606** |
| Patient–card pairs with **>1** such visit booking | **2,627** |

**Custom transactions (same Stripe saved-card definition, 2026):** **203** patients, **273** successful custom payments; **29** patient–card pairs with **>1** custom payment on the same card (separate from visit “booking” counts).

**Interpretation for stakeholders:** a large cohort already **reuses** the same saved Stripe card across **multiple** bookings in a single year. Forcing full manual card entry repeatedly is likely to **hurt conversion** disproportionately for this group.

### 4.4 “Heavy wallet” cards (all time, >1 payment)

| Metric | Value |
|--------|------:|
| Distinct saved Stripe cards with **>1** captured marketplace payment (all time) | **14,176** |
| Highest observed payment count on a single `credit_card_id` | **91** payments |

### 4.5 Monthly TPV and transaction counts (saved Stripe card pay-ins)

TPV = `SUM(payment.amount)` per calendar month of `payment.created_at`.  
**April 2026 excluded** (partial month in snapshot). **Newest month first.**

| Month | TPV (MXN) | Transactions |
|------:|----------:|---------------:|
| 2026-03 | 7,265,787.00 | 7,626 |
| 2026-02 | 6,028,845.00 | 6,252 |
| 2026-01 | 6,846,325.01 | 7,222 |
| 2025-12 | 5,328,957.02 | 5,718 |
| 2025-11 | 5,465,969.04 | 6,005 |
| 2025-10 | 5,210,465.02 | 5,680 |
| 2025-09 | 5,188,371.04 | 5,565 |
| 2025-08 | 5,138,629.00 | 5,722 |
| 2025-07 | 5,304,916.01 | 5,878 |
| 2025-06 | 4,982,388.41 | 5,540 |
| 2025-05 | 4,225,625.02 | 4,624 |
| 2025-04 | 5,092,623.04 | 5,278 |
| 2025-03 | 5,021,522.51 | 5,685 |
| 2025-02 | 4,276,754.00 | 4,885 |
| 2025-01 | 4,529,784.40 | 5,243 |
| 2024-12 | 3,253,044.13 | 3,767 |
| 2024-11 | 3,668,616.40 | 4,248 |
| 2024-10 | 3,838,942.00 | 4,496 |
| 2024-09 | 3,342,931.99 | 3,959 |
| 2024-08 | 2,952,252.00 | 3,568 |
| 2024-07 | 2,926,689.00 | 3,584 |
| 2024-06 | 2,662,931.00 | 3,277 |
| 2024-05 | 2,589,666.50 | 3,185 |
| 2024-04 | 2,510,449.25 | 3,055 |
| 2024-03 | 2,219,613.00 | 2,717 |
| 2024-02 | 2,011,583.75 | 2,527 |
| 2024-01 | 2,194,290.00 | 2,775 |
| 2023-12 | 1,427,342.50 | 1,825 |
| 2023-11 | 543,968.00 | 672 |
| 2023-10 | 175.00 | 7 |

**Takeaway:** monthly saved-card TPV in MX is already in the **multi-million MXN** range in recent full months — i.e. **material revenue exposure** if the wallet lane is disrupted.

---

## 5. How this supports PR #255 / REVPAY-6358

| Stakeholder concern | How the data helps | How the feature (request) answers |
|--------------------|--------------------|-----------------------------------|
| “Is Stripe wallet marginal?” | **No** — tens of thousands of authorised cards, thousands of monthly transactions, multi-million MXN TPV. | Dual-wallet design keeps Stripe wallet **alive** while dLocal wallet is introduced. |
| “Do patients actually reuse saved cards?” | **Yes** — thousands of patients with **>1** booking on the **same** saved card in 2026 alone. | Per-payment / per-provider wallet rules avoid forcing everyone back to **full card entry**. |
| “What breaks if we flip a single global switch?” | Saved-card lane is **economically meaningful**; friction hits **repeat payers** hardest. | Request defines **symmetric** Stripe/dLocal behaviour, **combined `/wallet`**, dedupe, and rollout considerations — aimed at **no pay-in cliff**. |

**Important:** PR #255 is the **product request** (scope + AC). It **aligns** with the evidence above; **guaranteeing** no TPV disruption still depends on **implementation quality**, rollout gates, and monitoring in production.

---

## 6. Limitations (read before quoting numbers externally)

1. **BroadDB is anonymised dev data** — excellent for **direction and order-of-magnitude**, not for audited financial statements or investor filings.  
2. **Month boundaries** use stored `payment.created_at` (no Mexico-local timezone adjustment).  
3. Metrics count **payments-app captured pay-ins** with the Stripe saved-card object — reconciliation to finance / NetSuite may differ slightly.  
4. **dLocal wallet** volume is **not** the subject of this extract; this doc is intentionally **Stripe-wallet evidence** for the “do not disable” argument.

---

*Prepared for stakeholder circulation. For the authoritative delivery spec, use PR #255 and Jira REVPAY-6358.*
