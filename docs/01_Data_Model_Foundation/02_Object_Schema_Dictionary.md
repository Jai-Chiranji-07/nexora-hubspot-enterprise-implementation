# Object & Property Schema Dictionary

## Document Control

| Field | Detail |
|---|---|
| Document Name | Nexora, Inc. — CRM Object & Property Schema Dictionary |
| Phase | Week 1 — CRM Data Model Foundation |
| Version | 1.1 |
| Status | Reflects live portal state as of this documentation snapshot |
| Related Documents | `01_Architecture_Overview.md`, `03_Data_Governance_Rules.md` |
| Last Updated | 2026-08-12 |

---

## Legend

- 🟢 **Native (Reused)** — a standard HubSpot property, deliberately reused instead of duplicating with a custom property (cost-efficient, no redundant data entry)[cite: 6].
- 🔵 **Custom (Built)** — a custom property created specifically for this data model[cite: 6].
- 🟡 **Calculation** — a custom property whose value is computed automatically (Custom Equation, Time Between/Since, or Rollup aggregate)[cite: 6].

---

## 1. Company

### Native Properties Reused (No Duplication)

| Property | Internal Name | Why Reused Instead of Custom |
|---|---|---|
| Lifecycle Stage | `lifecyclestage` | Default stage set (Subscriber → Lead → MQL → SQL → Opportunity → Customer → Evangelist) already matches the Week 0 Customer Lifecycle Journey exactly[cite: 6]. |
| Ideal Customer Profile | `hs_ideal_customer_profile` | Native Tier 1/2/3 ICP scoring — matches the Week 0 ICP tiering need without a custom field[cite: 6]. |
| Industry | `industry` | Standard native field, sufficient as-is[cite: 6]. |
| Annual Revenue | `annualrevenue` | Standard native field — distinct from recurring MRR/ARR, which is tracked via custom/rollup properties below[cite: 6]. |
| Number of Employees | `numberofemployees` | Standard native field, matches ICP's 20–300 employee range criterion directly[cite: 6]. |

### Custom Properties — Group: `Business Info`

| Property Label | Internal Name | Field Type | Options |
|---|---|---|---|
| Region | `region` | Dropdown select | US, UK, EU, ANZ, Canada[cite: 6] |
| Growth Stage | `growth_stage` | Dropdown select | Seed, Series A, Series B, Profitable SMB[cite: 6] |

### Custom Properties — Group: `Subscription & Billing`

| Property Label | Internal Name | Field Type | Options / Logic |
|---|---|---|---|
| Plan Tier | `plan_tier` | Dropdown select | Starter, Growth, Scale, Enterprise[cite: 6] |
| GTM Motion | `gtm_motion` | Dropdown select | Self-Serve PLG, Sales-Assisted[cite: 6] |
| Billing Currency | `billing_currency` | Dropdown select | USD, GBP, EUR, CAD, AUD[cite: 6] |
| Total Active MRR 🟡 | `total_active_mrr` | Calculation (Rollup — Sum) | Sums `mrr_amount` from associated **SaaS Subscription** records where `subscription_status = Active`[cite: 6] |

### Custom Properties — Group: `Compliance & Consent`

| Property Label | Internal Name | Field Type | Options |
|---|---|---|---|
| GDPR Applicable | `gdpr_applicable` | Single checkbox | —[cite: 6] |
| Data Processing Legal Basis | `data_processing_legal_basis` | Dropdown select | Consent, Legitimate Interest[cite: 6] |

### Custom Properties — Group: `Support & Success`

| Property Label | Internal Name | Field Type | Options / Logic |
|---|---|---|---|
| Customer Health Score 🟡 | `customer_health_score` | Calculation (Custom Equation) | Weighted formula — see `03_Data_Governance_Rules.md` §1.4[cite: 6] |
| Customer Health Bucket | `customer_health_bucket` | Dropdown select | Healthy, At Risk, Critical (auto-set via workflow in a later phase)[cite: 6] |
| Avg Feature Adoption 🟡 *(helper)* | `avg_feature_adoption` | Calculation (Rollup — Average) | Average of `feature_adoption_pct` from associated Contacts[cite: 6] |
| Open Ticket Count 🟡 *(helper)* | `open_ticket_count` | Calculation (Rollup — Count) | Count of associated Tickets where status ≠ Closed[cite: 6] |
| Avg NPS Score 🟡 *(helper)* | `avg_nps_score` | Calculation (Rollup — Average) | Average of `nps_score` from associated Contacts[cite: 6] |
| Subscription Health Input 🟡 *(helper)* | `subscription_health_input` | Calculation (Rollup — Average) | Average of `subscription_status_score` from associated SaaS Subscription records[cite: 6] |

---

## 2. Contact

### Native Properties Reused (No Duplication)

| Property | Internal Name | Why Reused Instead of Custom |
|---|---|---|
| Lifecycle Stage | `lifecyclestage` | Shared stage set with Company; no separate contact-level lifecycle needed[cite: 6]. |
| Email | `email` | Standard identity field[cite: 6]. |
| Owner | `hubspot_owner_id` | Standard native field for CSM/Sales rep assignment[cite: 6]. |

