# Timberhill - Membership Rules & Configuration

> **For Developer Reference**
> Last Updated: January 2026
> Location: Timberhill

---

## Overview

This document defines the membership types, pricing structure, and business rules for Timberhill. This location has more complex membership rules including age restrictions, time-based access, and therapy tracking.

---

## Membership Categories

### 1. Standard Active Memberships

| Plan Name | Type | Access Level | Notes |
|-----------|------|--------------|-------|
| Individual Health Club | Individual | Health Club | Standard gym access |
| Individual Full Club | Individual | Full Club | Full facility access |
| Couples Health Club | Couples | Health Club | 2 members |
| Couples Full Club | Couples | Full Club | 2 members |
| Family Health Club | Family | Health Club | Family members |
| Family Full Club | Family | Full Club | Family members |

### 2. Senior Memberships

| Plan Name | Type | Access Level | Notes |
|-----------|------|--------------|-------|
| Individual Senior Health Club | Individual | Health Club | Senior rate |
| Individual Senior Full Club | Individual | Full Club | Senior rate |
| Couples Senior Health Club | Couples | Health Club | Senior rate |
| Couples Senior Full Club | Couples | Full Club | Senior rate |
| Family Senior Health Club | Family | Health Club | Senior rate |
| Family Senior Full Club | Family | Full Club | Senior rate |

### 3. Platinum Memberships

| Plan Name | Type | Notes |
|-----------|------|-------|
| Platinum Individual Membership | Individual | Premium tier |
| Platinum Couples Membership | Couples | Premium tier |
| Platinum Family Membership | Family | Premium tier |

### 4. Extended Family Member

| Plan Name | Type | Notes |
|-----------|------|-------|
| Extended Family Member | Add-on | For family over 25 living in household |

### 5. Student Memberships

| Plan Name | Type | Notes |
|-----------|------|-------|
| Student Membership | Individual | 2-month minimum required |

### 6. Temporary Memberships

| Plan Name | Duration | Types Available |
|-----------|----------|-----------------|
| Weekly Temp | 7 days | Individual, Couples, Family |
| Monthly Temp | 1 month | Individual, Couples, Family |

### 7. Therapy Memberships

| Plan Name | Duration | Notes |
|-----------|----------|-------|
| Therapy Membership | 1 month | Requires medical documentation |

### 8. Playroom Memberships

| Plan Name | Children Allowed |
|-----------|------------------|
| Playroom - One Child | 1 |
| Playroom - Two Children | 2 |
| Playroom - Three Children | 3 |

### 9. Personal Training

| Plan Name | Notes |
|-----------|-------|
| Personal Training Packages | Various session counts |
| Personal Training Memberships | Ongoing training |

---

## Business Rules

### Rule 1: Couples/Family Member Eligibility

```
VALIDATION REQUIRED AT MEMBER ADDITION:

Eligibility Requirements:
1. Must live in the same household as primary member
2. Must be under 26 years of age

Implementation:
- Calculate age from birthdate at time of addition
- IF age >= 26 THEN block addition with error message
- Display: "This person does not qualify for this membership.
           Family members must be under 26 years of age and
           living in the household."
```

```php
// Age validation for family/couples members
function isEligibleFamilyMember($birthdate) {
    $age = calculateAge($birthdate);
    return $age < 26;
}

function calculateAge($birthdate) {
    $birth = new DateTime($birthdate);
    $today = new DateTime('today');
    return $birth->diff($today)->y;
}
```

### Rule 2: Daytime Membership Variant

```
ALL standard memberships have a "Daytime" version option.

Daytime Membership Features:
- Reduced joining/initiation fee
- Restricted access hours

Access Hours:
- Monday-Friday: 7:00 AM - 4:00 PM only
- Saturday-Sunday: Anytime (no restriction)

CHECK-IN VALIDATION:
- IF membership.is_daytime = TRUE
- AND current_day IN ['Monday','Tuesday','Wednesday','Thursday','Friday']
- AND (current_time < 07:00 OR current_time > 16:00)
- THEN display_alert("DAYTIME MEMBERSHIP - Outside allowed hours")
- Allow check-in but flag for staff attention
```

