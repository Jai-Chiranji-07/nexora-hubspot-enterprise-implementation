# Diagrams & Screenshot Tracking Guide

## Document Control

| Field | Detail |
|---|---|
| Document Name | Nexora, Inc. — Data Model Diagrams & Evidence Tracking |
| Phase | Week 1 — CRM Data Model Foundation |
| Status | Built & Live in Developer Portal |
| Related Documents | `01_Architecture_Overview.md`, `02_Object_Schema_Dictionary.md` |

---

## 1. Entity Relationship Diagram

Renders natively on GitHub (Mermaid support built into GitHub Markdown).

```mermaid
erDiagram
    COMPANY {
        string name
        string domain
        string plan_tier
        string region
        string gtm_motion
        number total_active_mrr
        number customer_health_score
        string hs_ideal_customer_profile
        string lifecyclestage
    }
    CONTACT {
        string email
        string lifecyclestage
        string product_role
        date trial_start_date
        string trial_status
        number feature_adoption_pct
        number nps_score
    }
    DEAL {
        string dealname
        string dealstage
        string pipeline
        number amount
        number hs_mrr
        number hs_arr
        date closedate
    }
    TICKET {
        string subject
        string status
        string pipeline
    }
    SAAS_SUBSCRIPTION {
        string subscription_name
        string plan_tier
        string billing_provider
        string subscription_status
        number mrr_amount
        number arr_amount
        string external_billing_id
        date renewal_date
    }
    PRODUCT_USAGE_EVENT {
        string usage_period_label
        date usage_week_start_date
        number login_count
        number features_used_count
        string primary_feature_used
    }

    COMPANY ||--o{ CONTACT : employs
    COMPANY ||--o{ DEAL : has
    COMPANY ||--o{ TICKET : has
    COMPANY ||--o{ SAAS_SUBSCRIPTION : owns
    COMPANY ||--o{ PRODUCT_USAGE_EVENT : "rolls up"
    CONTACT ||--o{ PRODUCT_USAGE_EVENT : generates
    CONTACT }o--o{ SAAS_SUBSCRIPTION : "billing contact"
    DEAL ||--o| SAAS_SUBSCRIPTION : creates
```

---

## 2. Screenshot Tracking Layout

### Suggested Folder Structure

```
docs/01_Data_Model_Foundation/
├── 01_Architecture_Overview.md
├── 02_Object_Schema_Dictionary.md
├── 03_Data_Governance_Rules.md
├── 04_Diagrams_And_Screenshots.md
└── screenshots/
    ├── 01_custom_objects_list.png
    ├── 02_saas_subscription_object_settings.png
    ├── 03_product_usage_event_object_settings.png
    ├── 04_company_properties_groups_view.png
    ├── 05_contact_properties_groups_view.png
    ├── 06_calc_arr_amount_config.png
    ├── 07_rollup_total_active_mrr_config.png
    └── 08_external_billing_id_unique_constraint.png
```

### Capture Mapping

| Filename | What to Capture | Supports |
|---|---|---|
| `01_custom_objects_list.png` | Settings → Objects → Custom Objects, showing both SaaS Subscription and Product Usage Event listed | `01_Architecture_Overview.md` §2 |
| `02_saas_subscription_object_settings.png` | SaaS Subscription object config screen — primary display property, enabled associations | `01_Architecture_Overview.md` §3 |
| `03_product_usage_event_object_settings.png` | Product Usage Event object config screen | `01_Architecture_Overview.md` §3 |
| `04_company_properties_groups_view.png` | Company property groups list showing Business Info, Subscription & Billing, Compliance & Consent, Support & Success | `02_Object_Schema_Dictionary.md` §1 |
| `05_contact_properties_groups_view.png` | Contact property groups list | `02_Object_Schema_Dictionary.md` §2 |
| `06_calc_arr_amount_config.png` | ARR Amount property editor showing the Custom Equation and the **green "No issues" validation indicator** | `03_Data_Governance_Rules.md` §1.1 |
| `07_rollup_total_active_mrr_config.png` | Total Active MRR rollup config showing Sum type, associated object = SaaS Subscription, and the Additional Condition filter (`subscription_status = Active`) | `03_Data_Governance_Rules.md` §1.3 |
| `08_external_billing_id_unique_constraint.png` | `external_billing_id` property editor showing "Require unique value" toggled on | `03_Data_Governance_Rules.md` §3.2 |

**Usage note:** Each `.md` file in this folder should embed its relevant screenshot(s) inline using standard Markdown image syntax once captured, e.g.:

```markdown
![Total Active MRR rollup configuration](./screenshots/07_rollup_total_active_mrr_config.png)
```
