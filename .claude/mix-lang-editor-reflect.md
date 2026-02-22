# Mix Standard Library — Editor: Reflect & Slate Components

> Sources:
> - [reflect](https://www.notion.so/1ec2708f02674490843320f09c45ed20)
> - [reflectComponent](https://www.notion.so/1061d464528f817ebfc7d9c8a6e67b84)
> - [slateComponent](https://www.notion.so/1061d464528f81caaa78f020598d6157)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-editor-dom.md](./mix-lang-editor-dom.md) | [mix-lang-editor-slate.md](./mix-lang-editor-slate.md) | [mix-lang-editor-runtime.md](./mix-lang-editor-runtime.md)

---

## `reflect` — Module, Screen & Spreadsheet Introspection

> Source: [reflect](https://www.notion.so/1ec2708f02674490843320f09c45ed20) (since PR 171)

```
module reflect
  type info = {
    name: string, libID: string, arity: number,
    params: array(string), meta: data, space: modSpace
  }
  type unitInfo = { libID: string, requires: array(string) }

  // Physical modules (since PR1213: names are libID-prefixed)
  def modules : null -> array(string)
  def moduleInfo : string -> option(info)    // pass qualified name
  def moduleQuery : null -> stream(info)
  def moduleCheck : A -> option(moduleDefinition)   // PR905: is value a module instance?

  // Screens, agents, components (since PR1201)
  def screens / agents / components : null -> array(string)
  def screenInfo / agentInfo / componentInfo : string -> option(info)
  def screenQuery / agentQuery / componentQuery : null -> stream(info)

  // Libraries (since PR1213)
  def units : null -> array(string)
  def unitQuery : null -> stream(unitInfo)

  // Navigation
  def push : string -> array(data) -> data
  def pushWithTransition : string -> string -> array(data) -> data   // PR989

  // Spreadsheet introspection
  def spreadsheet : data -> data    // null = whole spreadsheet; or filter map
  def enablePropagation / disablePropagation : data -> data

  // Case decomposition
  def decomposeCase : any -> option([string, option(any)])      // PR1160
  def decomposeCaseData : data -> option([string, option(data)])  // PR1312

  // Deprecated (use moduleInfo instead)
  def moduleArity / moduleParams / moduleMeta : string -> ...
module end
```

### Key Notes

**Module names (since PR1213):** `modules()` returns qualified names like `foo-20221213@abcdef.M`. Pass these to `moduleInfo`. The `space` field: `modPhysical` for real modules;
`modScreen/modAgent/modComponent` for meshes.

**`reflect.spreadsheet(filter)`** → map of cell records:

```
{ "cellName": { value, mode, mixName, meta, fullName, frozen, dependsOn, aliasOf } }
```

- `mode`: `"in"` (var), `"out"`, `"prop"` (private propagating), etc.
- Use `reflect.spreadsheet({fullName: cellName(x)})` to query a specific cell.

**`reflect.push(name, params)`** — navigate to a screen by name dynamically. Returns value when the pushed screen is popped. Can fail if: name unknown, wrong param count, non-serializable params, or
module too simple.

**`reflect.decomposeCase(v)`** → `some(["caseName", some(arg)])` or `null` if not a case value. E.g. `decomposeCase(some(123))` → `some(["some", some(123)])`.

**`reflect.disablePropagation()`** / **`reflect.enablePropagation()`** — temporarily pause cell update propagation in REPL; `enablePropagation` re-evaluates whole spreadsheet.

**`reflect.moduleCheck(v)`** — returns `some({name: "X"})` if `v` is a module instance, otherwise `null`.

**Spreadsheet statistics** (`reflect.statistics()`):

```
{ cells, inCells, outCells, propCells, undefCells, frozenCells,
  redefinitions, aliases, updates, outputs, inversions, actionClosures }
```

---

## `reflectComponent` — Instantiate Registered Components by Name

> Source: [reflectComponent](https://www.notion.so/1061d464528f817ebfc7d9c8a6e67b84) (since PR 1201)

```
module reflectComponent
  def instantiateComponent : string -> map(link(data)) -> module type slateComponent
module end
```

- Runs a registered component by name. Component must appear in `reflect.components()`.
- `params` is a `map(link(data))` — use `link(cell)` or `newConstCell(expr)` for arbitrary values.
- Returns a module with `.output` (map(data) of output params) and `.view` (DOM from screen card).
- **Performance warning**: runs via slate — much slower than normal components. Not for use in lists.

```
private cell p1 = 42
private cell c = reflectComponent.instantiateComponent("MyComp", { p1: link(p1) }:map)
alias c_output = c.output
alias c_view = c.view
```

---

## `slateComponent` — Run a Mesh as a Component

> Source: [slateComponent](https://www.notion.so/1061d464528f81caaa78f020598d6157) (since PR 1201)

```
module slateComponent(mod_name:string, mesh:slateDef.mesh, params:map(link(data)))
  cell output : data    // map(data) of output params; empty map if no output node
  cell view : data      // value of the screen card
module end
```

- `mod_name` — used for cell naming in traces/profiling; does not need to be unique.
- `params` — pass with `link(existingCell)` or `newConstCell(expr)` for computed values.
- **Performance warning**: much slower than normal components. Not for use in lists.