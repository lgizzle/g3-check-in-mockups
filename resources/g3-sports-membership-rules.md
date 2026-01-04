# G3 Sports - Membership Rules & Configuration

> **For Developer Reference**
> Last Updated: January 2026
> Location: G3 Sports

---

## Overview

This document defines the membership types, pricing structure, and business rules for G3 Sports. Use this as the source of truth when implementing membership management features.

---

## Membership Types

### Paid Memberships

| Plan Name | Type | Monthly Rate | Init Fee | Max Members | Service Fee | Status |
|-----------|------|--------------|----------|-------------|-------------|--------|
| Full Membership | Individual | $55.00 | $99.00 | 1 | $9.00 | Active |
| Full Membership | Couples | $85.00 | $99.00 | 2 | $9.00 | Active |
| Full Membership | Family | $120.00 | $99.00 | 4 | $9.00 | Active |
| Student Full Membership | Individual | $40.00 | $49.00 | 1 | $9.00 | Active |
| G3 Volleyball - Academy | Individual | $150.00 | $50.00 | 1 | $0.00 | Active |

### Complimentary / Special Memberships (No Charge)

| Plan Name | Type | Monthly Rate | Init Fee | Max Members | Service Fee | Status |
|-----------|------|--------------|----------|-------------|-------------|--------|
| T3 Membership | Individual | $0.00 | $0.00 | 1 | $0.00 | Active |
| T3 Membership | Couples | $0.00 | $0.00 | 2 | $0.00 | Active |
| T3 Membership | Family | $0.00 | $0.00 | 4 | $0.00 | Active |
| G3 Volleyball - Coach | Individual | $0.00 | $0.00 | 1 | $0.00 | Active |
| G3 Volleyball - Coach | Couples | $0.00 | $0.00 | 2 | $0.00 | Active |
| G3 Employee | Individual | $0.00 | $0.00 | 1 | $0.00 | Active |
| G3 Employee | Family | $0.00 | $0.00 | 4 | $0.00 | Active |
| TAC Employee | Misc | $0.00 | $0.00 | 10 | $0.00 | Active |

---

## Field Definitions

| Field | Data Type | Description |
|-------|-----------|-------------|
| `plan_name` | string | The name of the membership plan |
| `type` | enum | Member count category: `Individual`, `Couples`, `Family`, `Misc` |
| `monthly_rate` | decimal | Recurring monthly charge (USD) |
| `init_fee` | decimal | One-time initiation/setup fee charged at signup (USD) |
| `max_members` | integer | Maximum number of people allowed on this membership |
| `service_fee` | decimal | Monthly service/billing fee (USD) |
| `status` | enum | Plan availability: `Active`, `Inactive`, `Discontinued` |

---

## Business Rules

### 1. Payment Method Required
```
All memberships MUST have a valid payment method attached to the account holder.
- No payment method = membership cannot be activated
- Payment method should be validated before setting status to Active
```

### 2. Membership Status Logic
```
Status values:
- ACTIVE: Member in good standing, can check in
- INACTIVE: Membership on hold or payment issue
- SUSPENDED: Temporarily suspended (show suspension end date)
- TERMINATED: Membership ended (show termination date)

Hold date activation → Set status to INACTIVE
```

### 3. Check-In Display Requirements
```
When member checks in, display:
- Current membership status (ACTIVE/INACTIVE/etc.)
- Termination date (if set)
- Suspension date/end date (if applicable)
- Hold status (if on hold)
```

### 4. Member Count by Type
```php
// Max members allowed per membership type
$maxMembers = [
    'Individual' => 1,
    'Couples'    => 2,
    'Family'     => 4,
    'Misc'       => 10  // Special case for TAC Employee
];
```

### 5. Billing Calculation
```php
// Total monthly charge
$totalMonthly = $monthly_rate + $service_fee;

// Example: Individual Full Membership
// $55.00 + $9.00 = $64.00/month
```

---

## Data Structure (JSON)

For import or API reference:

