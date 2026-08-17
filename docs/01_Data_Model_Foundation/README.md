# 🏗️ Week 1: CRM Data Model Foundation

Welcome to the Data Model Foundation directory for the **Nexora, Inc.** HubSpot Enterprise implementation.

This phase establishes the structural core of our CRM, translating the business requirements (B2B SaaS, hybrid PLG + Sales-led) into a highly scalable, GDPR-compliant HubSpot data architecture.

## 📂 Directory Index

Navigate through the technical documentation below:

1. [**01_Architecture_Overview.md**](./01_Architecture_Overview.md)
   * Executive summary, object relationships, and key architectural trade-offs (e.g., why we bypassed the native Commerce object for a multi-provider Custom Object approach).
   
2. [**02_Object_Schema_Dictionary.md**](./02_Object_Schema_Dictionary.md)
   * The complete property mapping across Company, Contact, Deal, and our Custom Objects, strictly adhering to `snake_case` internal naming.
   
3. [**03_Data_Governance_Rules.md**](./03_Data_Governance_Rules.md)
   * Technical logic for Calculated Properties (Rollups, Equations), UI validation rules (e.g., non-negative MRR), and our API-level deduplication strategy.
   
4. [**04_Diagrams_And_Screenshots.md**](./04_Diagrams_And_Screenshots.md)
   * Visual Entity Relationship Diagram (ERD) and photographic evidence of the portal configuration.

## 🎯 Key Achievements in Week 1
* Successfully deployed **2 Custom Objects**: `SaaS Subscription` and `Product Usage Event`.
* Configured **30+ custom properties** with strict data typing and dropdown constraints.
* Wired advanced **Calculations & Rollups** (e.g., `Total Active MRR`, `ARR Amount`).
* Enforced UI-level **data validation** and unique record constraints.