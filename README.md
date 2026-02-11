# PPCPilot.org

A suite of tools and resources for the powered parachute (PPC) community by Joel Wehr.

**Website:** [ppcpilot.org](https://ppcpilot.org)

## Projects

### ppc-pilot-web
Django + React web application deployed at ppcpilot.org. Provides:
- Flight logging, checklists, equipment tracking, maintenance records
- Community resource directory (manufacturers, flight schools, component makers)
- Interactive resource map
- Google OAuth authentication
- Pilot profile management

**Tech:** Django 5, React 18, TypeScript, Tailwind CSS, PostgreSQL (SQLite dev), TanStack Query

### ppc-pilot-maui
.NET MAUI 10 cross-platform mobile app for iOS, Android, and Windows. Provides:
- 7 pre-flight, in-flight, and post-flight checklists
- Automatic flight log tracking grouped by date
- In-flight practice mode with counters for repeated maneuvers
- Syncs with ppcpilot.org via REST API

**Tech:** .NET MAUI 10, SQLite, MVVM (CommunityToolkit.Mvvm), UraniumUI

### ppc-flight-school
Training content and platform planning for PPC flight education:
- **ppc-knowledgebase** - 119,000 words of training content across 11 modules
- **ppc-mobile-app** - Legacy copy of MAUI app (canonical location is now `ppc-pilot-maui`)
- **ppc-training-platform** - Planned AI-enhanced web training platform (Next.js + FastAPI)

## Infrastructure

- **Web hosting:** AWS Lightsail (Ubuntu, Nginx, Gunicorn)
- **Domain:** ppcpilot.org with Let's Encrypt SSL
- **Auth:** Google OAuth via django-allauth + dj-rest-auth (JWT)
- **Sync:** REST API endpoints for mobile app data sync

## Getting Started

Each sub-project has its own git repository. See the CLAUDE.md file for detailed development instructions.
