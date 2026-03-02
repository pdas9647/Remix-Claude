# Mix Standard Library — Module Index

> Source: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)
> These modules are always available in any app.

---

## Module Quick Reference

| Module             | Description                                       | Detail File                      |
|--------------------|---------------------------------------------------|----------------------------------|
| `bool`             | Kleenean 3-valued logic                           | mix-lang-core.md                 |
| `number`           | Arithmetic, math, rounding, format, bitsets       | mix-lang-core.md                 |
| `string`           | String ops, split/concat, JSON parse/format       | mix-lang-core.md                 |
| `option`           | null/some(x), map/andThen                         | mix-lang-core.md                 |
| `result`           | error/ok, map/andThen/orElse                      | mix-lang-core.md                 |
| `bitset`           | Integer sets (128-bit vector tree)                | mix-lang-core.md                 |
| `passThrough`      | Wrap value in cell for link()                     | mix-lang-core.md                 |
| `regex`            | Compile/find/replace/split/templates/lex          | mix-lang-core.md                 |
| `array`            | Map/filter/reduce/sort/join                       | mix-lang-collections-basic.md    |
| `set`              | Immutable string sets, set-theoretic ops          | mix-lang-collections-basic.md    |
| `map`              | String-keyed maps: get/set/merge/descend          | mix-lang-collections-basic.md    |
| `gset`             | Generic sets (key-based dedup)                    | mix-lang-collections-advanced.md |
| `uniqArray`        | Ordered arrays with key uniqueness                | mix-lang-collections-advanced.md |
| `circuitarray`     | Module-per-row instantiation, keyed caching       | mix-lang-collections-advanced.md |
| `db`               | Primary DB: CRUD, query pipeline, live queries    | mix-lang-db.md                   |
| `db-changes`       | Old→new db module API migration                   | mix-lang-db.md                   |
| `dbRemote`         | HTTP-protocol remote DB access                    | mix-lang-db.md                   |
| `minidb`           | In-session pure-Mix DB (Wasm-compatible)          | mix-lang-db.md                   |
| `query`            | AST builder for db filters/projections            | mix-lang-query.md                |
| `registry`         | Session-scoped case value store (experimental)    | mix-lang-query.md                |
| `resource`         | Static resource URLs *(deprecated → file)*        | mix-lang-query.md                |
| `http`             | HTTP client: GET/POST/PUT/PATCH/DELETE            | mix-lang-io.md                   |
| `file`             | Blob storage: get/read/list/create/register       | mix-lang-io.md                   |
| `binary`           | Byte sequences: slice/concat, encode/decode       | mix-lang-io.md                   |
| `crypto`           | Hashing (MD5/SHA/HMAC/RSA), AES-GCM               | mix-lang-io.md                   |
| `wasm`             | WebAssembly FFI: load/funN/wasiCall               | mix-lang-io.md                   |
| `mime`             | MIME headers, media types, multipart              | mix-lang-io.md                   |
| `stream`           | Lazy streams: map/filter/reduce/sort/zip/join     | mix-lang-stream.md               |
| `websocket`        | WebSocket client: read/write/channels             | mix-lang-stream.md               |
| `xml`              | XML parse/navigate/search (server-only)           | mix-lang-stream.md               |
| `agent`            | Agent descriptors & invocation (local/remote)     | mix-lang-platform-runtime.md     |
| `auth`             | Auth helpers: signIn/Up, resetPassword            | mix-lang-platform-runtime.md     |
| `env`              | Runtime env vars, identity, base URLs             | mix-lang-platform-runtime.md     |
| `secrets`          | Per-user encrypted secret/token storage           | mix-lang-platform-runtime.md     |
| `logging`          | Log info/warning/error → client console           | mix-lang-platform-concurrency.md |
| `metrics`          | Timers, profiling, tracing, statistics            | mix-lang-platform-concurrency.md |
| `co`               | Coroutines, channels, pub/sub, mutexes            | mix-lang-platform-concurrency.md |
| `embed`            | Named embedded screens (group/instance)           | mix-lang-platform-concurrency.md |
| `track`            | Amp session control via WebSocket                 | mix-lang-platform-concurrency.md |
| `memo`             | Mutable add/remove with ordered handles           | mix-lang-platform-concurrency.md |
| `random`           | Pseudo/crypto random ints, floats, strings        | mix-lang-platform-concurrency.md |
| `util`             | withDefault, isDefined, ifThenElse                | mix-lang-platform-concurrency.md |
| `calendar`         | Date/time types, arithmetic, weeks, quarters      | mix-lang-platform-messaging.md   |
| `color`            | Color spaces (SRGB/HSL/HWB), mix, CSS             | mix-lang-platform-messaging.md   |
| `messaging-pubsub` | Pub/sub topics, retained, live queries, cloud     | mix-lang-platform-messaging.md   |
| `loader`           | Dynamic library loading, expose screens/agents    | mix-lang-platform-messaging.md   |
| `dom`              | DOM types: create/access/serialize, sync cells    | mix-lang-editor-dom.md           |
| `editor-node`      | Nodegraph introspection: bindings, traversal      | mix-lang-editor-dom.md           |
| `editor-screen`    | Screen-level introspection: byRef/byName          | mix-lang-editor-dom.md           |
| `editor-types`     | Binding direction (input/output), visual types    | mix-lang-editor-dom.md           |
| `reflect`          | Module/screen/agent introspection, push nav       | mix-lang-editor-reflect.md       |
| `reflectComponent` | Instantiate component by name → output+view       | mix-lang-editor-reflect.md       |
| `slateComponent`   | Run mesh as component → output+view               | mix-lang-editor-reflect.md       |
| `slateDef`         | Mesh definition: create/wire/label/compose        | mix-lang-editor-slate.md         |
| `slateDyn`         | Dynamic slate registration: screen/agent/comp     | mix-lang-editor-slate.md         |
| `slateGen`         | Generate/save Mix code from mesh                  | mix-lang-editor-slate.md         |
| `compiler`         | Library/executable type defs, libID, storedBinary | mix-lang-editor-runtime.md       |
| `container`        | Array/map ops: safeGet/descend, pathUpdate, merge | mix-lang-editor-runtime.md       |
| `viewstack`        | Viewstack nav: push/pop/switch/embed/propagate    | mix-lang-editor-runtime.md       |
| `debugger`         | Remote debug, Mix event stream, DOM mirror        | mix-lang-editor-runtime.md       |
| `testing`          | Record/replay tests, snapshots, break-resume      | mix-lang-editor-runtime.md       |
| `vm`               | Additional VM instances: create/invoke/output     | mix-lang-editor-runtime.md       |

