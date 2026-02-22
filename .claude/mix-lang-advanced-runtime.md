# Mix Language — Advanced Runtime Integration

> Covers links, EscJSON representation, DB query patterns, and embedding viewstacks.
> See also [mix-lang-advanced.md](./mix-lang-advanced.md) for core language patterns (recursion, streams, modules, pattern matching).

---

## Links — Advanced

> Source: [links](https://www.notion.so/1061d464528f8159bc12ea2ae03ecbad)
> Core link/alias syntax: [mix-syntax-core.md § Spreadsheet (Cells)](./mix-syntax-core.md)

### `link(c)` does NOT create a dependency

Using `link(c)` in a cell does not add `c` to the dependency graph. The link captures the cell's
*identity*, not its value — so changes to `c` do not re-trigger the cell holding the link.

```
cell y = contents(link(x))   // reads x but y is NOT updated when x changes
cell z = x                   // z IS updated when x changes
```

### Assigning to link variables

A `link(T)<var>` can be the left-hand side of an assignment inside an action. In `def`s, need
`appstate` param:

```
def set_link : appstate -> link(A)<var> -> A -> data
def set_link(appstate, lnk, x) { lnk = x }
```

### `inCellName(link(c))`

Returns the internal string name of the backing cell. Used to tell the UI which cell to write into
(e.g. camera capture, file upload):

```
cell view = { "_rmx_type": "msg_camera_capture", "name": inCellName(link(picture)) }
```

### `dependentLink` (experimental)

Recomputed link — recreated only when the second function's result changes:

```
dependentLink(creator_fn, dependency_fn)
// creator_fn: _ -> link       (builds the link)
// dependency_fn: _ -> value   (link recreated only when this value changes)

alias zz = dependentLink(_ -> M(z).y, _ -> floor(z % 5))
// link to M(z).y; recreated only when floor(z % 5) changes
```

Useful for controlling when module instances are rebuilt (e.g. only when array length changes, not
on every row value update).

### `mapLinkArray` (experimental)

Maps over an array of rows, applying a link-creating function; returns a link to the resulting
array of cell values:

```
alias view = mapLinkArray((k, _) -> M(rows, k).view, rows)
// view updates when rows changes, but individual links not recreated on value-only changes
// Caution: runtime error if rows array length increases — combine with dependentLink
```

### Limitation: no links between views

Links cannot cross view boundaries. A newly pushed view starts a fresh spreadsheet — you cannot
pass a `link(T)` to it as a push parameter.

---

## JSON / EscJSON Representation

> Source: [json-changes](https://www.notion.so/1061d464528f817aacb5db1a7d575617)

Mix types richer than JSON (case types, db refs, calendar dates, color values) are serialized with
**escape tags**. Non-serializable types: streams, closures, tokens.

### Escape tag format

```
{ "_rmx_type": "{tag}case:ok", "_rmx_value": "hello" }  // ok("hello")
{ "_rmx_type": "{tag}ref",     "_rmx_value": "123" }     // db reference
```

To store the literal string `{tag}...` in `_rmx_type`, prefix with `{str}` (double-escape):

```
{ "_rmx_type": "{str}{tag}case:ok" }   // plain string, not a case value
```

`{str}` only appears in `_rmx_type` fields.

### `string` module JSON functions

```
string.formatJSON(x)           // serialize; fails if x contains non-JSON-native values
string.formatEscJSON(x)        // serialize with escape tags
string.parseJSON(s)            // parse; does NOT interpret escape tags
string.parseJSONResult(s)      // same, returns result(string, data)
string.parseEscJSON(s)         // parse AND interpret escape tags
string.parseEscJSONResult(s)   // same, returns result(string, data)
```

Think twice before enabling EscJSON on untrusted input — escape tags can affect db query behavior.

### Full tag type reference

> Source: [escjson](https://www.notion.so/1061d464528f8110aae4ef61230e4783) (canonical reference)

| Tag                                        | Meaning                   | Notes                               |
|--------------------------------------------|---------------------------|-------------------------------------|
| `{tag}case:red`                            | parameterless case `red`  | no `_rmx_value`                     |
| `{tag}case:meters` + `_rmx_value`          | case with arg `meters(v)` |                                     |
| `{tag}case:db.ref` + `_rmx_value`          | database reference        | canonical form                      |
| `{tag}bin` + `_rmx_value` (base64)         | binary/blob data          |                                     |
| `{tag}blob:image/png` + nested `{tag}bin`  | blob generator            | expanded to saveable db blob record |
| `{tag}undefined`                           | `undefined` value         |                                     |
| `{tag}nan` / `{tag}posinf` / `{tag}neginf` | special float values      |                                     |
| `{tag}int` + `_rmx_value`                  | large integer (bigint)    | input-only; output uses JSON number |
| `{tag}float` + `_rmx_value` (hex FP)       | precise float             | input-only; output uses JSON number |
| `{tag}str` / `{tag}null` / `{tag}bool`     | string / null / bool      | input-only; output uses native JSON |

Note: the json-changes page uses `{tag}ref`; the escjson reference page uses `{tag}case:db.ref`. Treat `{tag}case:db.ref` as canonical.

### DB refs

Old string form `"{ref}123"` is deprecated. Current canonical form:

```
{ "_rmx_type": "{tag}case:db.ref", "_rmx_value": "abcd" }
```

- `db.makeRef("{ref}123")` — convert old-style string to a db ref
- `db.isRef(x)` — test whether x is a db ref
- Db refs behave like strings in string operations (e.g. `string.length` works)

### `_escJSON_` notation for Mix constants

```
def const = _escJSON_ { _rmx_id: { "_rmx_type": "{tag}ref", "_rmx_value": "123" }, name: "foo" }
```

- Value after `_escJSON_` must be a compile-time constant
- `{tag}` in `_rmx_type` fields is interpreted

### Track protocol escape tag behavior

- **Requests**: escape tags OFF by default; enable per parameter:
    - `msg_view_invoke`: `argEscEnabled: true`
    - `msg_view_load` / `msg_view_push`: `argsEscEnabled: true`
    - `msg_view_input`: `valueEscEnabled: true`
    - Exception: `nextActions` in `msg_view_invoke` always has escape tags enabled
- **Responses**: always formatted with escape tags (any Mix type can appear in output)

---

## DB Query Notes

> Source: [examples](https://www.notion.so/1061d464528f815fad1ed849ced729c1)

### `db.all()` vs `db.head()`

- `db.all()` — returns all non-user-deleted records across **all versions**
- `db.head()` — returns only the **latest version** of each non-user-deleted record; only available when the query engine is directly connected to FigDB (i.e. server-side agents, not remote HTTP
  queries)

### `def` redefinition scoping

A second `def` with `=` in the same scope **replaces** the first, and **all prior references** see the new value:

```
def x : data
def y = (arg -> x)   // y closes over the scope, not the current value
def x = 1
def x = 2
y(null)              // returns 2, not 1!
```

This is intentional — global `def`s act like named cells in the dependency graph, not like let-bindings.

---

## Embedding Viewstacks

> Source: [embedding-viewstacks](https://www.notion.so/1061d464528f81a7a3a0e65bf1f509f6)

A secondary viewstack can be created, embedded visually into the main one, and controlled independently.

### Creating a secondary viewstack

```
declare myScreen(param:data)

let vs = viewstack.embed(appView myScreen({p1: ...}))          // returns embeddedView
// or by string name:
let vs = viewstack.embed(viewstack.dynamicAppView("myScreen", {p1: ...}))

viewstack.terminate(vs)    // destroy when done (frees memory)
```

### Reading output (DOM stream)

```
let msgStream = viewstack.cellMessages(vs)
// stream of {cellname: string, cellvalue: data}
// pseudo-cells: $location (navigation events), $nextActions (follow-up actions)

// filter to DOM cell + next actions (use for remote embedded apps too):
let domStream = msgStream
  |> stream.filter(msg -> msg.cellname === "$nextActions"
                          || string.endsWith(msg.cellname, ".view"))

// prepend placeholder so UI doesn't wait:
let effStream = stream.singleton({cellname:"dummy.view", cellvalue: <placeholder>}) ++ domStream
```

### Consuming stream into spreadsheet

```
live cell msgToEmbed = co.consumeWithActions(effStream)
// co.consumeWithActions can handle $nextActions; co.consume cannot

private cell domToEmbed = [ msgToEmbed.cellvalue ]
// connect domToEmbed to a container card tile
```

### Complete action template (embed/replace pattern)

```
var cell lastStack : option(embeddedView) = null
var cell effStream = stream.singleton({cellname:"dummy.view", cellvalue:null})
live cell msgToEmbed = co.consumeWithActions(effStream)

// action to embed a screen:
do {
  switch (lastStack) {
    case some(vs): viewstack.terminate(vs)
    case null:
  }
  let vs = viewstack.embed(appView myScreen(param));
  lastStack = some(vs);
  let msgStream = viewstack.cellMessages(vs);
  let domStream = msgStream
    |> stream.filter(msg -> msg.cellname === "$nextActions"
                            || string.endsWith(msg.cellname, ".view"));
  effStream = stream.singleton({cellname:"dummy.view", cellvalue:null}) ++ domStream
}
```

Input side (action closures): local embedded viewstacks handle action routing automatically. Remote apps require the Track protocol.