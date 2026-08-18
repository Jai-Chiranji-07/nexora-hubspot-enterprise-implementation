# Data Governance Rules

## Document Control

| Field | Detail |
|---|---|
| Document Name | Nexora, Inc. — Data Governance & Calculation Logic |
| Phase | Week 1 — CRM Data Model Foundation |
| Status | Built & Live in Developer Portal |
| Related Documents | `01_Architecture_Overview.md`, `02_Object_Schema_Dictionary.md`, `04_Diagrams_And_Screenshots.md` |

---

## 1. Calculation & Rollup Logic

All calculated properties use HubSpot's **Calculation** field type, built via the "Build Calculation" flow, using one of three calculation types: Custom Equation, Time Between/Since, or Rollup aggregate (Min/Max/Count/Sum/Average across associated records).

### 1.1 ARR Amount (`arr_amount` — SaaS Subscription)

- **Calculation type:** Custom Equation
- **Formula:** `[properties.mrr_amount] * 12`
- **Purpose:** Derives Annual Recurring Revenue directly from Monthly Recurring Revenue, avoiding manual dual-entry.

### 1.2 Trial Days Remaining (`trial_days_remaining` — Contact) ⚪ **Modified / Reverted to Number**

- **Original Plan:** Built as a two-property chain.
- **Actual Build:** Due to HubSpot calculation chaining constraints encountered during the portal build, `trial_days_remaining` was deployed as a standard **Number** field rather than a Custom Equation. It will be maintained via Workflows (Week 3) rather than real-time property calculations.
- **Purpose:** Powers PLG self-serve trial nurture logic (Week 3) without requiring a workflow to recompute elapsed time on every enrollment check.

### 1.3 Total Active MRR (`total_active_mrr` — Company)

- **Calculation type:** Rollup — Sum
- **Property summed:** `mrr_amount`
- **Associated record type:** SaaS Subscription
- **Additional Condition (filter):** `subscription_status` equals `Active`
- **Purpose:** Gives an always-current, single source of truth for a Company's current recurring revenue, independent of Deal-level `hs_mrr` (which reflects the deal that sold the subscription, not necessarily its current live state after upgrades/downgrades).

### 1.4 Customer Health Score (`customer_health_score` — Company) ⚪ **Backlog (Deferred to Week 5)**

*Note: This architecture is deferred to the Service Hub implementation phase when Ticket and NPS data models are established.*

**Layer 1 — Rollup helper properties (Company) [Planned]:**
*   `avg_feature_adoption` (Average)
*   `open_ticket_count` (Count)
*   `avg_nps_score` (Average)
*   `subscription_health_input` (Average)

**Layer 2 — Final weighted formula (Company, Custom Equation) [Planned]:**
```text
([properties.avg_feature_adoption] * 0.40) +
(MAX(0, 100 - ([properties.open_ticket_count] * 10)) * 0.25) +
([properties.avg_nps_score] * 10 * 0.20) +
([properties.subscription_health_input] * 0.15)
```

### 1.5 Subscription Status Score (`subscription_status_score` — SaaS Subscription, helper) **⚪ Backlog (Deferred to Week 5)**

*Note: Deferred alongside the Customer Health Score architecture.*

Calculation type: Custom Equation

Formula [Planned]:

```text
IF([properties.subscription_status] == "Active", 100,
IF([properties.subscription_status] == "Past Due", 50, 0))
```

Purpose: Converts the subscription_status dropdown into a numeric input so it can feed the Company-level subscription_health_input rollup and, ultimately, the Customer Health Score formula.

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
