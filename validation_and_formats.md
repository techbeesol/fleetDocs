# Validation Rules & Formats
## Fleet Management System

| Field | Value |
|---|---|
| Version | 1.0 (Draft) |
| Date | 2026-10-10 |
| Status | Draft — ready for review |
| Related | docs/data_model_and_api_contracts.md, docs/business_requirement.md |

> Every rule below is enforced on the **backend** (authoritative) and mirrored on the
> **frontend** for instant feedback. Frontend validation is UX only; never trust it
> for security.

---

## 1. Global Conventions

| Rule | Convention |
|---|---|
| Trimming | Trim leading/trailing whitespace on all text inputs before validating/saving. |
| Required vs optional | Required fields rejected with `422` if missing/empty. Optional fields may be null. |
| Max length default | Short text 100 chars, descriptions/notes 1000 chars (unless stated). |
| Case | Vehicle number stored UPPERCASE; emails/usernames not used (phone login). |
| Numbers | Money = `NUMERIC(12,2)` ≥ 0; KM = non-negative integers. |
| Empty string | Treated as "not provided" for optional fields. |

---

## 2. Formats (India-specific)

| Item | Rule | Example | Regex (guide) |
|---|---|---|---|
| **Phone** | Indian 10-digit mobile, first digit 6–9 | `9876543210` | `^[6-9]\d{9}$` |
| **Aadhaar** | 12 digits; **store full (encrypted at rest)**, **display masked** `XXXX XXXX 1234` | `1234 5678 9012` | `^\d{12}$` |
| **Vehicle number** | Indian RTO format, uppercase, loose validation | `TN01AB1234` | `^[A-Z]{2}\d{1,2}[A-Z]{1,3}\d{1,4}$` |
| **License number** | Alphanumeric, uppercase, 8–20 chars (required when designation = Driver) | `TN0120210001234` | `^[A-Z0-9-]{8,20}$` |
| **Date (display)** | `DD/MM/YYYY` | `09/10/2026` | — |
| **Date/time** | **Store UTC**, **display IST** (Asia/Kolkata, UTC+5:30) | — | — |
| **Currency** | `₹` with Indian grouping, 2 decimals | `₹1,00,000.00` | — |

> **Security note on Aadhaar:** Aadhaar is sensitive PII. Store it **encrypted at
> rest**, never log it, and only ever return the masked form in API responses. Expose
> the full value to no role in the UI.

---

## 3. Field-Level Rules by Entity

### 3.1 User
| Field | Required | Rule |
|---|---|---|
| name | Yes | 2–100 chars |
| phone | Yes | Indian mobile (`^[6-9]\d{9}$`); unique per company |
| password | Yes (create) | **Min 6 chars, no complexity** (driver-friendly); hashed (bcrypt) |
| role | Yes | one of `super_admin`, `admin`, `user` |
| designation | Yes | e.g. Driver, Mechanic, Co-pilot (free text, 2–50) |
| aadhaar | Optional | 12 digits if provided; stored encrypted, shown masked |
| dob | Optional | valid date; must be in the past |
| license_number | Conditional | required when designation = `Driver`; else optional |
| description | Optional | ≤ 1000 chars |

### 3.2 Vehicle
| Field | Required | Rule |
|---|---|---|
| vehicle_number | Yes | RTO format, uppercase; unique per company |
| insurance_till | Optional | valid date |
| insurance_status | Yes | `valid` / `expired` / `pending` |
| fc_till | Optional | valid date |
| fc_status | Yes | `valid` / `expired` / `pending` |
| rc_card_key | Optional | object-storage key |

### 3.3 Trip
| Field | Required | Rule |
|---|---|---|
| driver_id | Yes | existing user in same tenant |
| vehicle_id | Optional | existing vehicle in same tenant |
| location | Optional | ≤ 200 chars |
| trip_date | Optional | valid date |
| start_km | Optional | integer ≥ 0 |
| end_km | Optional | integer ≥ 0 and ≥ start_km |
| billed_amount | Optional | `NUMERIC(12,2)` ≥ 0 (fare) |
| fare_gst_amount | Optional | ≥ 0 if provided |
| advance_paid | Optional | ≥ 0 and ≤ billed_amount |
| balance_amount | — | server-computed (`billed_amount - advance_paid`) |
| payment_status | Yes | `unpaid` / `partial` / `paid` |
| status | — | server-managed (`draft`→`submitted`→`approved`/`rejected`) |

### 3.4 Trip Expense
| Field | Required | Rule |
|---|---|---|
| category | Yes | `fuel`/`toll`/`parking`/`permit`/`other` |
| amount | Yes | `NUMERIC(12,2)` ≥ 0 |
| gst_amount | Optional | ≥ 0 if provided; ≤ amount |
| description | Optional | ≤ 500 chars |

### 3.5 Service
| Field | Required | Rule |
|---|---|---|
| vehicle_id | Yes | existing vehicle in same tenant |
| service_cost | Yes | ≥ 0 |
| other_maintenance_cost | Yes | ≥ 0 |
| start_date / est_end_date / end_date | Optional | valid dates; end ≥ start when both set |
| core_service / spares / description | Optional | text |

### 3.6 Trip Planner
| Field | Required | Rule |
|---|---|---|
| destination_location | Yes | 2–200 chars |
| round_trip | Yes | boolean |
| estimated_km | Optional | integer ≥ 0 |
| planned_from / planned_to | Yes | valid dates; `planned_to ≥ planned_from` |
| total_days | — | server-computed, read-only |
| trip_notes | Optional | ≤ 1000 chars |

---

## 4. Standard Validation Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields are invalid.",
    "details": [
      { "field": "phone", "message": "Enter a valid 10-digit mobile number." },
      { "field": "end_km", "message": "End KM must be greater than or equal to Start KM." }
    ]
  }
}
```
HTTP status: `422`. Frontend shows messages inline under each field.

---

## 5. Message Style
- Clear, friendly, specific: "Enter a valid 10-digit mobile number" (not "Invalid input").
- Never leak internals or raw values.
- Same message text for the same rule across all three apps (keep in `frontend/shared`).

