# Collaboration & Assembly — Steel Thread

> Source: [Collaboration & Assembly](https://www.notion.so/44d11988dfb24d0ea32fb8871637e72a)

---

## Overview

End-to-end workflow showcasing the three persona pipeline from asset creation to end-user personalisation.

**tl;dr:**

1. Developer builds asset in Remix Studio with config points (including consumer-grade configurator form) and updates the asset library
2. Publishes a catalog page with configuration (Tooling)
3. Assembler configures and inserts it in Remix Studio, publishes flow as .remix to a workspace (Tooling)
4. Consumer gets a configurator landing page (URL to a Web App) to personalise and get a public clip link

## Three Personas

### A: Asset Developers (Remix team + future dev partners) — Remix Studio

- Build components with configuration points (e.g. About Us: logo, banner, name, title, email, phone, video, website link)
- Config points split by persona:
    - **Assembler-configured:** logo, banner, video, website link
    - **End-user personalisable (edge):** name, title, email, phone, video
- Component must also be configured with `workspace, db, user` for cloud data enablement
- Update component to cloud asset library
- Build & publish a configurator form/screen with shortlisted (Const-based) config points for end-user persona
- Publish a catalog page where the component can be configured (all config points + pre-associated end-user configurator form)

### B: Assemblers (team, service partners, customer citizen developers) — Remix Studio

- Search catalog for templates, select and configure component
- Copy/paste into assembly surface (Studio)
- Assemble flow in Studio: use screen templates, insert configured components, build the flow
- Use tooling to package, upload, and get a configurator "app" link (constructs full "left-right" configurator link, potentially a landing page)

### C: Consumers (line of business end users) — Configurator Apps

- Go to landing page, select template or land in Configurator
- Personalise flow/clip/app via configurator panel (e.g. name, title, phone, email, optional video)
- Preview, get a link & QR to distribute
- Redirect to info about mobile app, register to see clip activity & stats

## Open Questions

- How/where/by whom is workspace configuration done so that stats, contact forms, push notifications etc. are set up for a particular customer?