```php
// Daytime membership check-in validation
function validateDaytimeCheckin($membership) {
    if (!$membership['is_daytime']) {
        return ['valid' => true];
    }

    $dayOfWeek = date('N'); // 1=Monday, 7=Sunday
    $currentHour = (int)date('H');
    $currentMinute = (int)date('i');

    // Weekend - always allowed
    if ($dayOfWeek >= 6) {
        return ['valid' => true];
    }

    // Weekday - check hours (7:00 AM - 4:00 PM)
    $currentTime = $currentHour * 60 + $currentMinute;
    $startTime = 7 * 60;  // 7:00 AM
    $endTime = 16 * 60;   // 4:00 PM

    if ($currentTime < $startTime || $currentTime > $endTime) {
        return [
            'valid' => false,
            'alert' => 'DAYTIME MEMBERSHIP - Checking in outside allowed hours (M-F 7am-4pm)'
        ];
    }

    return ['valid' => true];
}
```

### Rule 3: Extended Family Member

```
SPECIAL MEMBERSHIP TYPE - Display reminder popup when creating:

Eligibility:
1. Must be living in the household
2. For family members who do NOT qualify for regular membership
3. Typically: children over 25, parents, other relatives

Billing Requirement:
- MUST be billed to primary member's account
- Extended family member CANNOT pay for their own membership
- Link payment to primary member's payment method

UI POPUP (display when creating this membership type):
┌─────────────────────────────────────────────────────────────┐
│  EXTENDED FAMILY MEMBER - Staff Reminder                    │
├─────────────────────────────────────────────────────────────┤
│  Please confirm with the member:                            │
│                                                             │
│  ☐ Does this person live in your household?                 │
│  ☐ This membership will be billed to YOUR account           │
│    (extended family cannot pay separately)                  │
│                                                             │
│  This membership is for family members over 25 who          │
│  live with the primary member.                              │
│                                                             │
│                              [Cancel]  [Confirm & Continue] │
└─────────────────────────────────────────────────────────────┘
```

### Rule 4: Student Membership

```
MINIMUM TERM: 2 months

UI POPUP (display when creating student membership):
┌─────────────────────────────────────────────────────────────┐
│  STUDENT MEMBERSHIP - Staff Reminder                        │
├─────────────────────────────────────────────────────────────┤
│  Please inform the member:                                  │
│                                                             │
│  "Student memberships require a 2-month minimum             │
│   commitment."                                              │
│                                                             │
│                              [Cancel]  [Confirm & Continue] │
└─────────────────────────────────────────────────────────────┘
```

### Rule 5: Temporary Memberships

```
PRICING: Always full price - NO prorating

EXPIRATION:
- Weekly: Expires exactly 7 days from start date
- Monthly: Expires exactly 1 month from start date

Auto-calculate expiration:
$expirationDate = ($type == 'weekly')
    ? $startDate->modify('+7 days')
    : $startDate->modify('+1 month');
```

```php
// Temporary membership expiration
function calculateTempExpiration($startDate, $durationType) {
    $expiration = new DateTime($startDate);

    switch ($durationType) {
        case 'weekly':
            $expiration->modify('+7 days');
            break;
        case 'monthly':
            $expiration->modify('+1 month');
            break;
    }

    return $expiration->format('Y-m-d');
}
```

### Rule 6: Therapy Memberships

```
CRITICAL TRACKING REQUIRED

Pricing & Duration:
- Always full price - NO prorating
- Always expires 1 month from start date

Documentation Required:
- Must have doctor's note OR physical therapy note on file
- Note must be obtained BEFORE membership starts

Calendar Year Limit:
- MAXIMUM 4 months per calendar year
- Track: therapy_months_used (reset each Jan 1)

Renewal Rules:
- If member uses less than 4 months, they need a NEW note to resume
- Example: Used 2 months, stopped, wants 2 more = needs new PT/doctor note
- NEVER allow more than 4 months total in a calendar year

VALIDATION:
IF therapy_months_used >= 4 THEN
    BLOCK with message: "Maximum therapy membership months reached
                         for this calendar year (4 of 4 used)"
```

