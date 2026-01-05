# G3 Sports - Services & Packages

> **For Developer Reference**
> Last Updated: January 2026
> Location: G3 Sports

---

## Overview

These are **consumable packages** - one-time purchases with session/visit tracking. Unlike memberships (recurring billing), these are punch-card style products where each use decrements the remaining balance.

---

## Check-In Requirements

```
At check-in, the system should:
1. Display all active packages with remaining visits
2. Show: Package Name | Remaining Uses | Total Purchased
3. Allow staff to "use" one session (deduct from balance)
4. Prevent deduction if balance = 0
```

---

## Day Passes

| Service Name | Price | Visits | Per-Visit Cost |
|--------------|-------|--------|----------------|
| Day Pass | $10.00 | 1 | $10.00 |
| Day Pass - 10 Visit + 1 | $100.00 | 11 | $9.09 |
| Free Day Pass | $0.00 | 1 | $0.00 |

---

## Personal Training - Individual Member - 30 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $45.00 | $45.00 |
| 4 pack | $176.00 | $44.00 |
| 8 pack | $344.00 | $43.00 |
| 12 pack | $504.00 | $42.00 |
| 20 pack | $800.00 | $40.00 |

---

## Personal Training - Individual Member - 60 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $70.00 | $70.00 |
| 4 pack | $272.00 | $68.00 |
| 8 pack | $528.00 | $66.00 |
| 12 pack | $768.00 | $64.00 |
| 20 pack | $1,200.00 | $60.00 |

---

## Personal Training - Couples - 60 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $100.00 | $100.00 |
| 4 pack | $384.00 | $96.00 |
| 8 pack | $752.00 | $94.00 |
| 12 pack | $1,092.00 | $91.00 |
| 15 pack | $1,335.00 | $89.00 |
| 20 pack | $1,720.00 | $86.00 |

---

## Personal Training - Individual Athlete - 90 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $90.00 | $90.00 |
| 4 pack | $352.00 | $88.00 |
| 8 pack | $688.00 | $86.00 |
| 12 pack | $1,008.00 | $84.00 |

---

## Team Training - 60 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $30.00 | $30.00 |
| 6 pack | $150.00 | $25.00 |
| 12 pack | $180.00 | $15.00 |

---

## Team Training - 90 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $40.00 | $40.00 |
| 6 pack | $215.00 | $35.83 |

---

## Small Group Training - 60 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $45.00 | $45.00 |
| 4 pack | $172.00 | $43.00 |
| 8 pack | $320.00 | $40.00 |
| 12 pack | $444.00 | $37.00 |

---

## Small Group Training - 90 Min

| Pack Size | Price | Per-Session |
|-----------|-------|-------------|
| 1 pack | $60.00 | $60.00 |
| 4 pack | $222.00 | $55.50 |
| 8 pack | $448.00 | $56.00 |
| 12 pack | $648.00 | $54.00 |

---

## Monthly Programming

| Service Name | Price | Sessions |
|--------------|-------|----------|
| Personal Training - Monthly Programming | $75.00 | 1 |

---

## Hybrid Personal Training

Monthly packages combining in-person sessions with programming:

| Package | Price | Sessions Included |
|---------|-------|-------------------|
| Silver | $225.00 | 1 |
| Gold | $291.00 | 2 |
| Platinum | $419.00 | 4 |

### 3-Month Hybrid Packages

| Package | Price | Total Sessions |
|---------|-------|----------------|
| 3-Month Silver | $629.00 | 3 |
| 3-Month Gold | $814.00 | 6 |
| 3-Month Platinum | $1,173.00 | 12 |

### 6-Month Hybrid Packages

| Package | Price | Total Sessions |
|---------|-------|----------------|
| 6-Month Silver | $1,152.00 | 6 |
| 6-Month Gold | $1,493.00 | 12 |
| 6-Month Platinum | $2,155.00 | 24 |

---

## Data Structure (JSON)

```json
{
  "location": "G3 Sports",
  "service_type": "consumable_package",
  "tracking": {
    "method": "session_deduction",
    "display_at_checkin": true,
    "allow_manual_deduction": true
  }
}
```

---

## Database Schema Suggestion

```sql
CREATE TABLE service_packages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    location_id INT NOT NULL,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50),           -- 'day_pass', 'personal_training', 'team_training', etc.
    subcategory VARCHAR(50),        -- '30_min', '60_min', '90_min'
    client_type VARCHAR(50),        -- 'individual', 'couples', 'athlete', 'team', 'small_group'
    price DECIMAL(10,2) NOT NULL,
    sessions_included INT NOT NULL,
    status ENUM('Active', 'Inactive') DEFAULT 'Active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE member_packages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    package_id INT NOT NULL,
    purchase_date DATE NOT NULL,
    sessions_total INT NOT NULL,
    sessions_remaining INT NOT NULL,
    status ENUM('Active', 'Exhausted', 'Expired') DEFAULT 'Active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (package_id) REFERENCES service_packages(id)
);

CREATE TABLE package_usage (
    id INT AUTO_INCREMENT PRIMARY KEY,
    member_package_id INT NOT NULL,
    used_date DATETIME NOT NULL,
    deducted_by INT,                -- staff user id
    notes VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (member_package_id) REFERENCES member_packages(id)
);
```

---

## Questions for Clarification

1. **Monthly Programming** - Is this a recurring monthly charge or a one-time package?
2. **Hybrid packages** - Do the 3-month and 6-month versions auto-bill monthly, or is it paid upfront?
3. **Expiration** - Do unused sessions expire? If so, after how long?
4. **Transfers** - Can sessions be transferred between family members?
5. **Refunds** - Can partially-used packages be refunded?
