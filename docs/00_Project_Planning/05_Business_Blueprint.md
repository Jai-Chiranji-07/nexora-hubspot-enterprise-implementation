# Business Blueprint

## Overview

This document defines the complete operating model of Nexora, Inc. ("Nexora"), an AI-powered B2B SaaS company.

It establishes how the business operates across marketing, sales, customer success, customer support, operations, and product engineering while providing the operational foundation for every customer-facing business process.

---

# Business Overview

Nexora is a subscription-based AI-powered B2B SaaS company that helps growing businesses centralize marketing, sales, customer success, operations, and analytics through intelligent automation.

The platform enables organizations to manage their complete customer lifecycle from first interaction to long-term customer growth using one unified CRM ecosystem.

---

# Scale Assumptions

These assumptions establish the operational scale Nexora's HubSpot instance is designed to support. They drive workflow limits, custom object volume planning, API rate-limit considerations, and reporting granularity throughout the implementation.

## Year 1 Targets

- **Total Contacts:** ~15,000
- **Total Companies:** ~2,500
- **New Marketing Qualified Leads:** ~600/month
- **New Sales Qualified Leads:** ~150/month
- **New Deals Created:** ~40/month
- **Active Paying Customers (EOY 1):** ~350
- **Support Tickets:** ~200/month

## Year 3 Targets (Scale Case)

- **Total Contacts:** ~120,000
- **Active Paying Customers:** ~2,500
- **Enterprise Accounts (Custom Quote tier):** ~50

These figures inform decisions such as: workflow re-enrollment limits, whether custom object associations need pagination-aware API scripts, and when a dedicated data warehouse sync (Operations Hub) becomes necessary versus native HubSpot reporting being sufficient.

---

# Go-to-Market Motion

Nexora follows a **hybrid Product-Led Growth (PLG) + Sales-Assisted** go-to-market model:

- **Starter and Growth tiers** are self-serve: customers sign up via the 14-day free trial, activate independently, and convert without sales involvement. Lifecycle automation for this segment is triggered by in-app usage signals and billing events rather than manual sales qualification.
- **Scale and Enterprise tiers** are sales-assisted: leads are routed to a Sales Qualified Lead (SQL) stage, undergo discovery calls and demos, and move through a manually-managed deal pipeline with quotes and negotiation.

This distinction directly shapes the CRM implementation: self-serve signups enter the lifecycle via product/billing webhooks (Stripe → HubSpot), while sales-assisted leads enter via form submissions and lead scoring thresholds that trigger SQL routing.

---

# Business Departments

## Marketing

### Responsibilities

- Generate website traffic
- Capture leads
- Run marketing campaigns
- Nurture prospects
- Qualify leads

---

## Sales

### Responsibilities

- Manage sales opportunities
- Conduct discovery calls
- Deliver product demonstrations
- Prepare quotations
- Close deals
- Forecast revenue

---

## Customer Success

### Responsibilities

- Customer onboarding
- Product adoption
- Subscription renewals
- Customer health monitoring
- Upsell opportunities

---

## Customer Support

### Responsibilities

- Ticket management
- Knowledge Base
- SLA management
- Customer satisfaction

---

## Operations

### Responsibilities

- CRM administration
- Data quality
- Workflow automation
- Business process optimization
- System integrations

---

## Product & Engineering

### Responsibilities

- REST APIs
- Webhooks
- CRM Extensions
- Third-party integrations
- Product development

---

# Lead Lifecycle

1. Anonymous Website Visitor
2. Lead Capture
3. Marketing Qualified Lead (MQL)
4. Sales Qualified Lead (SQL)
5. Discovery Call
6. Product Demo
7. Opportunity Created
8. Proposal / Quote
9. Deal Closed
10. Customer

---

# Customer Lifecycle

1. Subscription Activated
2. Customer Onboarding
3. Product Adoption
4. Customer Support
5. Customer Health Monitoring
6. Subscription Renewal
7. Upsell / Expansion
8. Customer Advocacy

---

# Customer Health Score Methodology

Customer health monitoring is driven by a calculated HubSpot property (`customer_health_score`) rather than a subjective judgment, combining four weighted signals:

| Signal | Weight | Source |
|---|---|---|
| Product Usage (logins/active features in last 30 days) | 40% | Product usage events synced via API |
| Support Ticket Volume & Severity (last 90 days) | 25% | Service Hub tickets |
| NPS / CSAT Score (most recent survey) | 20% | Service Hub surveys |
| Subscription Tenure & Payment Status | 15% | Stripe → HubSpot Deal/Subscription sync |