```php
// Therapy membership validation
function canAddTherapyMembership($memberId, $year = null) {
    $year = $year ?? date('Y');

    $monthsUsed = getTherapyMonthsUsed($memberId, $year);

    if ($monthsUsed >= 4) {
        return [
            'allowed' => false,
            'message' => "Maximum therapy membership months reached for $year (4 of 4 used)",
            'months_used' => $monthsUsed,
            'months_remaining' => 0
        ];
    }

    return [
        'allowed' => true,
        'months_used' => $monthsUsed,
        'months_remaining' => 4 - $monthsUsed,
        'requires_new_note' => $monthsUsed > 0 && !hasActiveTherapyMembership($memberId)
    ];
}
```

### Rule 7: Playroom Memberships

```
No special rules.

Eligibility: If child meets playroom age requirements,
             they can have this membership.

Tiers based on number of children:
- One child
- Two children
- Three children
```

### Rule 8: Personal Training - Non-Member Fee

```
NON-MEMBER SURCHARGE: $7.00 per session

If purchaser is NOT an active member:
- Add $7.00 × number of sessions to total price
- Example: 10-pack = base price + ($7.00 × 10) = base price + $70.00

UI VALIDATION:
- Check if purchaser has active membership
- IF no active membership THEN display reminder:

┌─────────────────────────────────────────────────────────────┐
│  ⚠️  NON-MEMBER TRAINING PURCHASE                           │
├─────────────────────────────────────────────────────────────┤
│  This person is not an active member.                       │
│                                                             │
│  Non-member fee: $7.00 × [sessions] = $[total]              │
│                                                             │
│  ☐ I have added the non-member fee to this purchase         │
│                                                             │
│                              [Cancel]  [Confirm & Continue] │
└─────────────────────────────────────────────────────────────┘
```

```php
// Personal training non-member fee calculation
function calculateTrainingPrice($basePrice, $sessions, $isActiveMember) {
    $nonMemberFeePerSession = 7.00;

    $nonMemberSurcharge = $isActiveMember ? 0 : ($nonMemberFeePerSession * $sessions);

    return [
        'base_price' => $basePrice,
        'sessions' => $sessions,
        'is_member' => $isActiveMember,
        'non_member_surcharge' => $nonMemberSurcharge,
        'total' => $basePrice + $nonMemberSurcharge
    ];
}
```

---

## System-Wide Features Requested

### Feature 1: Payment Failed Alert

```
CURRENT: Account shows "red" when payment fails on the 15th
REQUESTED: More descriptive alert

When CC fails on billing date (15th):
- Display prominent alert at check-in
- Message: "PAYMENT UPDATE NEEDED"
- Should be automatic (not relying on manual notes)

UI at Check-In:
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  PAYMENT UPDATE NEEDED                                  │
│                                                             │
│  The payment method on file failed on [date].               │
│  Please update payment information.                         │
│                                                             │
│                                          [Update Now]       │
└─────────────────────────────────────────────────────────────┘
```

```php
// Payment status check
function getPaymentAlertStatus($member) {
    if ($member['payment_failed']) {
        return [
            'show_alert' => true,
            'alert_type' => 'payment_failed',
            'message' => 'PAYMENT UPDATE NEEDED',
            'details' => 'Payment method failed on ' . $member['last_payment_attempt_date'],
            'failed_date' => $member['last_payment_attempt_date']
        ];
    }
    return ['show_alert' => false];
}
```

### Feature 2: Required Fields for New Members

