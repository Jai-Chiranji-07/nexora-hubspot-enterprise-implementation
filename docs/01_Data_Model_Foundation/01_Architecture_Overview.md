# Data Model Architecture Overview

## Document Control

| Field | Detail |
|---|---|
| Document Name | Nexora, Inc. — HubSpot CRM Data Model Architecture |
| Phase | Week 1 — CRM Data Model Foundation |
| Version | 1.1 |
| Status | Built & Live in Developer Portal — Fully Verified |
| Owner | CRM Administrator / RevOps Manager |
| Related Documents | `docs/00_Project_Planning/05_Business_Blueprint.md`, `docs/00_Project_Planning/01_HubSpot_Environment_Setup.md` |
| Last Updated | 2026-08-12 |

---

## 1. Executive Summary

This document defines the CRM data architecture built for Nexora, Inc.'s HubSpot Enterprise instance — a hybrid Product-Led Growth (PLG) + Sales-Assisted B2B SaaS company targeting $500K–$20M ARR companies across the US, UK, EU, and ANZ, operating under GDPR/CCPA/PIPEDA compliance obligations locked in Week 0[cite: 5].

The data model was designed against three explicit constraints derived from the locked business blueprint: (1) support for multi-provider recurring billing (Stripe **and** Razorpay), (2) GDPR-aware data governance from the first record created, and (3) architecture that remains performant at the Year 1 scale assumption of ~15,000 contacts and ~350 active paying customers[cite: 5].

---

## 2. Data Architecture Overview

### Standard Objects (Native HubSpot)

| Object | Role in Architecture |
|---|---|
| **Contact** | Individual people — trial users, decision-makers, end users. Carries PLG trial-tracking and product usage properties[cite: 5]. |
| **Company** | The organization/account. Carries plan, region, billing, and calculated health/revenue properties — the primary reporting entity[cite: 5]. |
| **Deal** | Sales pipeline tracking for the Sales-Assisted motion (Scale/Enterprise tiers). Native `hs_mrr`/`hs_arr` fields reused directly[cite: 5]. |
| **Ticket** | Support/service tracking (Service Hub). Feeds the Customer Health Score via open-ticket rollup[cite: 5]. |
| **Product** *(native library)* | Represents the 4 pricing tier SKUs (Starter/Growth/Scale/Enterprise) for use in Quotes and Deals[cite: 5]. |

### Custom Objects

| Object | Role in Architecture |
|---|---|
| **SaaS Subscription** | One record per subscription instance per Company — the system of record for recurring billing state, independent of which payment provider processed it[cite: 5]. |
| **Product Usage Event** | One record per Contact per week — an aggregated product usage snapshot feeding health scoring and PLG trial engagement tracking[cite: 5]. |

---

## 3. Entity Relationship Overview

- **Company ↔ SaaS Subscription** — one Company can have multiple Subscription records over its lifetime (history is retained on plan changes/cancellations rather than overwritten in place)[cite: 5].
- **Deal ↔ SaaS Subscription** — the Deal that closed-won generates the corresponding Subscription record; this is the sales-to-billing handoff point[cite: 5].
- **Company ↔ Contact** — native; standard account-to-people relationship[cite: 5].
- **Company ↔ Deal** — native; standard sales pipeline relationship[cite: 5].
- **Company ↔ Ticket** — native (Service Hub); feeds the `open_ticket_count` rollup used in health scoring[cite: 5].
- **Contact ↔ Product Usage Event** — a Contact generates one weekly usage snapshot; the primary engagement-tracking association[cite: 5].
- **Company ↔ Product Usage Event** — secondary association enabled for account-level (not just individual-level) usage rollups in future reporting phases[cite: 5].

Full visual ERD (Mermaid, GitHub-renderable): see `04_Diagrams_And_Screenshots.md`[cite: 5].

---

## 4. Key Architectural Trade-Offs & Decisions

### Decision 1 — Custom `SaaS Subscription` Object vs. HubSpot's Native Commerce `Subscription` Object

**Context:** During schema verification against the live portal, we confirmed HubSpot Enterprise ships a native standard object, `Subscription`, under Commerce Hub — purpose-built to "manage recurring revenue, billing, and payments inside HubSpot." This object is tightly coupled to HubSpot's own native Stripe-processed payment flow[cite: 5].

