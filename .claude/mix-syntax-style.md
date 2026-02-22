# Mix Language — Directives, Pragmas & Style Guide

> Sources:
> - [directives](https://www.notion.so/1061d464528f81c08c15e5ae817171ee)
> - [styleguide](https://www.notion.so/1061d464528f819099d2e81ae669855d)
>
> Parent: [Mix Syntax](https://www.notion.so/1061d464528f81d492f4e1b0f8c8adf4) → [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

See also: [mix-syntax-core.md](./mix-syntax-core.md) | [mix-syntax-semantics.md](./mix-syntax-semantics.md)

---

## Directives & Typing Modes

### Lexing directives

```
#line <line> "<filename>"     // set line number + filename for next line (filename optional)
#push <line> "<filename>"     // like #line but scoped; undo with #pop (since PR 547)
#pop                          // restore previous line/filename state (since PR 547)
```

Must be at leftmost column. `#push`/`#pop` can be nested.

### Source redirect (since PR 557)

```
module m
_source_ { url:"https://wherever" }
```

Redirects amp to fetch source from a remote URL (HTTP GET → 200). No other definitions or directives allowed; no module params. Must be publicly accessible. Cannot nest redirects.

### Double semicolon `;;`

Forces end of current `def` or `cell` phrase. Useful for separating known-good and questionable code sections so error line numbers are accurate:

```
def x = 10 ;;
[0] ;;        // error here points here, not to x above
def y = 20 ;;
```

### Pragmas (`_pragma_`)

Placed between `def`s and `cell`s; effective until end of module or next pragma:

```
_pragma_ { name: value, name: value, ... }   // values are any JSON
```

**Warning control** (since PR#1424):

- `warningFilter:"<spec>"` — comma-separated warning names with optional `+`/`-` prefix. `"-deprecated"` disables deprecation warnings; `"+deprecated"` re-enables. See `mix_client -warn-print` for full list.

**Typing pragmas** (since PR#534) — `auto*` are ON by default; `strict*` are OFF by default:

- `autoTypeAssertion:false` — disable auto runtime type assertions for downcasts
- `autoMakeStream:false` — disable auto conversion of values to singleton streams (e.g. `1 ++ 2`)
- `autoMakeLink:false` — disable auto conversion of cell names to links where needed
- `strictContainerTyping:true` — container literals (tuples, maps) get concrete types instead of `data`
- `strictTyping:true` — enables all of the above plus stricter typing rules (since PR#892):
    - Default (non-strict): inserts runtime assertions, falls back to `data`, compound literals typed as `data`
    - Strict: no auto-conversions, types kept minimal, compound literals get concrete types (e.g. tuple literals → tuple types)

**Spreadsheet pragmas**:

- `withCells:false` — forbids adding cells to the module (lighter translation)
- `withCells:true` — enforces spreadsheet machinery availability (for stdlib use only)

---

## Meta Data

Modules, functions, and cells can carry a `_meta_` annotation (always a constant JSON object):

```
module M _meta_ { ... }
def f : t _meta_ { ... }
cell c : t _meta_ { ... }
```

Common meta forms:

- **Deprecation**: `_meta_ { deprecated:true, expires:"yyyy-mm-dd", alternatives:["alt1"] }`
- **Renaming**: `_meta_ { renamed:true, expires:"yyyy-mm-dd", replacement:"newname" }`
- **Module type**: `_meta_ { modType:"screen" }` — values: `screen`, `agent`, `component`

---

## Code Style Guide

> Source: [styleguide](https://www.notion.so/1061d464528f819099d2e81ae669855d)

Rules for the indentation-based parser:

**Definitions and cells**: starting keyword (`def`, `cell`) on column 1. Content either entirely on same line or indented on following lines — never partly on the first line and partly below.

```
def x = 1                    // good: all on one line
def y =                      // good: indented on next line
  2
def f = (x ->                // BAD: content split across first + following line
  x+1)
```

**Indentation**: subexpressions spanning multiple lines must be indented; indentation must fit the bracket structure. Entire subexpression on one line → no indentation needed.

**Closing brackets**: right bracket on the same column as left bracket (or "bracket trail" — all closings on one line). Forbidden: right bracket more to the left than the left bracket.

```
cell x =                     // good: matching columns
  [ { foo: 1 },
    { foo: 2,
      bar: "x"
    }
  ]
cell y =                     // good: bracket trail
  [ { foo: 1 },
    { foo: 2, bar: "x" } ]
```

**Function calls**: all on one line, or name on line 1 and args on line 2 indented, or args across lines aligned on the same column.

```
f(1, 2, 3)                   // good
f                            // good
  (1, 2, 3)
f(1,                         // good: args aligned
  2,
  3)
longname(1,                  // BAD: args not aligned
  2, 3)
```

**Long arithmetic**: continuation line starts with the operator symbol.

```
"hello " + "world"           // good: + at start of continuation
  + " again"
"hello " + "world" +         // BAD: operator at end of continued line
  " again"
```

**Commas**: all elements on one line, or every element on a new line with trailing comma (except last).

**`let` sequences**: each `let` on a new line, aligned; `;` at line end.

```
def f =
  (x ->
    let a = x+2;
    let b = x*x*x;
    a+b )
```