# Mix Standard Library — Query, Resource & Registry Modules

> Sources:
> - [query](https://www.notion.so/1061d464528f818b9732d6b6f321f187)
> - [resource](https://www.notion.so/d6783f5009fe471a8e626e37b6598a20)
> - [registry](https://www.notion.so/1061d464528f813f84cad16503e0c479)
    > Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

**Covers:** query (AST builder), resource (deprecated), registry (experimental)

---

## `query` — AST Query Builder

> Typically imported as: `import q = query`

### Module Signature

```
module query
  type ast = data

  // Filter comparisons
  def cmp : string -> string -> data -> ast   // op, field, value
  def cmpExpr : string -> ast -> data -> ast
  def eq : string -> data -> ast
  def neq : string -> data -> ast
  def lt : string -> data -> ast
  def gt : string -> data -> ast
  def le : string -> data -> ast
  def ge : string -> data -> ast
  def contains : string -> data -> ast
  def like : string -> data -> ast
  def featuresOne : string -> data -> ast
  def featuresAll : string -> data -> ast

  // Filter logic
  def nullFilter : ast
  def token : string -> ast
  def exists : string -> ast
  def afterWatermark : string -> ast
  def afterRow : number -> ast
  def not_ : ast -> ast
  def and : data -> ast          // array(ast) -> ast
  def or : data -> ast           // array(ast) -> ast
  def invoke : string -> data -> ast   // string -> array(ast) -> ast

  // Projection
  def object : data -> ast       // array(ast) -> ast
  def cloneMembers : ast
  def cloneMembersWithExpansion : ast
  def assignMember : string -> ast -> ast
  def removeMember : string -> ast
  def objectMember : ast -> string -> ast
  def followRef : ast -> ast

  // Scalars/lists for typed comparisons
  def scalarFallback : data -> ast
  def scalarIndexed : data -> ast
  def listFallback : data -> ast
  def listIndexed : data -> ast

  // Aggregation
  def count : ast
  def group : string -> ast

  // Map constructors
  def filterOfMap : data -> ast
  def filterOfMapFallback : data -> ast
  def projectionOfMap : data -> ast
module end
```

### Filter AST Nodes

- `eq("field", value)` / `neq` / `lt` / `gt` / `le` / `ge` — field comparisons (field name as string)
- `exists("field")` — record has this field
- `afterWatermark(watermark)` — record is after `_rmx_rev` watermark
- `not_(ast)`, `and([ast1, ...])`, `or([ast1, ...])` — boolean composition
- `nullFilter` — matches everything (useful as identity in AND/OR)
- `featuresOne(field, list)` / `featuresAll(field, list)` — list should be `listIndexed` or `listFallback`

Example: `.field == "foo"` → `q.eq("field", "foo")`

### Projection AST Nodes

- `objectMember(null, "fieldname")` — access field from current record; nest for deep access
- `followRef(valueAst)` — follow db reference to full record
- `object([...])` — create new object from members:
    - `cloneMembers` — copy all current fields
    - `cloneMembersWithExpansion` — copy + expand all refs recursively
    - `assignMember("key", valueAst)` — set field
    - `removeMember("key")` — remove field
- `scalarIndexed(v)` / `scalarFallback(v)` — typed scalar values
- `listIndexed(a)` / `listFallback(a)` — typed lists
- `invoke("funcName", [args])` — db engine function (e.g. `invoke("count", [])`)

Example: projection `{ cde: { *, -bar } }`:

```
q.object([q.assignMember("cde", q.object([q.cloneMembers, q.removeMember("bar")]))])
```

### Map Constructors

- `q.filterOfMap({"a": "A", "b": true})` → `q.and([q.eq("a", "A"), q.eq("b", true)])` — nested maps flatten to dot-notation
- `q.filterOfMapFallback({"a": "A", "b": 42})` → uses `scalarFallback` for non-indexed types
- `q.projectionOfMap({"a": 1, "b": 1})` → extracts just the keys as field projections (values ignored)

### Using ASTs

Three ways to use AST nodes:

1. Inside `db.filter(r -> db.eval(q.afterWatermark(wm)))` — filter AST at right place
2. Inside `db.map(r -> db.eval(q.object([...])))` — projection AST
3. Directly via `db.process(ast)` or `db.processPipeline([ast1, ast2])` in the pipeline

A full pipeline is an **array** of AST nodes: `part1 | part2 | part3` → `[ast1, ast2, ast3]`.

---

## `resource` — Static Resource Access *(Soft-Deprecated)*

> Source: [resource](https://www.notion.so/d6783f5009fe471a8e626e37b6598a20)
> **Deprecated — migrate to [`file`](./mix-lang-io.md) instead.**

```
module resource
  def url : option(string) -> string -> result(url, string)
  def bundleUrl : option(string) -> string -> string -> result(url, string)
  def get : option(string) -> string -> result(binary, string)
module end
```

First arg: optional DB name (`null` = current DB). Second arg: resource name. `bundleUrl` takes path inside bundle as third arg.

**Storage:** Resources stored as `_rmx_type == "resource"` records in DB during development. At compile time, a `resource.json` snapshot is created. At runtime, only `resource.json` is read. URLs are
environment-specific: `https://` on web, `file://` in widgets.

---

## `registry` — Session-Scoped Value Store *(Experimental)*

> Source: [registry](https://www.notion.so/1061d464528f813f84cad16503e0c479)

```
module registry
  type key
  def add : any -> key
  def addOnce : any -> key        // since PR#1012
  def find : key -> option(any)
module end
```

Stores arbitrary **case values** and allows fast lookup by key. The key encodes the case type name, enabling retrieval without knowing the concrete case type at lookup time.

**Pattern:** Wrap the value in a case type before adding; unwrap at lookup via pattern matching.

```
case myFn(string -> string)
def k = registry.add(myFn(string.uppercase))
// later:
registry.find(k) |> type(any) |> (some(myFn(f)) -> f | _ -> fail("not found"))
```

**`addOnce`** — stores only on first call (subsequent calls are no-ops for the same invocation site). Use for module-level singletons like sequence generators:

```
private def key = registry.addOnce(genstored(gentype(stream.enumUnbound())))
```