# PPCPilot Mobile App Specification & Sync Contract

**Version:** 1.0  
**Date:** 2026-05-30  
**Status:** Final (source-of-truth for iOS and Android native builds)  
**Author:** Web Explorer Agent  
**Audience:** iOS (Swift/SwiftUI) and Android (Kotlin/Compose) development teams

---

## 1. Executive Summary

This specification defines the complete contract for native iOS and Android apps built on the PPCPilot.org ecosystem. Both apps:
- Share identical data models and screen flows (7 checklists, flight logging, equipment mgmt, maintenance tracking)
- Sync bidirectionally with the Django backend via `/api/v1/sync/{pull,push}` endpoints every 5 minutes
- Use Google OAuth for authentication with JWT token storage
- Maintain local SQLite databases with conflict resolution (server wins)
- Support offline-first operation with automatic reconciliation

This document is the single source of truth. iOS and Android apps must implement every requirement identically to ensure data integrity and user experience parity.

---

## 2. App Screens & Navigation (13 Screens)

| Screen | Tab/Menu | Purpose | Data Source | Sync Dir |
|--------|----------|---------|-------------|----------|
| **Checklists Tab** |
| Checklists List | Checklists | Browse 7 checklist types; select to complete | Pull `/sync/pull/` → ChecklistTemplate | Pull |
| Checklist Detail | Checklists | Checkboxes + counters (In-Flight Practice); mark items | ChecklistLog items (local) | Push on save |
| **Flight Operations Tab** |
| Dashboard | Home | Recent flights, hours, landings, currency, medical status | Pull → Flight + calculations | Pull |
| Flight Log | Flights | List flights grouped by date | Pull → Flight list | Pull |
| Flight Detail | Flights | View/edit flight (location, weather, fuel, hobbs) | Pull → single Flight | Push on save |
| New Flight | Flights | Create flight | UI (new local) | Push after |
| **Equipment Tab** |
| Equipment List | Equipment | Browse PPC frames (aircraft) | Pull → PpcFrame list | Pull |
| Equipment Detail | Equipment | View/edit frame + engine, wing, propeller | Pull → PpcFrame | Push on save |
| New Equipment | Equipment | Create frame | UI (new local) | Push after |
| **Maintenance Tab** |
| Maintenance Log | Maintenance | List maintenance events; show upcoming | Pull → MaintenanceLog | Pull |
| Maintenance Detail | Maintenance | View/edit maintenance entry | Pull → MaintenanceLog | Push on save |
| New Maintenance | Maintenance | Create maintenance entry | UI (new local) | Push after |
| **Pilot Profile** |
| Pilot Profile | More/Settings | View/edit pilot info (cert, medical, endorsements) | Pull → PilotProfile | Push on save |

---

## 3. Core Data Models & Sync Semantics

### 3.1 Sync Status Enumeration

All syncable models carry a **SyncStatus** integer:
```
0 = Synced        (last push succeeded, in-sync with server)
1 = Modified      (local changes pending push)
2 = New           (created locally, never sent to server)
```

### 3.2 Remote ID & Identity Mapping

All syncable models have:
- **RemoteId** (nullable): ID assigned by Django after initial push
- **Local Id** (auto-increment): Primary key in local SQLite
- **CreatedAt** (ISO-8601 UTC): Immutable timestamp
- **ModifiedAt** (nullable ISO-8601 UTC): Latest local change

**Identity Flow:**
1. User creates Flight locally → Local ID = 42, RemoteId = null, SyncStatus = 2
2. Next sync → Push sends { local_id: 42, id: null, ...data }
3. Server returns { "flights": { "42": 123 } } (local 42 → remote 123)
4. App stores RemoteId = 123, SyncStatus = 0

### 3.3 Conflict Resolution

**Rule:** Server wins on all conflicts.

| Scenario | Behavior |
|----------|----------|
| Local modified, remote unchanged | Local change pushes; accepted |
| Remote modified, local unchanged | Pull overwrites local; SyncStatus → 0 |
| Both modified | Server version retained; SyncStatus → 0 |
| Local delete | Delete succeeds; remote deletion on next pull |

---

## 4. Data Models (Complete Schema)

