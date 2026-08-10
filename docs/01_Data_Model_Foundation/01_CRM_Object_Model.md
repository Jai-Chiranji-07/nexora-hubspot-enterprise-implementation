# CRM Object Model

## 1. Overview

The Nexora Enterprise HubSpot CRM uses a unified object model to manage customer, sales, support, subscription, learning, and enrollment data.

The data model is designed to provide a single source of truth across the complete customer lifecycle.

---

## 2. Standard CRM Objects

The implementation uses the following standard HubSpot CRM objects:

| Object | Purpose |
|---|---|
| Contact | Stores individual people and customer contacts |
| Company | Stores organizations and business accounts |
| Deal | Stores sales opportunities and commercial transactions |
| Ticket | Stores customer support and service requests |

---

## 3. Custom CRM Objects

The implementation uses the following custom CRM objects:

| Object | Purpose |
|---|---|
| Subscription | Stores customer subscription records and subscription-related information |
| Course | Stores courses and learning offerings |
| Enrollment | Stores individual customer enrollment records for courses |

---

## 4. Complete CRM Object Model

The complete Nexora CRM data model consists of:

1. Contact
2. Company
3. Deal
4. Ticket
5. Subscription
6. Course
7. Enrollment

These objects work together through HubSpot associations to maintain connected customer data.

---

## 5. Object Relationships

### Contact

Contact records are associated with:

- Company
- Deal
- Ticket
- Subscription
- Enrollment

### Company

Company records are associated with:

- Contact
- Deal
- Ticket
- Subscription

### Deal

Deal records are associated with:

- Contact
- Company
- Subscription

### Ticket

Ticket records are associated with:

- Contact
- Company

### Subscription

Subscription records are associated with:

- Company
- Contact
- Deal

### Enrollment

Enrollment records are associated with:

- Contact
- Course

### Course

Course records are associated with:

- Enrollment

---

## 6. Relationship Architecture

The core relationship model can be represented as:

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
