# Object Properties

## 1. Overview

Object properties define the structured data stored against each CRM object in the Nexora Enterprise HubSpot implementation.

The property model is designed around the business processes defined during Week 0 and the CRM data model configured during Week 1.

---

## 2. Standard CRM Objects

The following standard HubSpot objects are part of the Nexora CRM foundation:

- Contact
- Company
- Deal
- Ticket

Standard HubSpot properties are used wherever the required business information is already available.

Custom properties are created only where Nexora requires business-specific data that is not adequately represented by standard properties.

---

## 3. Custom CRM Objects

The following custom objects are part of the Nexora data model:

- Subscription
- Course
- Enrollment

These objects represent business entities that require their own records and relationships rather than being stored only as properties on standard CRM objects.

---

## 4. Contact Properties

### Standard Properties

The Contact object uses relevant standard HubSpot properties including:

- First Name
- Last Name
- Email
- Phone
- Mobile Phone
- Job Title
- Lifecycle Stage
- Lead Status
- Contact Owner
- Company

### Custom Property Groups

#### Sales Information

- Job Role
- Customer Decision Role
- Buying Authority

#### Customer Success

- Customer Health
- Churn Risk
- Onboarding Status
- Product Adoption Status
- Renewal Date
- Subscription Plan

#### Marketing Information

- Lead Source Detail
- Marketing Segment
- Customer Persona

---

## 5. Property Design Principles

The Nexora property model follows these principles:

1. Use standard HubSpot properties when they satisfy the business requirement.
2. Create custom properties only for Nexora-specific information.
3. Group related properties logically.
4. Use structured field types for controlled business values.
5. Use date fields for date-based business events.
6. Avoid creating duplicate properties when an appropriate standard property already exists.
7. Keep object-specific information on the correct CRM object.
8. Use associations between objects instead of duplicating related-record information as text properties.

---

## 6. Data Quality Considerations

Property design supports consistent CRM data by:

- Reducing free-text values where controlled selections are appropriate.
- Maintaining consistent naming conventions.
- Separating customer, sales, marketing, and customer-success information.
- Preventing unnecessary duplication between CRM objects.
- Supporting future workflows, reporting, segmentation, and automation.

---

## 7. Relationship to the Data Model

Properties describe the attributes of individual CRM records.

Associations describe relationships between CRM records.

Together, they form the core Nexora CRM data model:

```text
CRM Objects
    │
    ├── Properties
    │     └── Record attributes
    │
    └── Associations
          └── Record relationships