### 4.1 Flight
**Table:** `flights` | **Sync:** Yes (user-owned)

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id (local) | int | false | SQLite AUTO_INCREMENT |
| RemoteId | int | true | Server-assigned |
| flight_date | date | false | YYYY-MM-DD |
| departure_time | datetime | true | ISO-8601 UTC |
| landing_time | datetime | true | ISO-8601 UTC |
| duration_minutes | int | true | Calculated: (landing - departure) / 60 |
| location | string(300) | true | Takeoff/landing location |
| weather_conditions | string(300) | true | METAR or notes |
| notes | text | true | Pilot narrative |
| ppc_frame_id | int | true | FK → PpcFrame.RemoteId |
| flight_type | enum | false | 0=Local, 1=Cross-Country, 2=Training-Dual, 3=Training-Solo, 4=Practice, 5=Check-Ride |
| departure_location | string(300) | true | ICAO or name |
| landing_location | string(300) | true | ICAO or name |
| route | string(500) | true | 61.51(b)(1) point/route of flight |
| aircraft_ident | string(30) | true | N-number/ident snapshot for the logbook entry |
| total_time | float | true | 61.51(b)(1) total flight time (hours); defaults from hours_flown/duration |
| pic_time | float | true | Pilot-in-command hours (61.51(b)(2)) |
| sic_time | float | true | Second-in-command hours |
| solo_time | float | true | Solo hours |
| dual_received | float | true | Flight training received from instructor |
| cross_country_time | float | true | Cross-country hours |
| day_time | float | true | Day hours (61.51(b)(3)) |
| night_time | float | true | Night hours (61.51(b)(3)) |
| takeoff_count | int | true | Number of takeoffs |
| landing_count | int | true | Number of landings |
| night_landing_count | int | true | Night landings (61.57 currency) |
| fuel_start_gallons | float | true | Gallons at departure |
| fuel_end_gallons | float | true | Gallons at landing |
| fuel_consumed | float | true | Calculated: start - end |
| hobbs_start | float | true | Engine hours at start |
| hobbs_end | float | true | Engine hours at landing |
| hours_flown | float | true | Calculated: hobbs_end - hobbs_start |
| cruise_altitude_agl | int | true | Feet AGL |
| max_altitude | int | true | Feet MSL |
| wind_speed | float | true | Knots |
| wind_direction | string(10) | true | E.g., "NW" |
| wind_gusts | float | true | Knots |
| temperature | float | true | °C or °F (disambiguate in UI) |
| visibility | float | true | Statute miles or km |
| ceiling | int | true | Feet AGL |
| altimeter_setting | float | true | Inches Hg |
| density_altitude | int | true | Feet |
| weather_notes | text | true | Detailed narrative |
| instructor_name | string(200) | true | Dual flight instructor |
| maintenance_notes | text | true | Issues noted |
| lessons_learned | text | true | Debrief notes |
| gps_latitude | float | true | Decimal degrees |
| gps_longitude | float | true | Decimal degrees |
| created_at | datetime | false | ISO-8601 UTC (immutable) |
| modified_at | datetime | true | ISO-8601 UTC (updated on edit) |
| SyncStatus | int | false | 0=Synced, 1=Modified, 2=New |

**UI Behavior:**
- Date pre-filled from current date or first completed checklist
- Duration auto-calculated on landing_time entry
- All calculated fields computed before sync

---

### 4.2 PpcFrame (Aircraft)
**Table:** `ppc_frames` | **Sync:** Yes (user-owned)

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id (local) | int | false | SQLite AUTO_INCREMENT |
| RemoteId | int | true | Server-assigned |
| manufacturer | string(200) | true | E.g., "Falcon Ultralight" |
| model | string(200) | true | E.g., "F4" |
| serial_number | string(100) | true | Frame serial |
| n_number | string(20) | true | U.S. registration |
| year | int | true | Year manufactured |
| empty_weight | float | true | Pounds |
| seat_config | enum | false | 0=Solo, 1=Tandem |
| registration_class | enum | false | 0=Part103, 1=E-LSA, 2=S-LSA, 3=Experimental-AB, 4=Other |
| registration_date | date | true | FAA registration date (LSA only) |
| airworthiness_date | date | true | Airworthiness cert date (LSA only) |
| seats | int | true | Number of seats |
| max_gross_weight | float | true | Pounds (MTOW) |
| fuel_capacity_gallons | float | true | US gallons usable |
| base_location | string(200) | true | Home field |
| is_active | boolean | false | true = operational |
| notes | text | true | Custom notes |
| created_at | datetime | false | ISO-8601 UTC |
| modified_at | datetime | true | ISO-8601 UTC |
| SyncStatus | int | false | 0=Synced, 1=Modified, 2=New |

**Related Models:**
- Engine (OneToOne, ppc_frame_id FK)
- Wing (OneToOne, ppc_frame_id FK)
- Propeller (OneToOne, ppc_frame_id FK)

**UI Behavior:**
- Display name: "{manufacturer} {model}"
- Mark inactive frames with strikethrough

---

### 4.3 Engine, Wing, Propeller
**Tables:** `engines`, `wings`, `propellers` | **Sync:** Yes (via parent ppc_frame_id)

