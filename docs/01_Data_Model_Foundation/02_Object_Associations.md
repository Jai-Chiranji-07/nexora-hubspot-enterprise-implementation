# Object Associations

## 1. Overview

Object associations define how CRM records are connected within the Nexora Enterprise HubSpot CRM.

The association model connects customer, company, sales, service, subscription, and learning data into a unified CRM structure.

---

## 2. Contact Associations

The Contact object is associated with:

- Company
- Deal
- Ticket
- Subscription
- Enrollment

Contacts therefore act as a central customer identity across sales, service, subscription, and enrollment processes.

---

## 3. Company Associations

The Company object is associated with:

- Contact
- Deal
- Ticket
- Subscription

This allows organization-level customer information to remain connected with people, sales opportunities, support activity, and subscriptions.

---

## 4. Deal Associations

The Deal object is associated with:

- Contact
- Company
- Subscription

This connects sales opportunities with the customer, organization, and resulting subscription relationship.

---

## 5. Ticket Associations

The Ticket object is associated with:

- Contact
- Company

This connects support activity to both the individual requesting assistance and the organization they belong to.

---

## 6. Subscription Associations

The Subscription object is associated with:

- Company
- Contact
- Deal

This connects subscription records to the customer, organization, and originating commercial relationship.

---

## 7. Enrollment Associations

The Enrollment object is associated with:

- Contact
- Course

This connects an individual customer with the course in which they are enrolled.

---

## 8. Course Associations

The Course object is associated with:

- Enrollment

This allows each course to be connected with its enrollment records.

---

## 9. Association Matrix

| From Object | Associated Object |
|---|---|
| Contact | Company |
| Contact | Deal |
| Contact | Ticket |
| Contact | Subscription |
| Contact | Enrollment |
| Company | Contact |
| Company | Deal |
| Company | Ticket |
| Company | Subscription |
| Deal | Contact |
| Deal | Company |
| Deal | Subscription |
| Ticket | Contact |
| Ticket | Company |
| Subscription | Company |
| Subscription | Contact |
| Subscription | Deal |
| Enrollment | Contact |
| Enrollment | Course |
| Course | Enrollment |

---

## 10. Relationship Architecture

```text
Company
   │
   ├── Contact
   │     ├── Deal
   │     ├── Ticket
   │     ├── Subscription
   │     └── Enrollment
   │             │
   │             └── Course
   │
   ├── Deal
   │     └── Subscription
   │
   ├── Ticket
   │
   └── Subscription
