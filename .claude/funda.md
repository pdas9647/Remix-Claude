# Funda — Customer Success Project

> Source: [Customer Success Projects](https://www.notion.so/26f1d464528f80f3a6fffc1fe4cf294e)
> Detail: [Funda](https://www.notion.so/26f1d464528f807f9782d5bffc5401a5)
> Notes: [FUNDA notes](https://www.notion.so/21f1d464528f804da24df85d82f35d9d)

---

## Context

- Funda is a WhatsApp community of ~500-700 Indian founders for networking and support
- Current website: https://funda.club (mostly informational, low usage)
- Not a traditional sales cycle — community, not chosen ICP
- Remix Customer Success team builds the project, not the customer

## Workspace

- **Workspace ID:** `sEt2qxPydL`
- **Workspace URL:** https://agt.remixlabs.com/ws/sEt2qxPydL

## Scope — First Release: Member Directory

- Web app published on remix.app, responsive design (mobile + laptop)
- **Authentication:** Google, Office365, Apple
- **Admin users (named: ~2):** Maintain the member directory, use LinkedIn clipper to populate info
- **Members (anonymous: ~600):** Browse/search directory, contact other members
- Members cannot update own profiles in first phase, do not sign in

### Member Profile Fields

- Name, Title, Company, LinkedIn URL
- Mobile (+ show to members toggle), Email (+ show to members toggle)
- Upload picture
- "Reach out to me for" tags

### Member Listing

- Search by name, email, company, tag
- Navigate to directory listing with contact options

## Future Phases

- Member network summary report (weekly/monthly changes, reach, companies, skills)
- Semantic search using Snowflake Cortex Search
- Members can update their own profiles

## Architecture / Apps

### 1. FUNDA DBP (Digital Business Profile) — Public Website

Screens: Home, About Us, Events (listing + detail + registration), Startup listing, Support/Offers, Blog (off-platform), Join (web-to-lead form)

### 2. Admin App — Multi-screen Web App

Login via LinkedIn OAuth. Screens: Member list, Request-to-join list, Startup list, Event list, Support/Offer list. Each list has detail view + add/edit/delete. Admin uses LinkedIn clipper to populate
member info.

### 3. Member App — Web App

Login via LinkedIn OAuth. Screens: Member directory (search), LinkedIn profile navigation, Company info, User profile edit.

## Services (Cloud Agents)

All generic CRUD + search services except:

- Event registration: specialised (checks capacity)
- Events, Startup directory, Support/Offers, Candidate-to-join, Members: all generic get/add/edit/delete/search

## Reusable Services from this Engagement

1. UI delivery pattern (Arvind/Petavue approach)
2. Semantic search using Snowflake
3. General person + company data model with services (underpins CRM, support, directory, sales use cases)
4. Modularised clipper configs (maintainable, customer-enhanceable)

## Key Components

- **LinkedIn clipper** — Chrome extension flow for admins to clip LinkedIn profile data
- **LinkedIn OAuth** — Authentication for admin and member apps
- **Digital Business Profile (DBP)** — Multi-screen public website pattern
- **Snowflake Cortex Search** — Semantic search (future phase)

## Team Assignment (from notes)

- Padma: LinkedIn OAuth
- Riju: Workspace setup, cloud agents, component library, FUNDA DBP (events, about us, startups, offers)
- Arka: Admin app