**Engine:**
| Field | Type | Notes |
|-------|------|-------|
| id, RemoteId, ppc_frame_id | int | (standard) |
| manufacturer, model, serial_number | string | Rotax 912 iS, etc. |
| engine_type | enum | 0=Two-Stroke, 1=Four-Stroke |
| cooling_type | enum | 0=Air, 1=Liquid |
| total_hours | float | Cumulative engine hours |
| tbo_hours | float | Time Between Overhaul limit (Rotax 582≈300h, 912≈2000h) |
| last_overhaul_date | date | Last major overhaul |
| last_overhaul_hours | float | Engine hours at last overhaul |
| horsepower | float | Rated HP |
| displacement_cc | int | Displacement (cc) |
| cylinders | int | Cylinder count |
| fuel_type | enum | 0=Auto91, 1=Auto93, 2=100LL, 3=MogasE10, 4=Other |
| oil_type | string | Oil spec |
| gearbox_ratio | string | e.g. 2.62:1 |
| notes | text | Valve checks, etc. |
| created_at, modified_at, SyncStatus | datetime/int | (standard) |

**Computed:** `hours_until_tbo = tbo_hours - total_hours` (warn if < 50 hours)

**Wing:**
| Field | Type | Notes |
|-------|------|-------|
| id, RemoteId, ppc_frame_id | int | (standard) |
| manufacturer, model | string | BSD XS Streamer |
| size_sq_ft | float | Wing area |
| cell_count | int | Number of cells (7 or 9 typical) |
| wing_type | enum | 0=Square, 1=Elliptical |
| span_ft | float | Span (ft) |
| chord_ft | float | Chord (ft) |
| aspect_ratio | float | Span/chord (~3:1 max for PPC) |
| line_count | int | Number of lines |
| glide_ratio | float | 3-4:1 square, 5-6:1 elliptical |
| max_gross_weight | float | Max certificated load (lbs) |
| weight_range_min / weight_range_max | float | Recommended hook-in weight (lbs) |
| color | string | Canopy color |
| total_hours | float | Cumulative hours |
| manufacture_date | date | DOC |
| last_inspection_date | date | Last condition check |
| notes | text | (standard) |

**Propeller:**
| Field | Type | Notes |
|-------|------|-------|
| id, RemoteId, ppc_frame_id | int | (standard) |
| manufacturer, model | string | Warp Drive 3-blade |
| diameter, pitch | float | Inches |
| material | **enum (int)** | **CHANGED 2026-06-02 (was free-text string):** 0=Wood, 1=Composite, 2=Carbon Fiber, 3=Aluminum, 4=Other. Serializer also returns read-only `material_display` (string). **Native DTOs must update `material` String→Int.** |
| blade_count | int | constrained to 2–6 |
| hub | string | Hub type |
| rotation | enum | 0=Pusher, 1=Tractor (PPC usually pusher) |

