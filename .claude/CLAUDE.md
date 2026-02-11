# PPCPilot.org - Agent Context

## Project Overview

PPCPilot.org is a suite of tools for the powered parachute (PPC) community. It has three main components: a web app, a mobile app, and flight school training content. All are maintained by Joel Wehr.

## Directory Structure

```
ppcpilot-org/
├── ppc-pilot-web/          # Django + React web app (ppcpilot.org)
│   ├── backend/            # Django REST API
│   ├── frontend/           # React + TypeScript + Tailwind
│   └── deploy/             # Nginx, Gunicorn, deploy scripts
├── ppc-pilot-maui/         # .NET MAUI 10 mobile app
│   ├── Models/
│   ├── ViewModels/
│   ├── Views/
│   └── Services/
└── ppc-flight-school/      # Training content & platform
    ├── ppc-knowledgebase/  # 11 training modules (119k words)
    ├── ppc-mobile-app/     # Legacy MAUI copy (use ppc-pilot-maui instead)
    └── ppc-training-platform/ # Planned AI training platform
```

Each sub-folder is its own git repository. The parent `ppcpilot-org` repo contains only documentation and configuration.

## GitHub Repositories

| Local Folder | GitHub Remote | Visibility |
|---|---|---|
| ppc-pilot-web | joel-wehr/ppc-pilot-web | Private |
| ppc-pilot-maui | joel-wehr/ppc-mobile-app | Public |
| ppc-flight-school/ppc-knowledgebase | joel-wehr/ppc-knowledgebase | Public |
| ppc-flight-school/ppc-training-platform | joel-wehr/ppc-training-platform | Private |

**Note:** `ppc-flight-school/ppc-mobile-app` shares the same remote as `ppc-pilot-maui`. The canonical MAUI app location is `ppc-pilot-maui`.

## Web App (ppc-pilot-web)

### Backend (Django)
- **Settings module:** `ppc_admin.settings`
- **Key apps:** `accounts`, `flights`, `equipment`, `maintenance`, `checklists`, `community`
- **Auth:** Google OAuth via django-allauth + dj-rest-auth (JWT)
- **API prefix:** `/api/v1/`
- **Sync endpoints:** `GET /api/v1/sync/pull/`, `POST /api/v1/sync/push/`

### Frontend (React)
- **Build tool:** Vite
- **Styling:** Tailwind CSS with custom navy color
- **Data fetching:** TanStack Query (React Query)
- **Routing:** React Router v6
- **Components:** Custom DataTable, Pagination, Sidebar layout

### Deployment
- **Server:** AWS Lightsail (Ubuntu) under `joelwehr` AWS profile
- **IP:** 100.50.75.194
- **Domain:** ppcpilot.org (SSL via Let's Encrypt)
- **SSH:** `ssh -i ~/.ssh/lightsail-joelwehr-key.pem ubuntu@100.50.75.194`
- **Deploy:** `cd /opt/ppc-pilot-web && bash deploy/deploy.sh`
- **Server venv:** `/opt/ppc-pilot-web/backend/venv/`

### Google OAuth
- **Client ID:** `341895943811-4agpkvfpq3ib8uvirejulntpmvpq68lm.apps.googleusercontent.com`
- Configured in Django admin Social Applications

## Mobile App (ppc-pilot-maui)

### Tech
- .NET MAUI 10 targeting iOS, Android, Windows, MacCatalyst
- SQLite database (`pegasus_flight.db3`)
- MVVM with CommunityToolkit.Mvvm
- UraniumUI components

### Key Features
- 7 checklists (Preflight, Warm Up, Wing Layout, Pre-Start, In-Flight Practice, Post-Flight)
- In-Flight Practice uses counters (not checkboxes) - see `HasCounter` property
- Flight logging with automatic date grouping
- Background sync to ppcpilot.org every 5 minutes via `ApiSyncService`

### Build
- Android keystore: `pegasus-flight.keystore` (alias: `pegasus-flight`)
- iOS: Automatic Provisioning
- Android emulator: `pixel_7_-_api_35` (Pixel 7, API 35)
- JDK: `C:\Program Files\Android\openjdk\jdk-21.0.8`

### Known Gotchas
- Border can only have ONE child in XAML (wrapper Grid at ChecklistDetailPage.xaml line 57)
- SQLite can't use `.Date` in LINQ queries - load to memory first
- `ChecklistItem.Clone()` must preserve `HasCounter` property
- Pin `Microsoft.Maui.Controls` version explicitly (don't rely on `$(MauiVersion)`)

## Flight School (ppc-flight-school)

### ppc-knowledgebase
11 training modules covering PPC foundations, regulations, weather, aircraft systems, flight operations, emergency procedures, practical skills, advanced topics, maintenance, and cross-country flying.

### ppc-training-platform (Planned)
AI-enhanced web training platform. Planned stack: Next.js 14, FastAPI, PostgreSQL, Claude API.

## Working Across Projects

### Community/Resource Data
The `community` Django app manages locations (flight schools, PPC manufacturers, component manufacturers, mechanics, flight clubs, airports). The `CommunityLocation` model has `location_type` choices:
- 0 = Flight School
- 1 = PPC Manufacturer
- 2 = Component Manufacturer
- 3 = Mechanic
- 4 = Flight Club
- 5 = Airport

### Data Sync (Mobile ↔ Web)
- MAUI models have `RemoteId` and `SyncStatus` (0=Synced, 1=Modified, 2=New)
- Django models have `user` FK (null=True for backward compat)
- All ViewSets filter by `request.user`
- JWT tokens stored in MAUI SecureStorage

### Running Agents
Claude agents should be run from the `ppcpilot-org` root so they have visibility into all sub-projects. Each sub-project also has its own `.claude/CLAUDE.md` with project-specific instructions.
