# Asset / Component Search — Workflow & Reference

How Claude can answer "find me the best asset/component for X" by querying the Remix Agent Server.

> Last synced: **2026-05-06**

---

## What "assets" means

In Remix, **assets = reusable building blocks** published into a workspace library. They include:

- **Card** — visual card layouts
- **Component** — UI + logic components
- **Action** — utility tiles / actions (e.g. Wilber's "List Files", "Read local file")
- **Function** — pure logic functions (e.g. "Interpolate string template")
- **WebComp** — web components (e.g. `rmx-tui-grid`, `rmx-tui-chart`)
- **screen** — full screens
- **cloud_agent** / **widget_agent** / **agent** — deployed agents
- **Data** / **Wasm** / **code** — other published artifacts

Anyone in the team can publish to their personal library; Arvind / John / Wilber publish most of the shared components in `remix_labs`, `remix-libraries`, and `remix-dbp`.

---

## How the search works (mechanism)

Each library workspace exposes an internal **undocumented but stable** agent:

```
GET https://agt.remixlabs.com/v0/run-agent/{library_ws}/_rmx_search/get_assets_light
    ?inputEscEnabled=true&outputEscEnabled=true
Authorization: Bearer <JWT>
Content-Type: application/json
Body: {}
```

Returns `{ "assets": [ ... ] }` — full catalog. Each asset entry contains:

| Field                                                                 | Meaning                                                                                                                             |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `name`                                                                | Asset display name                                                                                                                  |
| `displayType`                                                         | Component / Card / Action / Function / WebComp / cloud_agent / screen / widget_agent / Wasm / Data / code / agent                   |
| `collection`                                                          | UI grouping (e.g. UI kit, snowflake, ai, util, data visualiser)                                                                     |
| `assetType`                                                           | e.g. `L2`                                                                                                                           |
| `metadata.description`                                                | Free-text description (often empty)                                                                                                 |
| `metadata.instructions`                                               | How to use it (often empty)                                                                                                         |
| `metadata.tags[]`                                                     | Searchable tags                                                                                                                     |
| `bdSignSet[]`                                                         | Bindings — each `{name, direction, type}`. **`direction: "True"` = InParam, `"False"` = OutParam** (counter-intuitive but verified) |
| `_rmx_id._rmx_value`                                                  | Asset ID — used in share URLs                                                                                                       |
| `clip`                                                                | Base64-encoded Mix AST (the copy-paste payload)                                                                                     |
| `preview`                                                             | Base64+zlib-deflated DOM tree of the visual preview (decode below)                                                                  |
| `_rmx_created_by` / `_rmx_last_modified_by` / `_rmx_last_modified_at` | **Often empty** for older assets — Studio reads author from a separate "source info" record                                         |

### Preview decoding

The `preview` field is **base64 → raw zlib deflate (`wbits=-15`) → JSON**. Two top-level shapes:

```python
import base64, zlib, json
parsed = json.loads(zlib.decompress(base64.b64decode(asset['preview'] + '=='), -15))
# parsed['type'] is "dom" (visual) or "json" (data/function)
```

For `type: "dom"`, walk `parsed['sg']`:

- `_rmx_type` — element kind: `dom_group`, `dom_text`, `dom_icon`, `dom_horizontalContainer`, `dom_verticalContainer`, `dom_button`, or web-component tags like `rmx-tui-grid`
- `display` — human label
- `props.style._rmx_global[]` — class names. Tailwind for `remix_labs`/`remix-libraries`; **"Default class set" naming** for `remix-dbp` (e.g. `"Default - Text"`, `"Heading or Title"`,
  `"White / Inverse Text Color"`)
- `props.style.color` / `props.style.backgroundColor` — actual hex codes (e.g. `#0F243F`, `#ffffff`) — **present even when class names are abstract**
- `props.value` / `props.text` / `props.placeholder` — text content
- `props.name` (for `dom_icon`) — specific icon (e.g. `chevron-triple-up`)
- `children[]` — nested nodes

### Share-link URL (deterministic)

```
https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/{library_ws}/{_rmx_id._rmx_value}
```

User opens this URL → "Copy to Clipboard" button → paste into a Studio screen → the asset drops in.

---

## Known library workspaces

| Workspace ID               | Asset count | Notes                                                                         |
|----------------------------|-------------|-------------------------------------------------------------------------------|
| `remix_labs`               | ~355        | Primary Remix-built library; Tailwind classes                                 |
| `remix-libraries`          | ~240        | "Remix advanced" — large parsing/conversion components                        |
| `remix-dbp`                | ~148        | DBP components; "Default class set" naming; "data visualiser" collection here |
| `com_remixlabs_john`       | ~112        | John's personal library                                                       |
| `com_remixlabs_wilber`     | ~110        | Wilber's library — many Action tiles                                          |
| `iaEj4QYboi`               | ~26         | Lumber workspace library                                                      |
| `com_remixlabs_padmanabha` | ~6          | User's own library                                                            |

Default search target: **`remix_labs`** unless user says otherwise.

---

## displayType breakdown (remix_labs, sample)

```
Component: 209  Card: 56  cloud_agent: 30  Function: 15
screen: 13      Action: 10  widget_agent: 8  Data: 5
Wasm: 4         WebComp: 2  code: 2          agent: 1
```

## Collections seen across libraries

```
UI kit, ai, duckdb, oauth, salesforce, sign in, snowflake,
stats & charts, util, webcomps, zendesk, examples, build-tools,
remix advanced, remix basic, remix draft, remix form inputs,
data visualiser
```

---

## Recommendation workflow

When user asks **"find best asset for X"**:

1. Pick library workspace (default `remix_labs`; ask if ambiguous).
2. Fetch `get_assets_light` (cache to `/tmp/<ws>_assets.json` for the session).
3. Filter / rank by `name` + `metadata.tags` + `metadata.description` + `collection` against the user's purpose. Token-overlap + tag-match is usually enough.
4. For top 1–3 candidates, decode `preview` → describe layout, colors, text, icons.
5. Report:

```
**<asset name>** ([displayType], collection: <col>)
<one-line visual description>
- In:  [in-params]
- Out: [out-params]
- Tags: [tags]
🔗 https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/<ws>/<asset_id>
```

---

## Periodic sync prompt (run every 2 weeks)

Paste this into a fresh Claude session. It refreshes the catalog, JWT, and Swagger spec.

```
Sync my Remix asset-component search capability. The reference is in
.claude/asset-component-search.md and .claude/server-apis-agent-swagger.md.

Steps:

1. Ask me for a fresh JWT (the previous one likely expired — JWTs last ~30 days).
   Do NOT save the JWT to any tracked file. Use it only in this session.

2. For each library workspace listed in .claude/asset-component-search.md (remix_labs, remix-libraries, remix-dbp, com_remixlabs_john, com_remixlabs_wilber, com_remixlabs_padmanabha, iaEj4QYboi), call:
   GET /v0/run-agent/{ws}/_rmx_search/get_assets_light?inputEscEnabled=true&outputEscEnabled=true with Bearer auth and {} body. Save each response to /tmp/{ws}_assets.json.

3. For each library, compare against the previously recorded asset counts:
   - Report new total count and delta (e.g. "remix_labs: 355 → 362, +7 new").
   - List notable new asset names (Components, Cards, Actions added since last sync). Include up to 10 most recently created — sort by _rmx_created_at desc.
   - Flag any assets that disappeared.

4. Re-fetch the Swagger spec at https://agt.remixlabs.com/v1/_doc/openapi.json with the same JWT. Diff against the endpoint tables in .claude/server-apis-agent-swagger.md. Report any new / changed / removed endpoints.

5. Check for any NEW library workspaces by listing all workspaces via GET /v0/ws that I have access to and aren't yet in the table.

6. Update .claude/asset-component-search.md (Last synced date + asset counts + any new workspaces) and .claude/server-apis-agent-swagger.md (Last fetched date + any spec changes). Use Edit tool, not Bash.

7. Summarize what changed in 5–10 bullets.
```

---

## Why bi-weekly cadence

- **Components get added regularly** (every few days some weeks; weekly is too noisy)
- **JWT rotates monthly** — bi-weekly gives 2 chances to catch it before expiry
- **Spec changes are rare** — monthly check via this same prompt is more than enough
- **Aligns with Remix's release cadence** (monthly minor releases, weekly patches)

If you ever skip a sync, no harm — the saved catalog stays usable, you just won't know about brand-new components until the next sync.