> **Sync-contract change log (2026-06-02, web PR #16):** `Propeller.material` changed free-text string → integer enum (+ read-only `material_display`). `Flight.wind_direction` now constrained to the 16-point compass + `VRB` (still a string). `CommunityLocation.location_type` added `6 = Maintenance & Repair`; `state` stays free-text (US codes constrained in UI only). **Only `Propeller.material` is a breaking type change for clients** — iOS/Android Propeller DTOs must treat `material` as Int (and may show `material_display`).
| ground_adjustable | boolean | Pitch ground-adjustable |
| total_hours | float | Cumulative hours |
| notes | text | (standard) |

---

### 4.4 ChecklistTemplate
**Table:** `checklist_templates` | **Sync:** Yes (user-owned)

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| RemoteId | int | true | Server-assigned |
| name | string(200) | false | E.g., "Preflight (Detailed)" |
| description | text | true | Help text |
| category | string(100) | true | Free-form grouping label ("Ground" / "Flight" / "Training" / "") — added 2026-05-30 |
| display_order | int | false | Sort order (0–10) |
| is_default | boolean | false | Pre-loaded default |
| is_active | boolean | false | Available for use |
| created_at, modified_at, SyncStatus | datetime/int | (standard) | (standard) |

**Standard default checklists (pulled from server, seeded per-user from the FAA handbook ground flow):**
`Pre-Flight (Detailed)` (0) · `Pre-Flight (Quick)` (1) · `Engine Start` (2) · `Warm-Up / Run-Up` (3) · `Wing Layout` (4) · `Before Takeoff` (5) · `In-Flight Checks` (6) · `Pre-Landing` (7) · `Landing` (8) · `Post Flight` (9) · `In-Flight Practice` (10, counter items). All `is_default = true`. Defaults are seeded **per user** (no shared `user=null` rows); a `POST /checklist-templates/seed-if-empty/` populates them on first use and `POST /checklist-templates/reset-defaults/` re-seeds only that user's default templates (custom templates preserved). See `docs/CHECKLISTS_REFERENCE.md` for the handbook sources.

---

### 4.5 ChecklistTemplateItem
**Table:** `checklist_template_items` | **Sync:** Yes (via template__user)

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| RemoteId | int | true | Server-assigned |
| template_id | int | false | FK → ChecklistTemplate |
| section | string(200) | true | E.g., "Engine", "Instruments" |
| description | string(500) | false | Item text |
| note | string(500) | true | Optional help/"why" text shown under the item — added 2026-05-30 |
| display_order | int | false | Sort within section |
| item_type | enum | false | 0=Checkbox, 1=Counter |
| is_required | boolean | false | Advisory items can be marked optional (default true) — added 2026-05-30 |
| created_at, modified_at, SyncStatus | datetime/int | (standard) | (standard) |

**In-Flight Practice Counters (item_type=1):**
- Steep turns left/right
- Stalls
- Slow flight
- etc.

**UI:** Show "-" and "+" buttons; display count center-aligned.

---

### 4.6 ChecklistLog & ChecklistLogItem
**Tables:** `checklist_logs`, `checklist_log_items` | **Sync:** Yes

**ChecklistLog:**
| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| RemoteId | int | true | Server-assigned |
| flight_id | int | true | FK → Flight (nullable) |
| checklist_type | int | false | DEPRECATED (kept for compat) |
| template_id | int | true | FK → ChecklistTemplate |
| completed_at | datetime | false | When marked complete (ISO-8601) |
| total_items | int | false | Count of items |
| checked_items | int | false | Count checked (0 for pure counters) |
| notes | text | true | Pilot notes |
| template_name | string(200) | true | Snapshot for display |
| created_at, modified_at, SyncStatus | datetime/int | (standard) | (standard) |

**ChecklistLogItem:**
| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| checklist_log_id | int | false | FK → ChecklistLog |
| template_item_id | int | true | FK → ChecklistTemplateItem |
| description | string(500) | false | Item text snapshot |
| section | string(200) | true | Section snapshot |
| note | string(500) | true | Help-text snapshot — added 2026-05-30 |
| item_type | enum | false | 0=Checkbox, 1=Counter |
| is_required | boolean | false | Required-flag snapshot (default true) — added 2026-05-30 |
| display_order | int | false | Sort order within the log (default 0) — added 2026-05-30 |
| is_checked | boolean | false | true if checked |
| count_value | int | false | For counters: final count |

**Linking to Flight:**
- If user completes checklist → Flight is auto-created by date (if not exists)
- flight_id populated automatically
- If flight deleted, log.flight_id set to null (log remains)

---

### 4.7 MaintenanceLog
**Table:** `maintenance_logs` | **Sync:** Yes (via ppc_frame__user) | Serializer: `fields = '__all__'`
**Updated 2026-05-30:** expanded to an FAA 14 CFR 43.9-style service record.

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| RemoteId | int | true | Server-assigned |
| ppc_frame_id | int | false | FK → PpcFrame (aircraft) |
| title | string(200) | true | Short label for the record (default '') |
| maintenance_date | date | false | When service was performed |
| maintenance_type | enum | false | 0=Inspection, 1=Repair, 2=Service, 3=Overhaul, 4=Modification |
| component | enum | false | 0=Airframe/Engine etc. (see MaintenanceComponent choices) |
| status | enum | false | MaintenanceStatus (default Completed) |
| description | text | true | Work performed |
| parts_used | text | true | Parts list |
| reference | string(300) | true | AD / SB / manual citation |
| cost | float | true | Total USD |
| parts_cost | float | true | Parts portion |
| labor_cost | float | true | Labor portion |
| labor_hours | float | true | Labor hours |
| engine_hours_at_service | float | true | Engine/Hobbs reading at time |
| aircraft_total_time | float | true | Aircraft total time at service |
| next_service_due_hours | float | true | Hours until next due (one-off) |
| next_service_due_date | date | true | Calendar date for next service (one-off) |
| performed_by | string(200) | true | Technician |
| vendor | string(200) | true | Shop/vendor |
| signature | string(200) | true | Sign-off name |
| certificate_number | string(100) | true | Mechanic/repairman cert # |
| return_to_service | bool | false | RTS sign-off (default false) |
| attachments | JSON list | true | `[{name, url}]` |
| notes | text | true | (standard) |
| created_at, modified_at, SyncStatus | datetime/int | (standard) | (standard) |
| ppc_frame_name, *_display | string | read-only | Convenience read-only fields from serializer |

---

### 4.7b MaintenanceSchedule  *(NEW model — 2026-05-30)*
**Table:** `maintenance_schedules` | **Sync:** Yes (via ppc_frame__user; in MODEL_MAP + USER_FK-style filter) | Serializer: `fields = '__all__'`
Per-aircraft recurring service rules with computed due status.

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| RemoteId | int | true | Server-assigned |
| ppc_frame_id | int | false | FK → PpcFrame |
| title | string(200) | false | e.g. "Oil & filter change" |
| component | enum | false | MaintenanceComponent |
| interval_hours | float | true | Recurrence in engine hours (nullable) |
| interval_months | int | true | Recurrence in months (nullable) |
| last_done_hours | float | true | Engine hours at last completion |
| last_done_date | date | true | Date of last completion |
| is_active | bool | false | default true |
| notes | text | true | |
| created_at, modified_at, SyncStatus | datetime/int | (standard) | (standard) |

**Read-only computed (from serializer; do not store):**
| Field | Type | Notes |
|-------|------|-------|
| next_due_hours | float | last_done_hours + interval_hours |
| next_due_date | date | last_done_date + interval_months |
| current_engine_hours | float | from linked Engine.total_hours |
| service_status | string | `'ok'` / `'due_soon'` / `'overdue'` — whichever-comes-first of hours/date; default thresholds 10h / 30d (per-user `config.maintenance` overrides) |

**Dashboard / Upcoming Service:** merge `maintenance_schedules` (by service_status) with one-off `maintenance_logs` that have next_service_due_*; group Overdue / Due Soon / On Track. Mobile can replicate the status logic locally from interval + last_done + current engine hours.

---

### 4.8 PilotProfile
**Table:** `pilot_profiles` | **Sync:** Yes (user-owned)

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | int | false | Auto-increment |
| RemoteId | int | true | Server-assigned |
| full_name | string(200) | true | Pilot legal name |
| certificate_type | enum | false | 0=None, 1=Student, 2=Sport, 3=Private, 4=Recreational, 5=Commercial, **6=ATP** (added 2026-06-02) |
| certificate_number | string(50) | true | FAA certificate |
| certificate_issue_date | date | true | Certificate issue date |
| ratings | text | true | JSON array of category/class codes (e.g. ["PPC-L","ASEL"]). Codes: ASEL/AMEL/ASES/AMES/GLIDER/RH/RG/PPC-L/PPC-S/WSC-L/WSC-S/BALLOON |
| instructor_certificates | text (JSON) | true | **NEW 2026-06-02** — JSON array of instructor codes: CFI/CFII/MEI/AGI/IGI (handle like ratings/endorsements) |
| instrument_rating | bool | false | **NEW 2026-06-02** — default false |
| pilot_status | enum | true | **NEW 2026-06-02** — 0=Pre-solo, 1=Student in training, 2=Rated–current, 3=Rated–lapsed, 4=Working toward a rating; null=unspecified |
| goal_certificate | enum | true | **NEW 2026-06-02** — target cert level, same taxonomy as certificate_type (0–6) |
| goal_rating | string | true | **NEW 2026-06-02** — target category/class code (e.g. "PPC-L"); blank=none |
| goal_target_date | date | true | **NEW 2026-06-02** — target date (ISO YYYY-MM-DD) |
| flight_review_date | date | true | Last 61.56 flight review (24-cal-month currency) |
| medical_type | enum | false | 0=Driver's License, 1=Third-Class, 2=BasicMed, 3=Second, 4=First |
| medical_expiration | date | true | Expiry date (triggers dashboard warning) |
| basicmed_date | date | true | Date of last BasicMed exam |
| max_wind_speed | float | true | Knots |
| max_crosswind | float | true | Knots |
| min_visibility | float | true | Statute miles |
| min_ceiling | float | true | Feet AGL |
| emergency_contact_name | string(200) | true | POC name |
| emergency_contact_phone | string(20) | true | Phone number |
| endorsements | text | true | JSON object (e.g., { "tailwheel": true }) |
| created_at, modified_at, SyncStatus | datetime/int | (standard) | (standard) |

Serializer also returns read-only display fields on pull: `certificate_type_display`, `medical_type_display`, `pilot_status_display`, `goal_certificate_display`.

> **Sync-contract change (2026-06-02, web PR #19 "rich Pilot Profile"):** all ADDITIVE — `certificate_type` gained `6=ATP` (existing 0–5 unchanged); new fields `instructor_certificates` (JSON), `instrument_rating` (bool), `pilot_status` (nullable enum), `goal_certificate` (nullable enum), `goal_rating` (string), `goal_target_date` (date). Backward-compatible — existing clients keep working until they opt in. Native DTO update tracked in task #24. Full detail: `ppc-pilot-web/docs/PILOT_PROFILE_SPEC.md`.

---

## 5. Authentication & JWT Management

### 5.1 Google OAuth 2.0 Flow

**Endpoints:**
- Authorization: `https://accounts.google.com/o/oauth2/v2/auth`
- Token Exchange: `POST /api/v1/auth/google/`
- Token Refresh: `POST /api/v1/auth/token/refresh/`

**Native App Configuration:**

| Platform | Client ID | Redirect URI |
|----------|-----------|--------------|
| **iOS** | `341895943811-4agpkvfpq3ib8uvirejulntpmvpq68lm.apps.googleusercontent.com` | `com.joelwehr.ppcpilot:/oauth2redirect` |
| **Android** | `341895943811-4agpkvfpq3ib8uvirejulntpmvpq68lm.apps.googleusercontent.com` | `com.joelwehr.ppcpilot:/oauth2redirect` |

**Flow:**
1. User taps "Sign in with Google"
2. App opens system browser → Google OAuth consent
3. User authorizes → Browser redirects to `com.joelwehr.ppcpilot:/oauth2redirect?code=AUTH_CODE`
4. App intercepts → extracts `code`
5. App sends `POST /api/v1/auth/google/` { "access_token": AUTH_CODE }
6. Backend validates with Google → returns JWT tokens

### 5.2 JWT Token Storage & Refresh

**Response from `/api/v1/auth/google/`:**
```json
{
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Token Storage:**
- **iOS:** Keychain (SecureEnclaveBacked if available)
- **Android:** EncryptedSharedPreferences
- **Keys:**
  - `auth_jwt_access` → access token (short-lived, ~5 min)
  - `auth_jwt_refresh` → refresh token (long-lived, ~7 days)

**Token Refresh on 401:**
1. Extract `refresh` from secure storage
2. POST `/api/v1/auth/token/refresh/` { "refresh": TOKEN }
3. Store returned `access`
4. Retry original request

**Error Handling:**
- `401 Unauthorized` → Attempt refresh; if fails, redirect to login
- `4xx other` → Log; skip sync
- `5xx` → Log; skip sync

### 5.3 Passkey (WebAuthn / FIDO2) Authentication  *(NEW — 2026-05-30)*

Passwordless sign-in using platform passkeys via the Android **Credential Manager**.
Same RP as the web app, so a passkey created on the website also works in the app
and vice-versa. On success the endpoints return the **same `{access, refresh}` JWT pair**
as Google login — store and use them identically (Section 5.2).

**Relying Party / origin config (server-side, fixed):**

| Item | Value |
|---|---|
| RP ID | `ppcpilot.org` |
| RP name | `PPCPilot` |
| Web origin | `https://ppcpilot.org` |
| Android package | `org.ppcpilot.android` |
| Android signing SHA-256 | `64:B3:D5:5A:83:FA:AD:50:52:4A:A8:88:C9:A8:1A:E4:FE:B2:F3:49:8D:75:8A:0C:98:E0:D5:FB:D9:75:88:C3` |

The app origin is derived server-side as `android:apk-key-hash:<base64url(sha256(signing-cert))>`
and is accepted automatically — the app does not send an origin. Digital Asset Links are
served at **`https://ppcpilot.org/.well-known/assetlinks.json`** (associates the package above
with the RP so Credential Manager will issue/use passkeys for `ppcpilot.org`).

**Endpoint base:** `/api/v1/auth/passkey/`

All ceremony exchanges follow the standard **WebAuthn JSON** form (base64url strings, no
padding). The server returns an opaque `handle` that the client must echo back on the
matching `complete` call. Challenges are single-use and expire after 5 minutes.

#### Register a passkey (adds to the logged-in account — `Authorization: Bearer` required)

`POST /api/v1/auth/passkey/registration/begin/`  → body: `{}` (uses the bearer token's user)
```jsonc
// 200 response — pass `.publicKey` to Credential Manager CreatePublicKeyCredentialRequest
{
  "handle": "<opaque-string>",
  "publicKey": {
    "rp": { "id": "ppcpilot.org", "name": "PPCPilot" },
    "user": { "id": "<b64url>", "name": "<username>", "displayName": "<name>" },
    "challenge": "<b64url>",
    "pubKeyCredParams": [ { "type": "public-key", "alg": -7 }, /* ... */ ],
    "excludeCredentials": [ /* already-registered creds */ ],
    "authenticatorSelection": { "residentKey": "required", "userVerification": "required", "requireResidentKey": true }
  }
}
```
`POST /api/v1/auth/passkey/registration/complete/`  (`Authorization: Bearer` required)
```jsonc
// request — `credential` is the Credential Manager JSON response verbatim
{
  "handle": "<from begin>",
  "name": "Pixel 8",                 // optional user label
  "credential": {
    "id": "<b64url>", "rawId": "<b64url>", "type": "public-key",
    "response": {
      "clientDataJSON": "<b64url>",
      "attestationObject": "<b64url>",
      "transports": ["internal", "hybrid"]
    }
  }
}
// 201 response: the stored passkey
{ "id": 1, "credential_id": "<b64url>", "name": "Pixel 8", "transports": ["internal","hybrid"],
  "aaguid": "...", "sign_count": 0, "created_at": "...", "last_used_at": null }
```

#### Passwordless login (no auth header — this *is* the login)

`POST /api/v1/auth/passkey/authentication/begin/`  → body: `{}`  (usernameless / discoverable)
```jsonc
// 200 — pass `.publicKey` to Credential Manager GetPublicKeyCredentialOption
{
  "handle": "<opaque-string>",
  "publicKey": {
    "challenge": "<b64url>",
    "rpId": "ppcpilot.org",
    "userVerification": "required",
    "allowCredentials": []          // empty → any discoverable passkey for the RP
  }
}
```
`POST /api/v1/auth/passkey/authentication/complete/`  (AllowAny)
```jsonc
// request — `credential` is the Credential Manager assertion JSON verbatim
{
  "handle": "<from begin>",
  "credential": {
    "id": "<b64url>", "rawId": "<b64url>", "type": "public-key",
    "response": {
      "clientDataJSON": "<b64url>",
      "authenticatorData": "<b64url>",
      "signature": "<b64url>",
      "userHandle": "<b64url>"
    }
  }
}
// 200 response — SAME shape as Google login; store + use per Section 5.2
{ "access": "<jwt>", "refresh": "<jwt>" }
```
The server resolves the account from the credential, verifies the assertion (signature,
origin, RP ID, challenge, user-present + user-verified), enforces **sign-count replay
protection**, updates `last_used_at`, and issues the JWT pair.

#### Manage passkeys (`Authorization: Bearer` required)

| Method | Path | Result |
|---|---|---|
| `GET` | `/api/v1/auth/passkey/credentials/` | List the user's passkeys (array of the 201 shape above) |
| `DELETE` | `/api/v1/auth/passkey/credentials/<id>/` | Remove a passkey → `204` |

**Failure handling:** any verification failure, expired/reused `handle`, unknown credential,
or detected replay returns `400` with `{ "detail": "..." }`. Treat `400` as "fall back to
Google login / retry begin". `409` on register means the credential is already registered.

---

## 6. Sync Protocol

### 6.1 Pull (Server → Mobile)

**Request:**
```
GET /api/v1/sync/pull/?since=2026-05-30T12:34:56.123456Z
Authorization: Bearer ACCESS_TOKEN
```

**Response:**
```json
{
  "data": {
    "flights": [
      {
        "id": 123,
        "flight_date": "2026-05-30",
        "departure_time": "2026-05-30T06:00:00Z",
        "landing_time": "2026-05-30T07:30:00Z",
        ...all fields...
        "created_at": "2026-05-30T06:00:00Z",
        "modified_at": "2026-05-30T07:30:00Z"
      }
    ],
    "ppc_frames": [...],
    "engines": [...],
    "wings": [...],
    "propellers": [...],
    "pilot_profiles": [...],
    "checklist_templates": [...],
    "checklist_template_items": [...],
    "checklist_logs": [...],
    "checklist_log_items": [...],
    "maintenance_logs": [...],
    "maintenance_schedules": [...]
  },
  "server_time": "2026-05-30T12:34:56.123456Z"
}
```

**Processing:**
1. For each entity type in `data`:
   - Find local record by `RemoteId` (create if not found)
   - Overwrite all local fields with server values
   - Set `SyncStatus = 0` (Synced)
   - Set `RemoteId = response.id`
2. Store `server_time` for next pull delta

---

### 6.2 Push (Mobile → Server)

**Request:**
```
POST /api/v1/sync/push/
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "entities": {
    "flights": [
      {
        "local_id": 42,
        "id": null,
        "flight_date": "2026-05-30",
        ...all fields except SyncStatus, RemoteId...
      },
      {
        "local_id": 43,
        "id": 123,
        ...modified fields...
      }
    ],
    ...other entity types...
  }
}
```

**Important:**
- Only include entities with `SyncStatus != 0`
- Omit `SyncStatus` and `RemoteId` from body
- Include `local_id` for ID mapping
- Set `id` to `null` for new, `RemoteId` value for updates

**Response:**
```json
{
  "id_map": {
    "flights": {
      "42": 456,
      "43": 123
    }
  },
  "server_time": "2026-05-30T12:34:56.123456Z"
}
```

**Processing:**
1. For each local_id → server_id mapping:
   - Find local record by local_id
   - Update `RemoteId = server_id`
   - Set `SyncStatus = 0`
   - Save to DB
2. Store `server_time`

---

### 6.3 Sync Frequency & Lifecycle

**Frequency:** Every 5 minutes (300 seconds)

**Lifecycle:**
- **iOS:** Start on login via Timer; stop on logout
- **Android:** Start on login via WorkManager; stop on logout

**Trigger Points:**
- On app launch: immediate pull
- On login: start timer/worker
- On logout: stop timer/worker
- On network restore: immediate sync

---

## 7. Dashboard & Calculated Stats

| Stat | Calculation | Notes |
|------|-------------|-------|
| **Total Hours** | SUM(Flight.hours_flown) all time | Pulled + summed locally |
| **Total Flights** | COUNT(Flight.id) all time | Pulled + counted locally |
| **Total Landings** | SUM(Flight.landing_count) all time | Pulled + summed locally |
| **Flights This Month** | COUNT(Flight.id) where flight_date >= 1st of month | Local query |
| **Hours This Month** | SUM(Flight.hours_flown) where flight_date >= 1st of month | Local query |
| **Landings Last 90 Days** | SUM(Flight.landing_count) where flight_date >= 90 days ago | Local query |
| **Currency Status** | landings_90d >= 3: "Current"; >=1: "Warning"; else "Expired" | App logic |
| **Medical Status** | If PilotProfile.medical_expiration: >30d: "Current"; >0d: "Warning"; else "Expired" | App logic |
| **Recent Flights** | Last 5 flights by flight_date DESC | Local query |
| **Upcoming Maintenance** | MaintenanceLog where next_service_due_date within 30 days | Local query |

---

## 8. Offline Behavior

**User Actions Offline:**
- Create/Edit/Delete items locally → SyncStatus = 2/1/deleted
- View lists → Show cached local data
- Sync on reconnect → Pull latest; push pending changes

**On Reconnect:**
1. Detect network available
2. Trigger immediate sync (no wait for interval)
3. Pull latest server state
4. Push pending changes
5. Notify user of status

---

## 9. UI Requirements

### 9.1 In-Flight Practice Counter Items

```
  ┌────────────────────────┐
  │ Steep Turn (Left)      │
  │                        │
  │   ─         ┌──┐       │
  │    − │ 3 │ +│  │       │
  │   ─         │  │       │
  │                        │
  │ (description text)     │
  └────────────────────────┘
```

- Buttons `-` and `+` flanking count
- Font ≥24pt
- On `-`: decrement (min 0)
- On `+`: increment (no max)

### 9.2 Flight List Grouping

```
May 30, 2026
├─ 06:00 – 07:30 | Sylmar | 1h 30m
├─ 14:00 – 15:15 | Sylmar | 1h 15m

May 29, 2026
├─ 09:00 – 10:00 | Littlerock | 1h
```

---

## 10. Error Handling & Logging

**Log all errors:**
- Network errors (DNS, timeout, connection refused)
- HTTP status codes (401, 400, 5xx)
- Serialization errors
- Database errors

**User-facing messages:**
- "Sync failed: Check your internet connection. Will retry in 5 minutes."
- "Sync failed: Sign in required. Please re-authenticate."
- "Sync failed: Server error. Will retry in 5 minutes."

---

## 11. Testing Checklist

| Test | Expected Outcome |
|------|------------------|
| Sign in with Google | Tokens stored; user logged in |
| Token expiry + refresh | App silently refreshes; sync continues |
| Initial pull | All data downloaded; SyncStatus=0 |
| Modify flight + push | RemoteId assigned; SyncStatus=0 |
| Offline → online | Auto-sync triggered; changes pushed |
| Complete In-Flight checklist | Counter items saved with count_value |
| View Preflight checklist | Checkboxes displayed; can check/uncheck |
| Create flight | Auto-links to checklist; appears in list |
| Add frame + engine | Both synced; RemoteIds linked |
| Set maintenance date | Appears in dashboard; correct warning |

---

## 12. API Reference (Quick)

| Method | Endpoint | Auth | Body |
|--------|----------|------|------|
| `GET` | `/api/v1/sync/pull/?since=<ts>` | Bearer | None |
| `POST` | `/api/v1/sync/push/` | Bearer | { entities: {...} } |
| `POST` | `/api/v1/auth/google/` | None | { access_token: "..." } |
| `POST` | `/api/v1/auth/token/refresh/` | None | { refresh: "..." } |

---

**Document Status:** Final, Production Ready  
**Last Updated:** 2026-05-30  
**Next Review:** 2026-08-30