### Custom Properties — Group: `Business Info`

| Property Label | Internal Name | Field Type | Options |
|---|---|---|---|
| Product Role | `product_role` | Dropdown select | Champion, End User, Economic Buyer, Technical Evaluator[cite: 6] |

### Custom Properties — Group: `Subscription & Billing`

| Property Label | Internal Name | Field Type | Options / Logic |
|---|---|---|---|
| Trial Start Date | `trial_start_date` | Date picker | —[cite: 6] |
| Trial Status | `trial_status` | Dropdown select | Active Trial, Converted, Expired – Not Converted[cite: 6] |
| Trial Days Elapsed 🟡 *(helper)* | `trial_days_elapsed` | Calculation (Time Since) | Elapsed time since `trial_start_date`, in days[cite: 6] |
| Trial Days Remaining 🟡 | `trial_days_remaining` | Calculation (Custom Equation) | `14 - trial_days_elapsed`[cite: 6] |

### Custom Properties — Group: `Product Usage`

| Property Label | Internal Name | Field Type | Options |
|---|---|---|---|
| Last Product Login Date | `last_product_login_date` | Date picker | —[cite: 6] |
| Feature Adoption % | `feature_adoption_pct` | Number (0–100) | —[cite: 6] |
| NPS Score | `nps_score` | Number | 0–10 scale[cite: 6] |

### Custom Properties — Group: `Compliance & Consent`

| Property Label | Internal Name | Field Type | Options |
|---|---|---|---|
| Consent Capture Timestamp | `consent_capture_timestamp` | Date picker (with time) | Audit-trail timestamp of when consent was captured[cite: 6] |

---

## 3. SaaS Subscription *(Custom Object)*

**Primary Display Property:** `subscription_name` (format convention: `{Company Name} – {Plan Tier} – {Start Date}`)[cite: 6]

**Associations:** Company (primary), Contact (billing contact), Deal (originating deal)[cite: 6]

### Properties — Group: `Subscription Details`

| Property Label | Internal Name | Field Type | Options / Logic |
|---|---|---|---|
| Plan Tier | `plan_tier` | Dropdown select | Starter, Growth, Scale, Enterprise[cite: 6] |
| Billing Currency | `billing_currency` | Dropdown select | USD, GBP, EUR, CAD, AUD[cite: 6] |
| Billing Provider | `billing_provider` | Dropdown select | Stripe, Razorpay[cite: 6] |
| Subscription Status | `subscription_status` | Dropdown select | Active, Past Due, Canceled, Trialing[cite: 6] |
| MRR Amount | `mrr_amount` | Number | Minimum-value validation enforced (`>= 0`)[cite: 6] |
| ARR Amount 🟡 | `arr_amount` | Calculation (Custom Equation) | `mrr_amount * 12`[cite: 6] |
| Subscription Status Score 🟡 *(helper)* | `subscription_status_score` | Calculation (Custom Equation) | `IF(status="Active", 100, IF(status="Past Due", 50, 0))`[cite: 6] |
| Subscription Start Date | `subscription_start_date` | Date picker | —[cite: 6] |
| Renewal Date | `renewal_date` | Date picker | —[cite: 6] |
| Next Billing Date | `next_billing_date` | Date picker | —[cite: 6] |
| External Billing ID | `external_billing_id` | Single-line text | **Unique value required** — API-level upsert key for Stripe/Razorpay sync (Week 7)[cite: 6] |

---

## 4. Product Usage Event *(Custom Object)*

**Primary Display Property:** `usage_period_label` (format convention: `{Contact Name} – Week of {Date}`)[cite: 6]

**Associations:** Contact (primary), Company (secondary, account-level rollups)[cite: 6]

### Properties — Group: `Usage Snapshot`

| Property Label | Internal Name | Field Type | Options |
|---|---|---|---|
| Usage Week Start Date | `usage_week_start_date` | Date picker | —[cite: 6] |
| Login Count | `login_count` | Number | —[cite: 6] |
| Features Used Count | `features_used_count` | Number | —[cite: 6] |
| Primary Feature Used | `primary_feature_used` | Single-line text | —[cite: 6] |
| Data Source | `data_source` | Dropdown select | Nexora App API, Manual Import[cite: 6] |

---

## 5. Deal *(Native, No Custom Additions in Week 1)*

| Property | Internal Name | Notes |
|---|---|---|
| Monthly Recurring Revenue | `hs_mrr` | Native — reused directly, no custom MRR property duplicated at the Deal level[cite: 6] |
| Annual Recurring Revenue | `hs_arr` | Native — reused directly[cite: 6] |
| Deal Name / Stage / Pipeline / Amount / Close Date / Deal Type | `dealname`, `dealstage`, `pipeline`, `amount`, `closedate`, `dealtype` | Standard fields, unchanged[cite: 6] |