---

## Extra Core Libraries (`mixc/`)

Not part of stdlib; must be explicitly imported.

| Library        | Module           | Description                                                                                             |
|----------------|------------------|---------------------------------------------------------------------------------------------------------|
| `mixc/parsing` | `csv`            | RFC-4180 CSV → `array(array(string))`. [Notion](https://www.notion.so/1451d464528f80688a22ca276faf82d9) |
| `driver`       | `compilerRule`   | Build rule records: scan/load/save/makeLibRule                                                          |
| `driver`       | `compilerFile`   | Source file records: scan/load/save/make                                                                |
| `driver`       | `compilerLib`    | Library DB records: scan/load/save/strip/publish/closure                                                |
| `driver`       | `compilerExe`    | Executable DB records: scan/load/save/publish                                                           |
| `driver`       | `compilerDriver` | Session orchestration: create/compile/save                                                              |
| `driver`       | `compilerREPL`   | Send compiled code to VM via connector                                                                  |
| `driver`       | `compilerClean`  | Cleanup (undocumented)                                                                                  |
| `driver`       | `compilerBuild`  | High-level build pipeline: execute/execOn                                                               |
| `driver`       | `codegenDriver`  | Generate Mix source from editor screen DB records                                                       |
| `driver`       | `codegenUtil`    | mkStylesRecord/mkFileMetaRecord                                                                         |
| `driver`       | `remixFile`      | Build .remix zip: manifest + runtime.json + appMeta                                                     |
| `driver`       | `remixRecipe`    | Declarative .remix recipe types                                                                         |

Detail files: [mix-driver-compiler.md](./mix-driver-compiler.md), [mix-driver-build.md](./mix-driver-build.md)