Each signal is normalized to a 0–100 scale and combined into a single weighted score, bucketed into **Healthy (70–100)**, **At Risk (40–69)**, and **Critical (0–39)**. A drop into "At Risk" or "Critical" triggers an internal workflow that notifies the assigned Customer Success Manager and can enroll the account in a re-engagement sequence.

---

# Internal Business Process

Marketing generates qualified leads.

↓

Sales qualifies opportunities, conducts discovery calls and product demonstrations, then converts prospects into customers.

↓

Customer Success ensures successful onboarding, product adoption, and long-term account growth.

↓

Customer Support resolves customer issues and maintains customer satisfaction.

↓

Operations maintains CRM data quality, automates business processes, and manages business systems.

↓

Leadership monitors business performance through dashboards, analytics, forecasting, and operational reporting.

---

# Technology Ecosystem

## Core Business Platform

- HubSpot CRM

---

## Business Tools

- Stripe
- Razorpay
- Slack
- Google Workspace
- Google Sheets
- Twilio
- Zapier
- Zoom
- Power BI
- OpenAI API

---

## AI Tooling Clarification

Nexora uses two distinct AI layers that serve different audiences — this distinction is intentional and should not be collapsed into a single "AI integration" line item:

- **HubSpot Breeze AI** — used internally by Nexora's own Marketing, Sales, and Operations teams *within* the HubSpot CRM itself (AI-assisted lead scoring, content generation, and workflow recommendations). This is HubSpot-native and requires no custom development.
- **OpenAI API** — powers the **AI Business Assistant module of the Nexora Platform product itself** (the feature Nexora sells to its own customers, per `02_Product_Definition.md`). This is a custom integration built on Nexora's own application layer, separate from the CRM, and is one of the touchpoints that syncs data back into HubSpot (e.g., logging AI Assistant usage as a product engagement event on the Contact timeline).

---

# Business KPIs

## Marketing

- Website Visitors
- Visitor-to-Lead Conversion Rate
- Marketing Qualified Leads (MQLs)

---

## Sales

- Opportunities Created
- Win Rate
- Sales Cycle Length
- Revenue

---

## Customer Success

- Customer Retention
- Renewal Rate
- Expansion Revenue

---

## Customer Support

- First Response Time
- Resolution Time
- Customer Satisfaction (CSAT)

---

## Leadership

- Monthly Recurring Revenue (MRR)
- Annual Recurring Revenue (ARR)
- Customer Lifetime Value (CLV)
- Revenue Forecast
- Pipeline Value

---

# Anticipated Custom Objects (Week 1 Preview)

Based on the business model defined above, the following custom objects are anticipated for the Week 1 – Data Model Foundation phase (replacing the generic Courses/Subscriptions/Enrollments placeholders from the original roadmap template with objects tailored to Nexora's actual B2B SaaS model):

- **Subscriptions** — tracks plan tier, MRR/ARR value, billing currency, and renewal date; associated with Company and synced from Stripe.
- **Product Usage Events** — logs feature adoption and login activity per Contact/Company; feeds the Customer Health Score.
- **Contracts / Renewals** — tracks Enterprise-tier custom contract terms, SLA commitments, and renewal cycles distinct from self-serve subscriptions.
- **Onboarding Milestones** — tracks stage-by-stage progress through the Customer Onboarding phase (Account Setup → Training → Implementation → First Value Achieved) for Customer Success visibility.

This section exists so that Week 0 documentation directly seeds the Week 1 data model design rather than requiring re-discovery.

---

# HubSpot Implementation Alignment

The business processes defined in this blueprint directly support the phased HubSpot implementation roadmap.

- Week 1 — CRM Data Model Foundation
- Week 2 — Lead Acquisition & CRM Foundation
- Week 3 — Marketing Automation
- Week 4 — Sales Processes
- Week 5 — Customer Success, Service & Operations
- Week 6 — Developer Platform
- Week 7 — Integrations
- Week 8 — Reporting, Dashboards & Executive Analytics

---

# Operating Principles

Nexora operates according to six core principles:

- Customer-First
- AI-First
- Automation-First
- Data-Driven Decision Making
- Unified CRM as the Single Source of Truth
- Scalable Platform Architecture