```
MANDATORY FIELDS when adding a new member:
1. Email address (required - cannot complete without)
2. Phone number (required - cannot complete without)

STRONGLY RECOMMENDED:
3. Credit card or bank account (display reminder if missing)

VALIDATION:
- Block form submission if email is empty
- Block form submission if phone is empty
- Display warning (but allow continue) if no payment method

UI Warning for missing payment method:
┌─────────────────────────────────────────────────────────────┐
│  ⚠️  NO PAYMENT METHOD                                      │
├─────────────────────────────────────────────────────────────┤
│  This member does not have a credit card or bank account    │
│  on file.                                                   │
│                                                             │
│  Would you like to add one now?                             │
│                                                             │
│                     [Add Payment Method]  [Skip for Now]    │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Structure (JSON)

```json
{
  "location": "Timberhill",
  "last_updated": "2026-01",
  "business_rules": {
    "family_member_max_age": 25,
    "family_member_must_cohabitate": true,
    "daytime_hours": {
      "weekday_start": "07:00",
      "weekday_end": "16:00",
      "weekend": "unrestricted"
    },
    "therapy_max_months_per_year": 4,
    "therapy_requires_documentation": true,
    "student_minimum_months": 2,
    "non_member_training_fee_per_session": 7.00,
    "required_member_fields": ["email", "phone"],
    "recommended_member_fields": ["payment_method"]
  }
}
```

---

## Membership Flags & Attributes

| Flag | Type | Description |
|------|------|-------------|
| `is_daytime` | boolean | Restricted to daytime hours M-F |
| `is_senior` | boolean | Senior pricing tier |
| `is_platinum` | boolean | Platinum/premium tier |
| `is_temporary` | boolean | Fixed-term membership |
| `is_therapy` | boolean | Requires medical documentation |
| `requires_cohabitation` | boolean | Members must live together |
| `max_member_age` | integer | Maximum age for additional members |
| `duration_type` | enum | 'ongoing', 'weekly', 'monthly' |
| `min_term_months` | integer | Minimum commitment period |

---

## Database Schema Additions (Timberhill-specific)

```sql
-- Additional columns for membership_plans
ALTER TABLE membership_plans ADD COLUMN is_daytime BOOLEAN DEFAULT FALSE;
ALTER TABLE membership_plans ADD COLUMN is_senior BOOLEAN DEFAULT FALSE;
ALTER TABLE membership_plans ADD COLUMN is_platinum BOOLEAN DEFAULT FALSE;
ALTER TABLE membership_plans ADD COLUMN is_temporary BOOLEAN DEFAULT FALSE;
ALTER TABLE membership_plans ADD COLUMN is_therapy BOOLEAN DEFAULT FALSE;
ALTER TABLE membership_plans ADD COLUMN duration_type ENUM('ongoing', 'weekly', 'monthly') DEFAULT 'ongoing';
ALTER TABLE membership_plans ADD COLUMN min_term_months INT DEFAULT NULL;
ALTER TABLE membership_plans ADD COLUMN max_member_age INT DEFAULT NULL;
ALTER TABLE membership_plans ADD COLUMN requires_cohabitation BOOLEAN DEFAULT FALSE;

-- Therapy tracking table
CREATE TABLE member_therapy_usage (
    id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    calendar_year INT NOT NULL,
    months_used INT DEFAULT 0,
    documentation_on_file BOOLEAN DEFAULT FALSE,
    documentation_date DATE NULL,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY unique_member_year (member_id, calendar_year),
    FOREIGN KEY (member_id) REFERENCES members(id)
);

-- Payment failure tracking
ALTER TABLE members ADD COLUMN payment_failed BOOLEAN DEFAULT FALSE;
ALTER TABLE members ADD COLUMN last_payment_attempt_date DATE NULL;
ALTER TABLE members ADD COLUMN payment_failure_reason VARCHAR(255) NULL;
```

---

## Staff Reminder Popups Summary

| Trigger | Popup Title | Key Message |
|---------|-------------|-------------|
| Creating Extended Family Member | "Extended Family Member - Staff Reminder" | Confirm living in household, will bill to primary |
| Creating Student Membership | "Student Membership - Staff Reminder" | 2-month minimum commitment |
| Selling Training to Non-Member | "Non-Member Training Purchase" | Add $7/session fee |
| Member has no payment method | "No Payment Method" | Recommend adding payment info |
| Daytime member checks in off-hours | "Daytime Membership Alert" | Outside allowed hours (M-F 7am-4pm) |
| Adding family member age 26+ | "Eligibility Error" | Block - must be under 26 |
| Payment failed on 15th | "Payment Update Needed" | CC on file didn't work |

---

## Questions for Clarification

> **Note to staff:** Please confirm with developer:

1. **Health Club vs Full Club** - What's the difference in facility access?
2. **Senior Age Threshold** - At what age does someone qualify for senior rates?
3. **Platinum Benefits** - What additional features do Platinum members get?
4. **Playroom Age Range** - What ages are eligible for playroom?
5. **Billing Date** - Is the 15th the standard billing date for all members?
6. **Daytime Weekend Hours** - "Anytime" on weekends - is there still an opening/closing time?

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-01 | Initial documentation | Staff |
