# Platform Topics — Runtime & Security

> Parent: Remix Documentation > User Guides and Tutorials > Topics
> Source pages linked per section below.
> See also: [platform-topics.md](./platform-topics.md) — Extension Points: Web Components, Wasm Functions, FFI Command Execution

---

## Working with Files from Users

> Source: [Working with files from users](https://www.notion.so/1191d464528f8032b146ed778c80c2e6)

Two-stage process to handle user-provided files:

### Stage 1 — Receive the File (→ Local File type)

Use two special card elements:

- **Drag-and-drop handler** — main component; provides `Local File` output from drag-and-drop
- **File input button** — opens OS file explorer (input type="file")

Both produce a `Local File` type value in Remix.

### Stage 2 — Upload the Local File

| Method                       | How                                                                                  | When to Use                    |
|------------------------------|--------------------------------------------------------------------------------------|--------------------------------|
| **API node**                 | Toggle "upload directly from client" in API node UI; set body to expect `Local File` | Third-party storage (S3, etc.) |
| **Upload File action**       | Special action node set to expect `Local File`                                       | Remix Files area               |
| **Inject Local File action** | Returns file content as `String`                                                     | Small text files (CSV, etc.)   |

**Note:** For performance, don't include large files in screen state — upload directly from client.

### Display File Details

Use a **LocalFile Node** to show file metadata (name, size, etc.) to the user.

---

## Cloud Agent Pubsub

> Source: [Cloud agent pubsub](https://www.notion.so/98f0325a059c4b31a0003fad4c364169)

Apps can subscribe to **topics published from cloud agents** via 3 new cloud topic options. This enables real-time push from server to client app. Re-fetch Notion page for the video walkthrough.

---

## Component Overriding

> Source: [Component overriding](https://www.notion.so/8d23c22ea59c4117951d9c535c8965b7)

Allows parameterizing components within a screen — swapping out an inner component with a configurable alternative.

### Constraints

- Components subject to overriding **cannot have Event type in/out bindings**

### Setup

1. In the component footer menu: mark the inner node as **"overridable"**
2. In the replacement component footer menu: mark it as **"callable"**
3. Connect the screen to wire the callable component into the overridable slot

Re-fetch Notion page for video walkthrough.

---

## Permissions Management

> Source: [Permissions management](https://www.notion.so/11b1d464528f80ceb963f650386dde34)
> Sub-page: [Make individual screens available to all users](https://www.notion.so/1111d464528f80b8a505c8668f0c726f)

### Screen-Level Anonymous Access

> See also: [Enable anonymous access](https://www.notion.so/14c1d464528f8079b1bcd79c5fdbf4c6)

By default, login is required to run any screen. You can now **set per-screen** whether anonymous users can access it.

- Enables apps with mixed public/protected flows
- Anonymous users accessing restricted screens are prompted to log in
- Set anonymous via the **footer menu** on the screen node; anonymous screens show a **green marker**

**Transition from anonymous to logged-in:**

- Uses a screen called `_rmx_auth` — a default implementation is provided by the builder
- You can create a custom `_rmx_auth` screen (example in catalog — TBD)
- Must implement the `remix/user-signin` **Client Action** (example in source of `https://remixlabs.com/run.html`)

**Note:** Builder emulation of anonymous access is incomplete (Amp permissions model differs from .remix files) — test with `https://remixlabs.com/run.html` instead

---

## Public, Private and Ephemeral Files

> Source: [Public, Private and Ephemeral Files](https://www.notion.so/14b1d464528f8074820dcfd0b3e6c72c)
> See also: stdlib module (Cloud Agent Server docs)

Three file privacy levels based on path prefix:

| Type          | Path Prefix                              | URL Expiry                     | Notes                       |
|---------------|------------------------------------------|--------------------------------|-----------------------------|
| **Private**   | anything not `/public/` or `/ephemeral/` | Signed, expires                | Re-generated on each access |
| **Ephemeral** | `/ephemeral/`                            | Signed, expires + auto-deleted | For temporary transfers     |
| **Public**    | `/public/`                               | Never expires                  | Safe to use anywhere        |

**Key rules:**

- Signed URLs for private/ephemeral files: new URL generated on every access from an agent → **do not store these URLs** (won't work after expiry)
- Secrets are environment-specific (not shared across workspaces, servers, or devices)

**Use cases:**

- Private: confidential PDFs, .remix files with API secrets, images not to be web-indexed
- Ephemeral: temporary file sharing between Desktop and Mobile
- Public: freely accessible assets

---

## Storing Secrets and Tokens

> Source: [Storing secrets and tokens](https://www.notion.so/1961d464528f80d6883fffbf05c34d56)

Two secure storage types, managed by the stdlib `secrets` module:

| Type        | Created By            | How to Access                                  |
|-------------|-----------------------|------------------------------------------------|
| **Secrets** | User applications     | `secrets.set(_, "key", value)` / `secrets.get` |
| **Tokens**  | 3rd-party OAuth flows | `secrets.getToken("plugin_name")`              |

Both are **tied to the specific user** and encrypted with a platform-managed key.

**Storage by runtime:**

- Platform server / agent server: encrypted in DB, keys stored separately
- Client-side (browser): both in local storage → use with caution

### Common Patterns

**Single-user API key:** Build a screen to enter the value → `secrets.set` → retrieve later with `secrets.get` (or via a dedicated agent on agent servers).

**Single-user OAuth token:**

1. Configure auth plugin on platform server (e.g. `auth.remixlabs.com`)
2. Redirect user through sign-in flow → token captured
3. Retrieve: `secrets.getToken("salesforce")`

**OAuth token in cloud agent or browser runtime:**

- Run auth flow through platform server → get token via endpoint:
  `GET https://auth.remixlabs.com/a/x/auth/{plugin_name}/originalToken`
- Authenticated with normal Remix token; platform server handles refresh

**Shared secrets** (no great built-in solution):

1. Store in DB with normal access control
2. Use an anonymous link to an agent that uses the secret
3. Explicit sharing scheme (pubsub, etc.)

**Important:** Don't store secrets in builder assets — they're saved as plain DB records and copied across projects.

**See also (referenced pages):**

- [Secrets stdlib module docs](https://www.notion.so/1061d464528f8147ae05db232fdd6db9)
- [Auth plugin configuration](https://www.notion.so/11f1d464528f8070b010f945a0b0e7af)
- [Anonymous link endpoint docs](https://www.notion.so/1061d464528f81d9b074cf5fd60bc600#9eda9a7acf88443793f42ca6d29d441e)
- [originalToken endpoint docs](https://www.notion.so/1061d464528f80449377e6c07e72b30b#f94fcdb0b6a444918ab2887366963181)

---

## Preview .remix File

> Source: [Preview .remix file](https://www.notion.so/10c1d464528f8025926ff046b12d5605)
> Image-only page (screenshot showing preview feature). Re-fetch for visual.
