# Custom Object Model

## 1. Overview

The Nexora CRM uses custom objects to represent business entities that require dedicated records, properties, and associations.

The custom object model consists of:

- Subscription
- Course
- Enrollment

These objects extend the standard HubSpot CRM model to support Nexora's subscription and learning processes.

---

## 2. Subscription Object

### Purpose

The Subscription object represents a customer's active or historical subscription relationship with Nexora.

### Business Role

A Subscription record connects the commercial relationship with the customer and organization.

### Key Relationships

Subscription is associated with:

- Contact
- Company
- Deal

### Core Business Information

The Subscription object supports information related to:

- Subscription identity
- Subscription plan
- Subscription status
- Customer relationship
- Commercial relationship

---

## 3. Course Object

### Purpose

The Course object represents a learning or training offering provided through the Nexora customer-success and onboarding process.

### Business Role

A Course record represents the specific course or learning offering associated with customer enrollment.

### Key Relationship

Course is associated with:

- Enrollment

---

## 4. Enrollment Object

### Purpose

The Enrollment object represents an individual's participation in a specific course.

### Business Role

Enrollment connects a Contact with a Course and provides a dedicated CRM record for tracking the learning relationship.

### Key Relationships

Enrollment is associated with:

- Contact
- Course

### Core Business Information

The Enrollment object includes:

- Enrollment Number
- Enrollment Status

---

## 5. Custom Object Relationship Model

```text
Contact
   │
   ├─────────────── Subscription
   │                     │
   │                     ├── Company
   │                     └── Deal
   │
   └─────────────── Enrollment
                         │
                         └── Course
