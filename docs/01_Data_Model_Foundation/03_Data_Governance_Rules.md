# Data Governance Rules

## Document Control

| Field | Detail |
|---|---|
| Document Name | Nexora, Inc. — Data Governance & Calculation Logic |
| Phase | Week 1 — CRM Data Model Foundation |
| Version | 1.1 |
| Related Documents | `01_Architecture_Overview.md`, `02_Object_Schema_Dictionary.md` |
| Last Updated | 2026-08-12 |

---

## 1. Calculation & Rollup Logic

All calculated properties use HubSpot's **Calculation** field type, built via the "Build Calculation" flow, using one of three calculation types: Custom Equation, Time Between/Since, or Rollup aggregate (Min/Max/Count/Sum/Average across associated records).

### 1.1 ARR Amount (`arr_amount` — SaaS Subscription)

- **Calculation type:** Custom Equation
- **Formula:** `[properties.mrr_amount] * 12`
- **Purpose:** Derives Annual Recurring Revenue directly from Monthly Recurring Revenue, avoiding manual dual-entry.

### 1.2 Trial Days Remaining (`trial_days_remaining` — Contact)

Built as a two-property chain rather than a single formula, per HubSpot's guidance to break complex nested logic into smaller chained properties:

- **Step 1 — `trial_days_elapsed`:** Calculation type = Time Since, base property = `trial_start_date`, output in days.
- **Step 2 — `trial_days_remaining`:** Calculation type = Custom Equation, formula = `14 - [properties.trial_days_elapsed]`.
- **Purpose:** Powers PLG self-serve trial nurture logic (Week 3) without requiring a workflow to recompute elapsed time on every enrollment check.

### 1.3 Total Active MRR (`total_active_mrr` — Company)

- **Calculation type:** Rollup — Sum
- **Property summed:** `mrr_amount`
- **Associated record type:** SaaS Subscription
- **Additional Condition (filter):** `subscription_status` equals `Active`
- **Purpose:** Gives an always-current, single source of truth for a Company's current recurring revenue, independent of Deal-level `hs_mrr` (which reflects the deal that sold the subscription, not necessarily its current live state after upgrades/downgrades).

### 1.4 Customer Health Score (`customer_health_score` — Company) — Two-Layer Architecture

**Layer 1 — Rollup helper properties (Company):**

| Helper Property | Calculation Type | Logic |
|---|---|---|
| `avg_feature_adoption` | Rollup — Average | Average `feature_adoption_pct` from associated Contacts |
| `open_ticket_count` | Rollup — Count | Count of associated Tickets where status ≠ Closed |
| `avg_nps_score` | Rollup — Average | Average `nps_score` from associated Contacts |
| `subscription_health_input` | Rollup — Average | Average `subscription_status_score` (see 1.5 below) from associated SaaS Subscription records |

**Layer 2 — Final weighted formula (Company, Custom Equation):**

```
([properties.avg_feature_adoption] * 0.40) +
(MAX(0, 100 - ([properties.open_ticket_count] * 10)) * 0.25) +
([properties.avg_nps_score] * 10 * 0.20) +
([properties.subscription_health_input] * 0.15)
```

- Weights (40/25/20/15) match the methodology defined in `docs/00_Project_Planning/05_Business_Blueprint.md`.
- `open_ticket_count` is inverted (more open tickets → lower score contribution).
- `avg_nps_score` is normalized from a 0–10 scale to 0–100 via `× 10`.
- **Note:** Enumeration properties (like `subscription_status`) cannot be referenced directly inside numeric equations — this is why `subscription_status_score` (below) exists as a numeric proxy.

### 1.5 Subscription Status Score (`subscription_status_score` — SaaS Subscription, helper)

- **Calculation type:** Custom Equation
- **Formula:**
```
IF([properties.subscription_status] == "Active", 100,
  IF([properties.subscription_status] == "Past Due", 50, 0))
```
- **Purpose:** Converts the `subscription_status` dropdown into a numeric input so it can feed the Company-level `subscription_health_input` rollup and, ultimately, the Customer Health Score formula.

---

## 2. Data Validation Rules

| Rule | Object / Property | Mechanism | Status |
|---|---|---|---|
| Email format | Contact `email` | Native HubSpot validation | ✅ Enforced (native, no configuration needed) |
| Enumeration-constrained input | All dropdown properties (`plan_tier`, `subscription_status`, `region`, etc.) | Dropdown field type inherently prevents free-text drift | ✅ Enforced by field type |
| Unique External Billing ID | SaaS Subscription `external_billing_id` | "Require unique value" advanced property setting | ✅ **Confirmed built** |
| MRR non-negative | SaaS Subscription mrr_amount | Number field "Minimum value" advanced setting (>= 0) | ✅ Configured & Enforced |


## 3. Deduplication & Merge Strategy

### 3.1 Native Dedupe — Scope Limitation

HubSpot's native deduplication/merge tooling covers **Contacts and Companies only**. Neither of our custom objects (`SaaS Subscription`, `Product Usage Event`) has access to native dedupe tooling — this is a platform limitation, not a configuration gap, and must be handled at the integration layer instead.

### 3.2 SaaS Subscription — Upsert Key

- **Field:** `external_billing_id` (single-line text, unique value required — confirmed built)
- **Strategy:** When syncing from Stripe or Razorpay (Week 7 — Third-Party Integrations), the integration script performs an API-level **upsert**: query for an existing SaaS Subscription record by `external_billing_id` before creating a new one. If found, update in place; if not found, create.
- **Why not rely on HubSpot dedupe:** Since native dedupe doesn't cover custom objects, this uniqueness constraint plus upsert-before-create logic is the only reliable safeguard against duplicate subscription records from repeated webhook deliveries or sync retries.

### 3.3 Product Usage Event — Composite Key Consideration

- Product Usage Event has no single natural unique identifier — its logical uniqueness is the pair `(contact, usage_week_start_date)`.
- HubSpot does not support composite-unique constraints across two properties natively.
- **Recommendation for Week 7 implementation:** enforce this at the application/integration layer — query for an existing record matching both the associated Contact and `usage_week_start_date` before inserting a new weekly snapshot, rather than relying on any portal-level setting.

### 3.4 Merge Behavior for Associated Custom Objects

- Native Contact/Company merges are expected to carry over associations to related custom object records, but this behavior should be **explicitly tested in the Standard Sandbox** (per `01_HubSpot_Environment_Setup.md` §3) before being relied upon in production — this is a documented assumption pending verification, not a confirmed guarantee.
