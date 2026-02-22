# Mix Standard Library — Module Index

> Source: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)
> These modules are always available in any app.

---

## Module Quick Reference

| Module             | Description                                                                                        | Detail File                      |
|--------------------|----------------------------------------------------------------------------------------------------|----------------------------------|
| `array`            | Array operations: map/filter/reduce/sort/join                                                      | mix-lang-collections-basic.md    |
| `bool`             | Kleenean 3-valued logic (_not/and/or/toString)                                                     | mix-lang-core.md                 |
| `number`           | Arithmetic, math, rounding, format, bitsets                                                        | mix-lang-core.md                 |
| `string`           | String ops, split/concat, JSON parse/format                                                        | mix-lang-core.md                 |
| `agent`            | Agent descriptors & invocation (local/remote/persistent)                                           | mix-lang-platform-runtime.md     |
| `auth`             | Auth helpers: emptyToken, signIn/Up, resetPassword (auth0)                                         | mix-lang-platform-runtime.md     |
| `binary`           | Byte sequences: slice/concat/indexing, encode/decode                                               | mix-lang-io.md                   |
| `bitset`           | Integer sets (search tree of 128-bit vectors)                                                      | mix-lang-core.md                 |
| `calendar`         | Date/time types (date/dateTime/timestamp), arithmetic, weeks, quarters                             | mix-lang-platform-messaging.md   |
| `circuitarray`     | Instantiate a module over each row of an array; keyed caching for reorderable lists                | mix-lang-collections-advanced.md |
| `co`               | Coroutines, channels, pub/sub, mutexes, live cells                                                 | mix-lang-platform-concurrency.md |
| `color`            | Color spaces (SRGB/LinRGB/HSL/HWB), mix, brightness, CSS format                                    | mix-lang-platform-messaging.md   |
| `compiler`         | Library/executable type defs: libID, library, executable, storedBinary                             | mix-lang-editor-runtime.md       |
| `container`        | Array & map ops: safeGet/descend, pathUpdate, shallowMap/deepMap, merge                            | mix-lang-editor-runtime.md       |
| `crypto`           | Hashing (MD5/SHA/HMAC/RSA), AES-GCM encryption                                                     | mix-lang-io.md                   |
| `db`               | Primary DB: CRUD, query pipeline, live queries                                                     | mix-lang-db.md                   |
| `db-changes`       | Migration notes: old→new db module API                                                             | mix-lang-db.md                   |
| `dbRemote`         | HTTP-protocol DB access (remote only)                                                              | mix-lang-db.md                   |
| `debugger`         | Remote debug commands, Mix event stream, DOM mirror stream                                         | mix-lang-editor-runtime.md       |
| `dom`              | DOM type: create/access/serialize nodes, sync cells, symbol table                                  | mix-lang-editor-dom.md           |
| `editor-node`      | Nodegraph node introspection: bindings, connections, traversal                                     | mix-lang-editor-dom.md           |
| `editor-screen`    | Screen-level editor introspection: byRef/byName, children                                          | mix-lang-editor-dom.md           |
| `editor-types`     | Binding direction (input/output) and visual types (vty_*)                                          | mix-lang-editor-dom.md           |
| `embed`            | Named embedded screens (group/instance/toggle)                                                     | mix-lang-platform-concurrency.md |
| `env`              | Runtime env vars, identity, base URLs, client props                                                | mix-lang-platform-runtime.md     |
| `file`             | Blob file storage: get/read/list/create/register                                                   | mix-lang-io.md                   |
| `gset`             | Generic sets of any type (key-based dedup)                                                         | mix-lang-collections-advanced.md |
| `http`             | HTTP client: GET/POST/PUT/PATCH/DELETE + config                                                    | mix-lang-io.md                   |
| `loader`           | Dynamic library loading, expose screens/agents/components, dynamic import                          | mix-lang-platform-messaging.md   |
| `logging`          | Log messages: info/warning/error (→ client console)                                                | mix-lang-platform-concurrency.md |
| `map`              | String-keyed maps: get/set/merge/descend/invert                                                    | mix-lang-collections-basic.md    |
| `memo`             | Mutable add/remove container with ordered unique handles                                           | mix-lang-platform-concurrency.md |
| `messaging-pubsub` | High-level pub/sub: topics, retained set/get, non-retained send, live queries, remote/cloud topics | mix-lang-platform-messaging.md   |
| `metrics`          | Timers, profiling, tracing, statistics, Mix event log                                              | mix-lang-platform-concurrency.md |
| `mime`             | MIME headers, media types, multipart/form-data                                                     | mix-lang-io.md                   |
| `minidb`           | In-session pure-Mix DB, Wasm-compatible                                                            | mix-lang-db.md                   |
| `option`           | Option type: null/some(x), map/andThen                                                             | mix-lang-core.md                 |
| `passThrough`      | Wrap value in anonymous cell for link()                                                            | mix-lang-core.md                 |
| `query`            | AST builder for db filters and projections                                                         | mix-lang-query.md                |
| `random`           | Pseudo/crypto random ints, floats, strings; alphabet constants                                     | mix-lang-platform-concurrency.md |
| `reflect`          | Module/screen/agent introspection, push navigation, spreadsheet inspect                            | mix-lang-editor-reflect.md       |
| `reflectComponent` | Instantiate registered component by name → output+view                                             | mix-lang-editor-reflect.md       |
| `regex`            | Regex: compile/find/replace/split/templates/lex                                                    | mix-lang-core.md                 |
| `registry`         | Session-scoped case value store (experimental)                                                     | mix-lang-query.md                |
| `resource`         | Static resource URLs *(deprecated → use file)*                                                     | mix-lang-query.md                |
| `result`           | Result type: error/ok, map/andThen/orElse                                                          | mix-lang-core.md                 |
| `secrets`          | Per-user encrypted secret/token storage                                                            | mix-lang-platform-runtime.md     |
| `set`              | Immutable sets of strings, set-theoretic ops                                                       | mix-lang-collections-basic.md    |
| `slateComponent`   | Run a mesh as component (mod_name, mesh, params) → output+view                                     | mix-lang-editor-reflect.md       |
| `slateDef`         | Mesh definition: create/wire/label/compose/export nodes                                            | mix-lang-editor-slate.md         |
| `slateDyn`         | Dynamic slate registration: screen/agent/component + loadCollections                               | mix-lang-editor-slate.md         |
| `slateGen`         | Generate/save Mix code from mesh; getImports                                                       | mix-lang-editor-slate.md         |
| `stream`           | Lazy streams: map/filter/reduce/sort/zip/join                                                      | mix-lang-stream.md               |
| `testing`          | Record/replay app tests, snapshots, break-and-resume protocol                                      | mix-lang-editor-runtime.md       |
| `track`            | Amp session control: create/compile/view via WebSocket                                             | mix-lang-platform-concurrency.md |
| `uniqArray`        | Ordered arrays with key-based uniqueness (exp.)                                                    | mix-lang-collections-advanced.md |
| `util`             | withDefault/noDefault, isDefined/isUndefined, ifThenElse                                           | mix-lang-platform-concurrency.md |
| `viewstack`        | Viewstack nav: push/pop/switch/embed/schedule/propagate                                            | mix-lang-editor-runtime.md       |
| `vm`               | Additional VM instances: create/destroy/sendCode/invoke/outputStream                               | mix-lang-editor-runtime.md       |
| `wasm`             | WebAssembly FFI: load/funN/wasiCall                                                                | mix-lang-io.md                   |
| `websocket`        | WebSocket client: read/write/channels                                                              | mix-lang-stream.md               |
| `xml`              | XML parse/navigate/search (server-only)                                                            | mix-lang-stream.md               |

