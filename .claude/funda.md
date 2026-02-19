# Funda — Customer Project

> Notion: [Funda](https://www.notion.so/26f1d464528f807f9782d5bffc5401a5) (under Customer Success Projects)
> Subpages: [FUNDA notes](https://www.notion.so/21f1d464528f804da24df85d82f35d9d)

- **Workspace ID**: `sEt2qxPydL`
- **Agent Server**: `agt.remixlabs.com`
- **Website**: [funda.club](https://www.funda.club/)
- **Domain**: WhatsApp community of ~500–700 Indian founders for networking and support

---

## Context

- Not a revenue customer — community, not traditional sales cycle, not chosen ICP
- Current website (funda.club) is informational only: about us, events, startup directory, support/offers, blog (1 post), join form — no login, WhatsApp is where everything happens
- Entirely built by Remix Customer Success team (not the customer)
- Phased delivery — measure adoption and feedback before expanding scope

---

## Architecture: Three Apps

### 1. Website = FUNDA Digital Business Profile (DBP)

Multi-screen web app published on remix.app, responsive design (mobile + laptop).

**Screens:**

| Screen           | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| Home page        | Landing                                                                     |
| About Us         | Static content                                                              |
| Events listing   | Vertical list; card: date, title, description, image                        |
| Event detail     | Full-screen component: title, location, date/time, picture, HTML text/links |
| Event register   | Registration form (checks capacity)                                         |
| Startup listing  | Search by tags, pagination; card: logo, name, description, URL, tags        |
| Support / Offers | Same container as startups; card: logo, name, description, URL, tags        |
| Blog             | Hosted off Remix platform                                                   |
| Join             | Web-to-lead form                                                            |

### 2. Admin App (Web)

Common nav layout with list → detail → add/edit/delete for each entity. Chrome extension for LinkedIn clipping.

**Screens:** Login (LinkedIn OAuth), Member list, Request-to-join list, Startup list, Event list, Support/Offer list

**Key flow:** Admin sees "requests to join" → clicks one → detail view with LinkedIn URL → opens LinkedIn profile → clips profile info via Chrome extension

### 3. Member App (Web)

**Screens:** Login (LinkedIn OAuth), Member directory (search), LinkedIn profile navigation, Company info, Edit own profile

---

## First Release Scope: Member Directory

Simplified first release — entirely delivered by Remix CS, no customer customization.

- **Surface**: Web app on remix.app, responsive design
- **Auth**: Google, Office365, Apple
- **Profile fields**: Name, Title, Company, LinkedIn, Mobile (+ show to members toggle), Email (+ show to members toggle), Upload picture, "Reach out to me for" tags
- **Member listing**: Search by name, email, company, tag → directory listing with contact options

---

## Services Architecture

All services are generic CRUD (get, save, delete, search) except event registration (capacity check).

| Service Domain    | Notes                                          |
|-------------------|------------------------------------------------|
| Events mgmt       | Generic CRUD + specialized register (capacity) |
| Startup directory | Generic CRUD                                   |
| Support / Offers  | Generic CRUD                                   |
| Candidate (join)  | Generic CRUD                                   |
| Member mgmt       | Generic CRUD                                   |

---

## Component & Card Strategy

- Cards for startups, members, events, partners — either AI-generated (Claude Desktop → prompt → web component → library) or built in Studio
- Goal: make cards and container layouts AI-configurable (experimental)
- Forms done via form generation through prompting
- List views and detail views assembled from library or via prompting
- Admins can potentially configure their own cards and contribute back to library

---

## Auth

- **Admin app + Member app**: LinkedIn OAuth
- **First release (member directory)**: Google, Office365, Apple
- LinkedIn OAuth setup was assigned to Padma

---

## Delivery Notes

- **Roles**: Remix CS assembles and deploys (customer can't self-serve yet — tools not user-friendly enough)
- **FUNDA admin**: Day-to-day administration + potential card configuration/contribution
- **Reusable patterns**: Snowflake Cortex semantic search, general person+company data model (underpins CRM/directory/sales use cases), modular LinkedIn clipper configs
