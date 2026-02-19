# Design Proposal — L1 Catalog Assets

> Source: [Design Proposal - L1 catalog assets](https://www.notion.so/13f1d464528f80f4a2ecceedce5a0051)

---

## Goal

Extend catalog operations (currently L2-only) to L1 (Project Home) level:

- **Publish** a screen/agent/code module to a catalog at L1
- **Search** L1 catalog assets and insert into an app at L1
- **Update** an existing catalog asset at L1
- **Sync** catalog assets at L1
- At **L2**: search nodes of L1 catalog assets (screens only, possibly tagged with 'node repository')
- On **Amp-hosted** catalogs: open and edit screens like normal (even if published from another machine)

## Key Constraints

### Amp-hosted catalog (e.g. remix-beta)

- Need full builder screens (editor_screen4 + editor_node4), not just JSON
- Users must be able to open/edit the screen in the builder
- Potential approach: first L1 visit triggers full materialization (same as paste sequence)

### Cloud-hosted catalog (e.g. agt-dev)

- JSON structure alone could work for screen-level operations
- **Problem:** existing L2 library search expects real nodes (agent queries editor_node4 in the db)
- Storing only JSON means individual nodes inside screens are not searchable
- Changing search to parse JSON is possible but updating nodes deep inside JSON is problematic

## Conclusions

- Copy/paste JSON is convenient for sending screens but must be **unpacked** (pasted) for full search and node-level updates
- Unpacking is non-trivial — builder does complex Elm processing at paste time (dependency checking, symbol insertion, style addition)
- **Open question:** may need a simpler JSON structure for L1 screen publishing, or accept reduced processing compared to current paste behaviour
