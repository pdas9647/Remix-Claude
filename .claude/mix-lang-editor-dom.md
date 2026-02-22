# Mix Standard Library — Editor: DOM & Node Introspection

> Sources:
> - [dom](https://www.notion.so/1061d464528f81e4b0b6f274bd0e53be)
> - [dom-sync-compress](https://www.notion.so/1061d464528f8112bf01fd7c1d32e2f6)
> - [editor-types](https://www.notion.so/1061d464528f815a9b59eca2599c6b7e)
> - [editor-node](https://www.notion.so/1061d464528f8169ad98d80c2c816f5f)
> - [editor-screen](https://www.notion.so/1061d464528f81dea0d3d7ef8eb5a157)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-editor-reflect.md](./mix-lang-editor-reflect.md) | [mix-lang-editor-slate.md](./mix-lang-editor-slate.md) | [mix-lang-editor-runtime.md](./mix-lang-editor-runtime.md)

---

## `dom` / `domUtil` — DOM Values & Manipulation

> Source: [dom](https://www.notion.so/1061d464528f81e4b0b6f274bd0e53be) (since PR 781/786)

### DOM Literal Syntax

```
{ _rmx_type: "dom_group",
  props: { style: { color: "red" }, events: { onClick: handler }, key: val },
  children: [ ... ]
}:dom
```

- `_rmx_type` must be a literal string.
- `style` and `events` inside `props` should be literal objects, or `domProps` values from `dom.propertyMap`.
- Children can be a literal array or a factored-out variable.
- **Do NOT transmit `dom` values between sessions** — symbol table is session-local.

### `sync cell`

```
sync cell x = <dom expression>
```

Special cell containing DOM, transmitted to client via sync protocol. When referenced from another cell, produces a **cell reference** rather than inlining the DOM:

```
cell y = { _rmx_type: "dom_group", children: [x] }:dom
// y contains a cell reference to x, not x's DOM directly
```

`domUtil.restore(y)` → `{_rmx_type:"dom_group", children:[{_rmx_type:"cell", index:<n>}]}`

### Property Lists

```
dom.propertyEmpty : domProps
dom.propertyMap : map(data) -> domProps      // creates domProps (has some cost vs literals)
dom.propertyAdd : domProps -> domProps -> domProps   // p2 takes precedence on conflicts
dom.restoreProperties : domProps -> map(data)
```

### Create Dynamically

```
dom.empty : dom                              // missing/null DOM
dom.createNode : string -> map(data) -> map(data) -> map(data) -> map(data) -> map(data) -> array(dom) -> dom
  // (rmxType, staticProps, dynProps, staticStyle, dynStyle, events, children)
  // static = put into symbol table; dynamic = volatile; events always volatile
dom.createNodeFromProps : string -> domProps -> domProps -> domProps -> domProps -> domProps -> array(dom) -> dom
dom.createCellRef : number -> option(string) -> dom
dom.createTree : data -> dom                 // slowest; converts whole nested object
```

### Access

```
dom.children : dom -> array(dom)
dom.rmxType : dom -> string
dom.props / dom.style / dom.events : dom -> domProps
```

### Serialize (session-local symbol references)

```
dom.toStream / dom.toArray : dom -> number* / array(number)
dom.fromStream / dom.fromArray : number* / array(number) -> dom
```

### Helpers

```
dom.isDom : data -> bool
dom.toDom : data -> option(dom)      // typed as dom if true
dom.isEmpty / dom.isNode / dom.isCellRef : data -> bool
```

### Symbol Table

Constants and accessors for low-level DOM serialisation format:

- Control symbols: `emptyControlSymbol`, `firstChildControlSymbol`, `nextChildControlSymbol`, `endChildControlSymbol`, `dynamicControlSymbol`, `singleCellRefControlSymbol`
- `numControlSymbols`=16, `numIntegerLiterals`=2001 (0–2000), `symbolOffset`=2048, `integerOffset`=16
- `dom.symbolNumber(n)` — symbol for integer n; `dom.symbol(d)` — symbol for any data; `dom.lookup(sym)` — value for symbol

### `domUtil`

```
domUtil.deref : dom -> dom                             // follow cell references
domUtil.restore : dom -> data                          // dom → conventional object (recursive)
domUtil.restoreIf : bool -> dom -> data                // conditional restore
domUtil.restoreArrayIf : bool -> array(dom) -> array(data)
```

---

## DOM Synchronization Protocol (`sync cell`)

> Source: [dom-sync-compress](https://www.notion.so/1061d464528f8112bf01fd7c1d32e2f6)

### `sync cell` keyword

Marks a `dom`-typed cell for automatic synchronization to the client:

```
sync cell p = {_rmx_type:"foo"}:dom    // value must be dom type
```

- When content changes, the cell is retransmitted automatically
- When a `sync cell` is referenced in another cell, it's included **by reference** (as `{_rmx_type:"cell", index:N}`), not by value — only the referenced cell is re-sent on change
- Requires `syncProtocol: 1` in `msg_session_create` to enable

### Enabling sync in session creation

```json
{
  "_rmx_type": "msg_session_create",
  "syncProtocol": 1,
  ...
}
```

### Protocol messages

- `$sync = [sheet, index1, value1, index2, value2, ...]` — transmits new cell contents as arrays of symbols for a named spreadsheet
- `$syncSwitch = "view-N"` — switches current spreadsheet (top of viewstack); bare cell refs use this as target sheet
- `$syncDrop = "view-N"` — drops a closed spreadsheet (when popping views)
- `$symbols = [v1, v2, ...]` — extends the symbol table; values can be: scalars, `{literal:[...]}` wrapped arrays/maps, or key/value lists (alternating key/val symbol pairs for static props)

### Symbol encoding

Symbols ≥ 2048 → index into symbol table (offset 2048). Range 16–2016 → integer literals (16=0, 17=1, ...). Range 0–15 → control symbols:

- `0` = empty DOM node (`dom.empty`)
- `1` / `2` / `3` = firstChild / nextChild / endChildren markers
- `4` = dynamic properties/styles/events follow
- `5` = cell reference follows (cellIndex, optSheet)

---

## `editorTypes` — Binding Direction & Visual Types

> Source: [editor-types](https://www.notion.so/1061d464528f815a9b59eca2599c6b7e) (since PR 1020)

```
module editorTypes
  case input
  case output
  type direction = input | output
  def revDirection : direction -> direction

  case vty_bool | vty_card | vty_color | vty_data | vty_date | vty_email
     | vty_event | vty_icon | vty_image | vty_list(data) | vty_location
     | vty_number | vty_object | vty_ref | vty_string | vty_token | vty_url
  type visualType = ...  // union of above cases

  def isEvent : visualType -> bool
  def isList : visualType -> bool
  def exportVisualType : visualType -> data
  def importVisualType : data -> result(string, visualType)
module end
```

- `direction` — used in `editorNode.binding` to describe whether a bindpoint is input or output.
- `visualType` — the editor-visible type of a bindpoint (e.g. `vty_string`, `vty_card`, `vty_event`).

---

## `editorNode` — Nodegraph Node Introspection

> Source: [editor-node](https://www.notion.so/1061d464528f8169ad98d80c2c816f5f)

```
module editorNode
  type t        // a loaded, parsed editor node
  type query    // lazy query that loads certain nodes

  type binding = {
    cellName: string, bdName: string, displayName: string,
    direction: editorTypes.direction, realDirection: editorTypes.direction,
    order: number, active: bool, isPayload: bool,
    vty: editorTypes.visualType, defaultValue: data, duplicate: bool
  }

  type connection = {
    outCellName: string, outBdName: string,
    inCellName: string, inBdName: string
  }

  // Query builders
  def byParentRef : string -> query
  def filterByType : string -> query -> query
  def filterByInParams / filterByOutParams : query -> query
  def filterByCellName : string -> query -> query
  def filterByInCellName / filterByOutCellName : connection -> query -> query
  def filterByDisplayName : string -> query -> query

  // Execute query
  def toStream : query -> stream(t)
  def toArray : query -> array(t)
  def first : query -> t

  // Node info
  def getType : t -> string
  def getCellName : t -> string
  def getDisplayName : t -> string

  // Traversal
  def children : t -> query
  def siblings : t -> query

  // Connections & bindings
  def bindings : t -> array(binding)
  def outConnections / inConnections : t -> array(connection)
  def followOutConnection : t -> connection -> [t, binding]
  def followInConnection : t -> connection -> [t, binding]
module end
```

---

## `editorScreen` — Screen-Level Editor Introspection

> Source: [editor-screen](https://www.notion.so/1061d464528f81dea0d3d7ef8eb5a157)

```
module editorScreen
  type t   // a loaded, parsed editor_screen record

  def byRef : string -> option(t)      // look up by permanent ref ID
  def byName : string -> option(t)     // look up by name
  def screens : null -> array(t)       // all regular screens (excludes Symbols/Styles)
  def getSymbols : null -> t           // the Symbols module
  def getStyles : null -> t            // the Styles module
  def all : null -> array(t)           // screens + Symbols + Styles

  def getRef : t -> string
  def getType : t -> _type
  def getName : t -> string
  def children : t -> editorNode.query
module end
```