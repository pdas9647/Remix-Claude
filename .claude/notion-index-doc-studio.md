# Notion Index — Documentation: Studio & Platform

Markers: → file.md = content captured | ○ = not yet fetched | ✗ = blank/skipped

---

## Root

- [Remix Documentation](https://www.notion.so/fe31096a438b4f29b103c189bc9a7fb8) — root page
- [Blog and updates](https://www.notion.so/ba9d06c6f70c4d349be54bfcdb8eaed4) — ○ empty hub (release notes: 18 Sep, 25 Sep 2024)

## Getting Started in Remix Studio

- [Getting Started in Remix Studio](https://www.notion.so/1051d464528f8008b584dd33015fd5b3) → glossary.md (9-step tutorial)
    - Step 1: Remix Studio — ○
    - Step 2: A New Project — ○
    - Step 3: A New Screen — ○
    - Step 4: Your First Visual Code — ○
    - Step 5: Edit a Card — ○
    - Step 6: Nodegraph Mechanics — ○
    - Step 7: Meet the Essential Nodes — ○
    - Step 8: Make Something Real — ○

## User Guides and Tutorials

- [User Guides and Tutorials](https://www.notion.so/3e247ff418a94194a8376371117c6bb2) — ○ container
    - [Glossary](https://www.notion.so/3bea488f743e43f38c4b4c2b44d58000) → glossary.md
- [Topics](https://www.notion.so/11b1d464528f801480a8d529a2297527) — container page
    - [Web Components](https://www.notion.so/1141d464528f8056a818e8b84c9c10d6) → platform-topics.md
    - [Wasm Functions](https://www.notion.so/1191d464528f803e8538d6d0783b58a1) → platform-topics.md
    - [Working with files from users](https://www.notion.so/1191d464528f8032b146ed778c80c2e6) → platform-topics.md
    - [Cloud agent pubsub](https://www.notion.so/98f0325a059c4b31a0003fad4c364169) → platform-topics.md
    - [Component overriding](https://www.notion.so/8d23c22ea59c4117951d9c535c8965b7) → platform-topics.md
    - [Permissions management](https://www.notion.so/11b1d464528f80ceb963f650386dde34) → platform-topics.md
        - [Make individual screens available to all users](https://www.notion.so/1111d464528f80b8a505c8668f0c726f) → platform-topics.md
    - [Preview .remix file](https://www.notion.so/10c1d464528f8025926ff046b12d5605) → platform-topics.md
    - [Public, Private and Ephemeral Files](https://www.notion.so/14b1d464528f8074820dcfd0b3e6c72c) → platform-topics.md
    - [Storing secrets and tokens](https://www.notion.so/1961d464528f80d6883fffbf05c34d56) → platform-topics.md
    - [Catalog and Collections](https://www.notion.so/ebc636c1e547468f80944acbbf751d70) → federated-servers.md
        - [Federated Servers](https://www.notion.so/6496884844e74752af5b94d933afb22a) → federated-servers.md
        - [Faceted Search](https://www.notion.so/10d1d464528f80ad8d5bf181ea499394) → federated-servers.md
        - [Example of a catalog page for an Icon Component](https://www.notion.so/f806cab33faf4872810f9fbe04c654d1) → federated-servers.md
        - [Variants](https://www.notion.so/1361d464528f801db88efc7b8a4bc063) → variants.md
        - [Catalog V2](https://www.notion.so/1a51d464528f80db82b8d7d4555f117a) → federated-servers.md
    - [Assemble About Us](https://www.notion.so/b2ad31b63fe8415382958c4e14b08b8e) → remix-dbp-assembly.md

## Remix Studio User Guide

- [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) — ○ top-level
- [Nodes](https://www.notion.so/13b1d464528f802c90b7c60485b36842) — container page
    - [In Params and Out Params](https://www.notion.so/1141d464528f809fbfa6d9983ad074fe) → remix-nodes-basics.md
    - [Cards & Components](https://www.notion.so/10ba63e7a90848f29d5a50f8bd4ac61d) → remix-nodes-basics.md
        - [File Uploading](https://www.notion.so/12e1d464528f8088900dcf2f4154cc24) → remix-nodes-basics.md
    - [Data objects, values and transforms](https://www.notion.so/1051d464528f8022a535d4ab9342d1b0) → remix-nodes-basics.md
    - [Queries](https://www.notion.so/1051d464528f80dcb464fae23efefcf2) → remix-nodes-queries.md
        - [Query syntax](https://www.notion.so/205ca411bf684f5593c3469e39de2af8) → remix-nodes-queries.md
    - [Decisions & State Machines](https://www.notion.so/1051d464528f80a48689d2743814b1af) → remix-nodes-actions.md (stub)
    - [Functions & Code Modules](https://www.notion.so/8c02beb038ec40fea16762bbd8aabfe8) → remix-nodes-actions.md (stub)
    - [Actions](https://www.notion.so/1051d464528f80c9b498daa44df9b685) → remix-nodes-actions.md
        - [Agent Connect tiles](https://www.notion.so/12f1d464528f80a4a372e439a6d074f4) → remix-nodes-actions.md
        - [Api](https://www.notion.so/1571d464528f80e9a9fee7fbbf80a0c5) → remix-nodes-actions.md
        - [Database Actions: Save and Delete](https://www.notion.so/1151d464528f8047b20fd8809e2bb766) → remix-nodes-actions.md
        - [Annotations](https://www.notion.so/1051d464528f8018a17fd69389243631) — ✗ blank
- Studio Server
    - [Auth Integrations](https://www.notion.so/11f1d464528f8070b010f945a0b0e7af) → server-apis-auth.md
    - [Legacy Platform Server API](https://www.notion.so/1061d464528f80449377e6c07e72b30b) → server-apis-legacy.md
- Cloud Agent Server
    - [Cloud Agent Server](https://www.notion.so/1061d464528f80f18e46dd27895aeda2) → server-apis-cloud.md (architecture: workspace ownership, RBAC, auth, usage examples)
    - [Cloud Agent Server API](https://www.notion.so/1061d464528f8022aa1cec129096c82b) → server-apis-cloud.md
    - [Cloud Agent Server v1 API](https://www.notion.so/1bd1d464528f80cb97cde448c9e2a944) → server-apis-cloud.md
- [Self-hosted Remix Server](https://www.notion.so/22a1d464528f8019be54edf3f3b1f576) → remix-deploy.md
- [Remix Database](https://www.notion.so/1061d464528f80248ae6cfe395ee6a9c) — ✗ skipped
- [Working with State](https://www.notion.so/1431d464528f8060964ecdcbfba99fd7) → remix-nodes-basics.md
- [Enable anonymous access](https://www.notion.so/14c1d464528f8079b1bcd79c5fdbf4c6) → platform-topics.md
- [Mixed Auth](https://www.notion.so/1741d464528f808cbb5ff371d571c437) → platform-builder-runtime.md
    - [Mixed auth test](https://www.notion.so/1b91d464528f802ebf9efe3417f1ea6f) — ○ no stable content
- [Macros](https://www.notion.so/2161d464528f801f8ddaf479d816d99f) → platform-builder-macros.md
- [External Actions](https://www.notion.so/2161d464528f8014ac9ce7eda3a134ab) → platform-builder-runtime.md
- [Window management Client Actions](https://www.notion.so/3111d464528f8076b873d8ca75c3b016) → platform-builder-runtime.md (platform vs user events: builder_open, runtime_open_screen, location-changed, open-location)
- [iOS Widget & Shortcut Agents](https://www.notion.so/2161d464528f803f867fe9bc1d4b61a6) → remix-deploy.md
- [Edit / Run / Configure modes](https://www.notion.so/2251d464528f808da28df02fbd305915) → platform-builder-macros.md
- [Building with Remote Data Sources](https://www.notion.so/22d1d464528f80278259e3f6ee68dcd1) → remix-nodes-actions.md (Service Agent "Edit Cloud Server" Navbar button; Project view shortcut)

## Deploying Remix Applications

- [Deploying Remix applications](https://www.notion.so/11b1d464528f8030a107dc7f5e8d27cc) → remix-deploy.md
    - [Running .remix files](https://www.notion.so/1971d464528f80e5bed2ec2a380d331f) → remix-deploy.md
    - [Deploying a flow to the Remix mobile app](https://www.notion.so/1051d464528f80b7b98dd77ab6c388bf) — ✗ blank
    - [Creating iOS and Android widgets in Remix](https://www.notion.so/1191d464528f8025a11be59d14e12ffd) → remix-deploy.md

## Remix Tools

- [Remix Tools](https://www.notion.so/13b1d464528f80d593a7f1de1e0d6d43) → remix-tools.md
    - [Cloud Workspace Tool](https://www.notion.so/12c1d464528f809b9f58e6a707b93fe1) → remix-tools.md
    - [Repository Tool](https://www.notion.so/2e91d464528f800985eee93ec9fd5842) → remix-tools.md

## Additional Documentation Pages

- [Mobile/Desktop .remix file syncing](https://www.notion.so/25d1d464528f806b8c82e873d275aab7) → remix-infra-setup.md
- [Mobile/Desktop deeplinks](https://www.notion.so/27a1d464528f803db94ac980a0bd84eb) → remix-infra-setup.md
- [Synced preferences](https://www.notion.so/2871d464528f80ae96dfeff9a738ca67) → remix-infra-setup.md
- [iOS widget debugging/testing](https://www.notion.so/2091d464528f806bb14ffa0bb037851b) → remix-deploy.md