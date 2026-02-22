# Remix Nodes — Basics (Params, Cards, State)

> Parent: Remix Documentation > User Guides and Tutorials >
> [Remix Studio User Guide](https://www.notion.so/1051d464528f8007a5f5d40969c49427) >
> [Nodes](https://www.notion.so/13b1d464528f802c90b7c60485b36842)

**Covers:** In Params/Out Params, Cards & Components, File Uploading, Data Objects/Values/Transforms, Working with State

---

## In Params and Out Params

> Source: [In Params and Out Params](https://www.notion.so/1141d464528f809fbfa6d9983ad074fe)

### Reordering Bindings

Bindings in in/out-params nodes can be reordered with up/down arrows. After reordering and publishing, **Push, Navigate, and Agent Connect nodes** referencing those params are automatically "patched"
to reflect the updated order when next opened.

### Agent Error Out-Params

Out-params in agents have 2 extra bindings beyond the normal data bindings:

- **error trigger** — invoke this to make the agent return an error
- **error string** — the error message returned

In the calling code:

- The `error next` binding fires
- The `error data` binding holds the error string

Normal data bindings (0 or more) are returned only if the agent ran correctly.

---

## Cards & Components

> Source: [Cards & Components](https://www.notion.so/10ba63e7a90848f29d5a50f8bd4ac61d)

### Cards

**Style Units** — CSS properties that take numerical values now support units beyond `px` (added December 2024): `em`, `rem`, `%`, `vw`, `vh`, etc.

**Text Inputs** — `debounce` property: milliseconds before the value in a text input is sent to the server. Default: `150ms`.

### Components

**Headless Components** — Components that return only data, not card data for display. Mark via: In/Out-Params → "Set as Headless" option. The Card node marked as "Screen" changes format; above in the
stack, the component view shows in/out param bindings in a more useful manner.

**Triggered Components (advanced)** — Similar to screens: no action in/out bindings. A runner can start and stop these components externally.

- Runner workspace: `https://agt.remixlabs.com/ws/com_remixlabs_simon`
- Example: `https://remix-dev.remixlabs.com/e/edit/builder-examples/triggered_comp`

### File Uploading

> Source: [File Uploading](https://www.notion.so/12e1d464528f8088900dcf2f4154cc24)
> See also: page [1571d464528f80e9a9fee7fbbf80a0c5](https://www.notion.so/1571d464528f80e9a9fee7fbbf80a0c5) (unfetched)

**Setup requires two nodes:**

- **Local File Controller** — provides `Local File` output to the app; set on its own node
- **Card with File Input and/or dropzone** — bound to the `default-screen` binding

**Controller properties:**

| Property   | Description                                                                                                       |
|------------|-------------------------------------------------------------------------------------------------------------------|
| `multiple` | Whether one or several files can be selected (applies to drag & drop too)                                         |
| `accept`   | File type filter; see [MDN accept attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/accept) |

**Important:** Switching from multiple to single file → also toggle the action binding.

**Directory drop** (since November 2024) — Files inside a dropped directory are returned with their path as the name.

---

## Data Objects, Values and Transforms

> Source: [Data objects, values and transforms](https://www.notion.so/1051d464528f8022a535d4ab9342d1b0)
> Note: Page content was truncated on fetch — re-fetch for full detail.

### Transform: Join

The **Join** node performs an **inner join** (SQL inner join semantics). It projects the union of fields from the joined data — it does **not** handle field name collisions.

---

## Working with State

> Source: [Working with State](https://www.notion.so/1431d464528f8060964ecdcbfba99fd7)

### Three State Scopes

| Scope            | Description                                          | Use Case                                                                                    |
|------------------|------------------------------------------------------|---------------------------------------------------------------------------------------------|
| **Local State**  | Operates at one level in the component graph         | Component-local data                                                                        |
| **Screen State** | Available throughout the component stack of a screen | Building autonomous components for assembly                                                 |
| **App State**    | Available across screen navigation (global)          | Persistent data across navigation; powerful but hard to debug, potential performance issues |

Get State / Set State nodes work with all three types.

### State Node

A read/write object where you specify fields (and/or enable arbitrary location access).

- **Initial value:** Connect the main in-binding to set initial value
- **Important:** Subsequent changes to the main in-binding are **ignored** — use Set State methods to update
- Only specified fields can be written to via the green navbar menu

### Set State

**Static locations:** Update specified values in state. Available on value out-bindings of actions, in-params, and components.

**Dynamic locations:** The Set State action node enables setting values at dynamically configured fields or paths. A path is an array of field names/indexes, e.g. `["output", "name"]` sets
`.output.name` within the state.

**How to use:**

1. Add "Set State" node: `+ NEW` → `Actions` → `SET STATE`
2. Select target state cell: "Screen State" or a named state node
3. Set or delete values at a top-level "Field" or at a "Path" in the cell

### Get State

**Static locations:** Analogue of Set State but for in-bindings.

**Dynamic locations:** If no field is set, Get State acts as a preview of the current state value.