**Constraint:** Nexora's Business Blueprint (Week 0) explicitly commits to **both Stripe and Razorpay** as billing providers, to support customers across regions where Stripe coverage is incomplete (notably parts of the ANZ/EU markets Razorpay was selected for). HubSpot's native Subscription object's sync pathway is scoped to HubSpot-native Stripe processing and does not natively represent a second, independent payment provider's subscription state[cite: 5].

**Decision:** Built a distinctly-named custom object, `SaaS Subscription` (internal name: `saas_subscription`), rather than the native `Subscription` object[cite: 5].

**Trade-off accepted:** We forgo any out-of-the-box Commerce Hub reporting/automation built specifically around the native Subscription object. In exchange, we gain (a) a billing-provider-agnostic data model that treats Stripe and Razorpay as equal, first-class sources via the `billing_provider` property, (b) elimination of UI ambiguity between two similarly-named record types in the CRM record-type selector, and (c) direct demonstration of custom-object schema design — one of the explicit Week 1 roadmap deliverables[cite: 5].

---

### Decision 2 — Deal Pipeline vs. a 4th Custom Object (`Onboarding Milestones`)

**Context:** The Week 0 Business Blueprint's "Anticipated Custom Objects" preview originally listed a 4th object, Onboarding Milestones, to track the Account Setup → Training → Implementation → First Value Achieved sequence[cite: 5].

**Analysis:** Onboarding is a **sequential, staged process tied 1:1 to a single won Deal** — this is precisely the data shape HubSpot's native Pipeline mechanism is built for, not a repeating/many-per-record data entity (unlike Subscriptions, which a Company can have many of over time, or Usage Events, which recur weekly)[cite: 5].

**Decision:** Onboarding will be modeled as a dedicated **"Customer Onboarding" Deal pipeline** (a second pipeline on the standard Deal object, distinct from the Sales pipeline), rather than a new custom object[cite: 5].

**Trade-off accepted:** We avoid unnecessary object sprawl and a duplicate data-entity to maintain. In exchange, we inherit native Deal reporting, forecasting, and automation tooling for free, at the cost of onboarding data living on the same object type as sales deals (mitigated by pipeline separation and clear naming). *(Pipeline construction itself is scoped to a later implementation phase, not Week 1 — this document records the object-vs-pipeline decision made during Data Model Foundation.)*[cite: 5]

---

### Decision 3 — Weekly-Aggregated Product Usage Events vs. Granular Event-Level Logging

**Context:** Scale Assumptions (Week 0 Business Blueprint) project ~15,000 contacts by end of Year 1, with an actively-engaged product user base logging in and using features daily[cite: 5].

**Analysis:** Logging one CRM record per individual user action (login, feature click, etc.) would require Nexora's application to fire a HubSpot API call per action — at Year 1 scale, this is a difference of roughly two orders of magnitude in record volume and API call volume compared to a weekly aggregation model (≈15,000 contacts × 365 potential daily events/year vs. ≈15,000 contacts × 52 weekly snapshots/year)[cite: 5].

**Decision:** `Product Usage Event` stores **one aggregated record per Contact per week** (`login_count`, `features_used_count`, `primary_feature_used`), synced via a scheduled batch job from Nexora's application rather than real-time per-action webhooks[cite: 5].

**Trade-off accepted:** We lose the ability to trigger instant, action-level automation (e.g., "fire a workflow the moment a user touches Feature X"). In exchange, we gain a CRM record volume and API call pattern that scales predictably and remains well within Enterprise API rate limits and record-count norms at Nexora's projected scale — an explicit scale-aware architecture decision appropriate for a system expected to grow past Year 1[cite: 5].

---

## 5. Scale & Performance Considerations

Based on Week 0 Scale Assumptions (Year 1: ~15,000 contacts, ~350 active paying customers, ~2,500 companies)[cite: 5]:

- **SaaS Subscription** — projected ~350–450 active/historical records by end of Year 1 (low volume, no special handling needed)[cite: 5].
- **Product Usage Event** — projected ~15,000 contacts × 52 weeks ≈ **780,000 records/year** at full engagement. This is the highest-volume custom object in the schema by a wide margin. **Recommendation (not required for Week 1 scope, flagged for future planning):** introduce a data-retention/archival policy — e.g., roll weekly snapshots older than 12 months into a monthly-aggregate record — before Year 2 to keep the object's active record count manageable for reporting performance[cite: 5].