---

## Extra Core Libraries (`mixc/`)

Not part of the stdlib; must be explicitly imported. Details in separate files.

| Library        | Module           | Description                                                                                   | Detail File            |
|----------------|------------------|-----------------------------------------------------------------------------------------------|------------------------|
| `mixc/parsing` | `csv`            | Parse RFC-4180 CSV → `array(array(string))`                                                   | mix-lang-advanced.md   |
| `driver`       | `compilerRule`   | Build rule records: scan/load/save/makeLibRule/saveLibRule/getExeRuleHash                     | mix-driver-compiler.md |
| `driver`       | `compilerFile`   | Source file records: scan/load/save/make (computes imports via session)                       | mix-driver-compiler.md |
| `driver`       | `compilerLib`    | Library DB records: scan/load/save/strip/publish/closure (topo sort)                          | mix-driver-compiler.md |
| `driver`       | `compilerExe`    | Executable DB records: scan/load/save/publish/createExeFromLibs (amp only)                    | mix-driver-compiler.md |
| `driver`       | `compilerDriver` | Session orchestration: createSession/compilePhrases/compileModules/saveLibrary/saveExecutable | mix-driver-compiler.md |
| `driver`       | `compilerREPL`   | Send compiled code to VM via connector: eval/sendCodeFromCompiler                             | mix-driver-compiler.md |
| `driver`       | `compilerClean`  | Cleanup (no API documented yet)                                                               | mix-driver-compiler.md |
| `driver`       | `compilerBuild`  | High-level build pipeline: execute/execOn/execEmbedImports                                    | mix-driver-build.md    |
| `driver`       | `codegenDriver`  | Generate Mix source from editor screen DB records                                             | mix-driver-build.md    |
| `driver`       | `codegenUtil`    | mkStylesRecord/mkFileMetaRecord                                                               | mix-driver-build.md    |
| `driver`       | `remixFile`      | Build .remix zip files: add entries, pack to blob                                             | mix-driver-build.md    |
| `driver`       | `remixRecipe`    | Declarative .remix recipe types (bundle/assets/recipe)                                        | mix-driver-build.md    |

