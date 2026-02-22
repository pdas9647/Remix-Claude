# Remix Nodes — Actions & Service Agents

> Parent: Remix Documentation > User Guides and Tutorials >
> [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) >
> [Nodes](https://www.notion.so/13b1d464528f802c90b7c60485b36842)

**Covers:** Decisions & State Machines, Functions & Code Modules, Actions (File Register, Agent Connect, Api node, Database Save/Delete), Annotations, Service Agent modules

---

## Decisions & State Machines

> Source: [Decisions & State Machines](https://www.notion.so/1051d464528f80a48689d2743814b1af)
> Note: Page content was truncated on fetch — re-fetch for full detail.

---

## Functions & Code Modules

> Source: [Functions & Code Modules](https://www.notion.so/8c02beb038ec40fea16762bbd8aabfe8)
> Note: Page content was truncated on fetch — re-fetch for full detail.

---

## Actions

> Source: [Actions](https://www.notion.so/1051d464528f80c9b498daa44df9b685)
> Note: Main page content was truncated on fetch. Sub-pages fully captured below.

### File Register

Used as part of a flow to upload files to a remote mixer. Visual implementation of `file.register`.

- Uses the term `location` instead of `app_designator` (unlike the underlying stdlib API)
- **Best practice:** Do registration from within a **service agent** (using Main or Other DB) — Admin (Remote) registrations require builder privileges
- Reference: [`file` stdlib module](https://www.notion.so/file-1061d464528f8136b486db76ee013016#1061d464528f811cb892c2db1b25a6a5)

---

## Agent Connect Tiles

> Source: [Agent Connect tiles](https://www.notion.so/12f1d464528f80a4a372e439a6d074f4)

Agents are modules called with parameters that return values. Three types:

| Type              | Description                                                                              | Setup                                                  |
|-------------------|------------------------------------------------------------------------------------------|--------------------------------------------------------|
| **Service Agent** | Deployed on a cloud server                                                               | Enter Cloud URL → Refresh → select workspace/app/agent |
| **Local Agent**   | Runs on same server as caller (cloud server if from cloud agent; Amp if from Amp screen) | No URL needed                                          |
| **Amp Agent**     | Rarely used; targets a specific Amp server                                               | Provide `ampPrefix` (URL to Amp server)                |

**Refresh button** — updates agent bindings when the agent's signature changes or when pointing to a different agent. In/out-params are mirrored on the action node.

**Client Side toggle** — runs the agent call from the client rather than the VM. Useful when: calling an MCP on `localhost`, or when the caller needs to access local resources.

---

## Api (Action Node)

> Source: [Api](https://www.notion.so/1571d464528f80e9a9fee7fbbf80a0c5)

Performs HTTP requests to servers. Opinionated (most API requests simple; special cases need Mix).

### URL Templates

Use `{...}` syntax in the URL for dynamic segments. *(Section truncated on fetch — re-fetch for full detail.)*

### Token

Turn on the token binding but leave it **unconnected** to use the current user's Remix token.

### Content-Type: multipart/form-data

Set the `Content-Type` header to `multipart/form-data` and pass an object as the body — encoded automatically.

- **VM side**: object values must be `String`
- **Client-side**: can include `Local File` types from an upload widget
- Manual fallback example: `https://remix-dev.remixlabs.com/e/edit/builder-examples/multipart_manual`

### Content-Type: application/x-www-form-urlencoded

Set the `Content-Type` header accordingly and pass an object as the body — encoded automatically. Also handles nested fields.

### Client-Side Operation

Runs the API request from the **client** (browser/app) rather than the VM server. Useful when:

- Calling `localhost` (e.g. local MCP server)
- Uploading local files — client does the upload directly without passing data into VM (significant performance benefit). Use together with
  the [File Upload pattern](https://www.notion.so/12e1d464528f8088900dcf2f4154cc24).

### Params

`outputEscEnabled=true` — see [referenced page](https://www.notion.so/92c2ae9e2068459a9af3178525a8a906) (unfetched).

---

## Database Actions: Save and Delete

> Source: [Database Actions: Save and Delete](https://www.notion.so/1151d464528f8047b20fd8809e2bb766)

### Create and Update

*(Placeholder — "Text to add". No content yet on page.)*

### Delete

Requires: **DB location** + **ref** of the record to delete.

*(Note: A previous version of this page mentioned error bindings added in Oct 2024, but that text was struck through — current status unclear. Re-fetch for latest.)*

---

## Annotations

> Source: [Annotations](https://www.notion.so/1051d464528f8018a17fd69389243631)
> Blank page — no content.

---

## Service Agent Modules

> Source: [Service Agent modules](https://www.notion.so/22d1d464528f80278259e3f6ee68dcd1)
> Parent: Remix Studio User Guide
> See also: [Agent Connect tiles](https://www.notion.so/12f1d464528f80a4a372e439a6d074f4)

Service Agent modules have an additional **"Edit Cloud Server"** button in the Navbar. Clicking it lets you see the details of a Cloud Server that has data on it — all queries, save actions, and
delete actions are then run against that server. This makes building new agents against a remote dataset much easier.

**Important:** Choosing a Cloud Server here does **not** change the generated code — all nodes will still point to the workspace where they are installed (not the preview server).