```json
{
  "location": "G3 Sports",
  "memberships": [
    {
      "plan_name": "Full Membership",
      "type": "Individual",
      "monthly_rate": 55.00,
      "init_fee": 99.00,
      "max_members": 1,
      "service_fee": 9.00,
      "status": "Active",
      "category": "Membership"
    },
    {
      "plan_name": "Full Membership",
      "type": "Couples",
      "monthly_rate": 85.00,
      "init_fee": 99.00,
      "max_members": 2,
      "service_fee": 9.00,
      "status": "Active",
      "category": "Membership"
    },
    {
      "plan_name": "Full Membership",
      "type": "Family",
      "monthly_rate": 120.00,
      "init_fee": 99.00,
      "max_members": 4,
      "service_fee": 9.00,
      "status": "Active",
      "category": "Membership"
    },
    {
      "plan_name": "Student Full Membership",
      "type": "Individual",
      "monthly_rate": 40.00,
      "init_fee": 49.00,
      "max_members": 1,
      "service_fee": 9.00,
      "status": "Active",
      "category": "Membership"
    },
    {
      "plan_name": "T3 Membership",
      "type": "Individual",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 1,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Complimentary"
    },
    {
      "plan_name": "T3 Membership",
      "type": "Couples",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 2,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Complimentary"
    },
    {
      "plan_name": "T3 Membership",
      "type": "Family",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 4,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Complimentary"
    },
    {
      "plan_name": "G3 Volleyball - Coach",
      "type": "Individual",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 1,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Membership"
    },
    {
      "plan_name": "G3 Volleyball - Coach",
      "type": "Couples",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 2,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Membership"
    },
    {
      "plan_name": "G3 Employee",
      "type": "Individual",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 1,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Employee"
    },
    {
      "plan_name": "G3 Employee",
      "type": "Family",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 4,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Employee"
    },
    {
      "plan_name": "TAC Employee",
      "type": "Misc",
      "monthly_rate": 0.00,
      "init_fee": 0.00,
      "max_members": 10,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Employee"
    },
    {
      "plan_name": "G3 Volleyball - Academy",
      "type": "Individual",
      "monthly_rate": 150.00,
      "init_fee": 50.00,
      "max_members": 1,
      "service_fee": 0.00,
      "status": "Active",
      "category": "Volleyball Academy"
    }
  ]
}
```

---

## Database Schema Suggestion

```sql
CREATE TABLE membership_plans (
    id INT AUTO_INCREMENT PRIMARY KEY,
    location_id INT NOT NULL,
    plan_name VARCHAR(100) NOT NULL,
    type ENUM('Individual', 'Couples', 'Family', 'Misc') NOT NULL,
    category VARCHAR(50),
    monthly_rate DECIMAL(10,2) DEFAULT 0.00,
    init_fee DECIMAL(10,2) DEFAULT 0.00,
    max_members INT DEFAULT 1,
    service_fee DECIMAL(10,2) DEFAULT 0.00,
    status ENUM('Active', 'Inactive', 'Discontinued') DEFAULT 'Active',
    requires_payment BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE member_memberships (
    id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    plan_id INT NOT NULL,
    status ENUM('Active', 'Inactive', 'Suspended', 'Terminated') NOT NULL,
    start_date DATE NOT NULL,
    termination_date DATE NULL,
    suspension_start DATE NULL,
    suspension_end DATE NULL,
    hold_start DATE NULL,
    hold_end DATE NULL,
    payment_method_id INT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (plan_id) REFERENCES membership_plans(id)
);
```

---

## Questions for Clarification

> **Note to staff:** Please confirm the following with the developer if unclear:

1. **Service Fee ($9.00)** - Is this a separate line item on invoices, or bundled into the monthly rate?
2. **T3 Membership** - What does "T3" stand for? Any special access restrictions?
3. **TAC Employee** - What is "TAC"? Why does this allow 10 members?
4. **Volleyball Coach** - Why is there a "Couples" option for coaches?

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-01 | Initial documentation | Staff |