**`csv.parse(s: string) -> array(array(string))`** — Outer array = lines, inner array = fields per line.
Example: `csv.parse("a,b\nc,d")` → `[["a","b"],["c","d"]]`

Source: [parsing-csv](https://www.notion.so/1451d464528f80688a22ca276faf82d9)

---

## Detail Files

- [mix-lang-core.md](./mix-lang-core.md) — Scalar types & utilities: bool, number, string, option, result, bitset, passThrough, regex
- [mix-lang-collections-basic.md](./mix-lang-collections-basic.md) — Collection types: array, set, map
- [mix-lang-collections-advanced.md](./mix-lang-collections-advanced.md) — Collection types: gset, uniqArray, circuitarray
- [mix-lang-db.md](./mix-lang-db.md) — db, db-changes, dbRemote, minidb (CRUD, query pipeline, live queries, filter compilability, references)
- [mix-lang-query.md](./mix-lang-query.md) — query AST builder (filter/projection/aggregation nodes, map constructors), resource (deprecated), registry (experimental)
- [mix-lang-io.md](./mix-lang-io.md) — http, file, binary, crypto, wasm, mime
- [mix-lang-stream.md](./mix-lang-stream.md) — stream (lazy ops: map/filter/reduce/sort/zip/join/tee), xml (parse/navigate/search, server-only), websocket (client: read/write/channels)
- [mix-lang-platform-runtime.md](./mix-lang-platform-runtime.md) — agent, auth, env, secrets
- [mix-lang-platform-concurrency.md](./mix-lang-platform-concurrency.md) — logging, metrics, co, embed, track, memo, random, util
- [mix-lang-platform-messaging.md](./mix-lang-platform-messaging.md) — calendar, color, messaging-pubsub, loader
- [mix-lang-editor-dom.md](./mix-lang-editor-dom.md) — dom/domUtil, editorTypes, editorNode, editorScreen
- [mix-lang-editor-reflect.md](./mix-lang-editor-reflect.md) — reflect, reflectComponent, slateComponent
- [mix-lang-editor-slate.md](./mix-lang-editor-slate.md) — slateDef, slateDyn, slateGen
- [mix-lang-editor-runtime.md](./mix-lang-editor-runtime.md) — container, viewstack, debugger, testing, compiler, vm
