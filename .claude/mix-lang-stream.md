# Mix Standard Library — Stream, XML & WebSocket Modules

> Sources:
> - [stream](https://www.notion.so/1061d464528f81fd8334c07f89c15b0f)
> - [xml](https://www.notion.so/11ca20aeafb348a38dde4f513b146001)
> - [websocket](https://www.notion.so/1061d464528f81e8af3dd2ed1d92f1fb)
>   > Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `stream` — Lazy Stream Processing

> Source: [stream](https://www.notion.so/1061d464528f81fd8334c07f89c15b0f)
> Also: `dataStream` module — identical to `stream` but limited to `stream(data)`.

### Module Signature

```
module stream
  // Basics
  def empty : stream(S)
  def singleton : S -> stream(S)
  def isEmpty : stream(S) -> bool
  def isNotEmpty : stream(S) -> bool
  def length : stream(S) -> number

  // Mapping & filtering (lazy; consume input only as needed)
  def map : (S -> T) -> stream(S) -> stream(T)
  def map2 : (S -> T -> U) -> stream(S) -> stream(T) -> stream(U)
  def indexedMap : (number -> S -> T) -> stream(S) -> stream(T)
  def indexedMap2 : (number -> S -> T -> U) -> stream(S) -> stream(T) -> stream(U)
  def indexOnlyMap : (number -> T) -> stream(S) -> stream(T)
  def flatMap : (S -> stream(T)) -> stream(S) -> stream(T)
  def zip : stream(S) -> stream(T) -> stream([S,T])
  def filter : (S -> bool) -> stream(S) -> stream(S)
  def deduplicateKeepFirst : (S -> string) -> stream(S) -> stream(S)
  def deduplicateKeepLast : (S -> string) -> stream(S) -> stream(S)

  // Reducing (consume entire stream)
  def reduce : (S -> T -> T) -> T -> stream(S) -> T
  def indexedReduce : (number -> S -> T -> T) -> T -> stream(S) -> T
  def lreduce : (S -> T -> T) -> (T -> stream(U)) -> (T -> stream(U)) -> T -> stream(S) -> stream(U)
  def andThen : (T -> result(S,U)) -> result(S, stream(T)) -> result(S, stream(U))

  // Min/max
  def minIndex : stream(data) -> number
  def minValue : stream(data) -> A
  def minIndexBy : (A -> B) -> stream(A) -> number
  def minValueBy : (A -> B) -> stream(A) -> A
  def maxIndex : stream(A) -> number
  def maxValue : stream(A) -> A
  def maxIndexBy : (A -> B) -> stream(A) -> number
  def maxValueBy : (A -> B) -> stream(A) -> A

  // Construction
  def lazy : (null -> stream(T)) -> stream(T)
  def initialize : number -> (number -> S) -> stream(S)
  def initUnbound : (number -> T) -> stream(T)
  def enumerate : number -> stream(number)
  def enumUnbound : null -> stream(number)
  def rev : stream(S) -> stream(S)

  // Permutations
  def permuteEnum : number -> stream(array(number))
  def permuteArray : array(S) -> stream(array(S))

  // Iteration
  def force : stream(S) -> stream(S)
  def iter : (S -> null) -> stream(S) -> null
  def exists : (S -> bool) -> stream(S) -> bool
  def forAll : (S -> bool) -> stream(S) -> bool
  def contains : S -> stream(S) -> bool
  def find : (S -> bool) -> stream(S) -> S
  def findMapped : (S -> T) -> stream(S) -> T
  def findMapped2 : (S -> T -> U) -> stream(S) -> stream(T) -> U

  // Slicing
  def first : stream(S) -> S
  def firstWithDefault : S -> stream(S) -> S
  def last : stream(S) -> S
  def lastWithDefault : S -> stream(S) -> S
  def skip : stream(S) -> stream(S)
  def skipn : number -> stream(S) -> stream(S)
  def firstn : number -> stream(S) -> stream(S)
  def lastn : number -> stream(S) -> stream(S)

  // Sort & merge (consume input)
  def grouped : (S -> data) -> stream(S) -> stream(array(S))
  def merge : (S -> data) -> (S -> data) -> stream(S) -> stream(S) -> stream(S)
  def sort : (S -> data) -> stream(S) -> stream(S)
  def sort_ci : (S -> string) -> stream(S) -> stream(S)
  def dirSort : bool -> (S -> data) -> stream(S) -> stream(S)
  def dirSort_ci : bool -> (S -> string) -> stream(S) -> stream(S)
  def sortBy : (A -> A -> bool) -> stream(A) -> stream(A)

  // Relations
  def innerJoin : (S -> string) -> stream(S) -> (T -> string) -> stream(T) -> (S -> T -> U) -> stream(U)

  // Conversion
  def toArray : stream(S) -> array(S)

  // Tee (split one stream into two)
  type tee(S)
  def tee : stream(S) -> tee(S)
  def leftOut : tee(S) -> stream(S)
  def rightOut : tee(S) -> stream(S)
module end
```

### Key Behaviors

- **Mutability**: Streams are mutable. `stream.first(s)` **consumes** the first element; `skip`/`skipn` also consume.
- **Laziness**: Most operations only consume input as needed. Exceptions: `sort`, `rev`, `lastn`, `deduplicateKeepLast`, `reduce` consume entire input.
- **Array to stream**: `a[]` converts array to stream. Slices: `a[p..]`, `a[..q]`, `a[p..q]`.
- **Concatenation**: `s ++ t` — drains `s` then `t`. **`undefined` not allowed** in streams.
- `map2` / `zip` — both inputs must end simultaneously (error otherwise).
- `deduplicateKeepFirst` — lazy; `deduplicateKeepLast` — consumes whole stream upfront.
- `lreduce` — lazy reduce emitting intermediate results as a stream.
- `andThen` — maps over stream elements; stops and returns `error` on first error from `f`.
- `grouped(f, s)` — groups adjacent elements where `f(x) == f(y)`.
- `innerJoin` — memoizes left stream entirely; matches by key string. Partially applicable.
- `tee` — splits stream into two with internal buffering; safe from different coroutines.
- `lazy(f)` — deferred stream; needed for recursive stream definitions.

---

## `xml` — XML Parsing & Navigation

> Source: [xml](https://www.notion.so/11ca20aeafb348a38dde4f513b146001)
> **Server-only** (amp runtime). Not available in client/browser VMs — missing FFIs.

### Types

```
case textNode(string)            // decoded text content (entities already resolved)
case elementNode(elementRec)     // element with attributes and children
type node = textNode | elementNode

type attributeRec = { name, space, prefix, local, value : string }
type elementRec   = { name, space, prefix, local : string,
                      props: map(attributeRec),
                      children: array(node) }
```

- `name` — `"prefix:local"` for namespaced, `"local"` otherwise. Only `name` and `space` are authoritative; `prefix`/`local` are parser convenience.
- `space` — namespace URI (`""` if none). `namespaces` map in `parse` normalizes prefixes.
- `props` — excludes `xmlns`/`xmlns:prefix` attrs; formatter re-adds them. Simplified form accepted: `{ att1: "one" }`.
- Only `textNode`/`elementNode` — comments, directives, PIs are ignored.

### Module Signature

```
module xml
  // Parse / format
  def parse : data -> map(string) -> string -> node   // parse(config, namespaces, xmlText)
  def format : data -> node -> string                 // format(config, tree)

  // JSON conversion
  def toJSON : node -> data
  def ofJSON : data -> node

  // Node type predicates
  def isElement : node -> bool
  def isText : node -> bool
  def keepElement : node -> elementNode(elementRec)   // undefined if not element
  def keepText : node -> textNode(string)             // undefined if not text

  // Attribute access
  def hasAttribute : string -> node -> bool
  def getAttribute : string -> node -> attributeRec
  def getAttributeValue : string -> node -> string
  def findAttribute : (attributeRec -> bool) -> node -> attributeRec
  def getId : node -> string

  // Content & navigation
  def getText : node -> string           // concatenated text of all text node(s)
  def elementName : node -> string       // undefined for text nodes
  def children : node -> array(node)
  def descend : array(number) -> node -> node   // walk path; undefined if missing
module end
```

**Search functions** (shallow = direct children; deep = self + all descendants in textual order):

| Pattern                                 | shallow | deep                  |
|-----------------------------------------|---------|-----------------------|
| `Find(predicate, node) → node`          | yes     | yes                   |
| `FindMapped(f, node) → S`               | yes     | yes                   |
| `FindIndex / FindPath`                  | →number | →array(number)        |
| `FindByName(name, node) → node`         | yes     | yes                   |
| `FindIndexByName / FindPathByName`      | →number | →array(number)        |
| `Filter(predicate, node) → array(node)` | yes     | yes + `IndexedFilter` |

**Find by ID** — same pattern: `FindById`, `FindIndexById` (shallow), `FindPathById` (deep). Bulk: `getMappingIdToPath`, `getMappingIdToNode`.

**Streaming**: `getIndexedNodeStream(node) → stream([path, node])`, `getNodeStream`, `getTextStream`, `getChildrenStream` — all return nodes in textual order.

**Entity decoding**: `decodeEntities(entityMap, s)`, `xmlEntities()`, `htmlEntities()` (full whatwg set). Decodes `&#65;`, `&#x42;`, `&amp;` etc.

### Key Behaviors

- `parse(config, ns, text)` — `config` unused (pass `{}`); `ns` maps namespace URIs to preferred prefixes
- `format(config, tree)` — `config` unused (pass `{}`); emits `xmlns` attributes as needed
- `toJSON`/`ofJSON` — simplified representation: elements as objects, text as strings
- `FindMapped` variants — return first non-`undefined` result of mapping function
- `getMappingIdToNode` — efficient for repeated id lookups (build once, use map)
- Iterate: `xml.getChildrenStream(n) |> stream.iter(...)` or `for (var x in ...)`

---

## `websocket` — WebSocket Client

> Source: [websocket](https://www.notion.so/1061d464528f81e8af3dd2ed1d92f1fb)

### Module Signature

```
module websocket
  type client = <"websocketClient">

  case textMsg(string)
  case binMsg(data)
  case pingMsg(data)
  case pongMsg(data)
  case closeMsg(data)    // payload: {code: number, text: string}
  type message = textMsg | binMsg | pingMsg | pongMsg | closeMsg

  def create : data -> client
  def isDown : client -> bool
  def close : client -> null
  def read : client -> message
  def readStream : client -> message*
  def readTextStream : client -> string*
  def write : client -> message -> null
  def writeChannel : client -> co.channel(message)
  def writeTextChannel : client -> co.channel(string)
  def keepAlive : client -> null

  // Close codes (1000–1015): NormalClosure, GoingAway, ProtocolError, UnsupportedData,
  //   NoStatusReceived, AbnormalClosure, InvalidPayload, PolicyViolation, MessageTooBig,
  //   MandatoryExtension, InternalServerErr, ServiceRestart, TryAgainLater, TLSHandshake
  def makeCloseMsg : number -> string -> closeMsg(data)
  def isCloseMsg : message -> bool
  def extractCodeFromCloseMsg : message -> number
  def extractTextFromCloseMsg : message -> string

  def isTextMsg : message -> bool
  def extractTextMsg : message -> string
  def isBinMsg : message -> bool
  def extractBinMsg : message -> data
module end
```

### `create` Config Object

```
{ url: string,            // required; must be ws:// or wss://
  timeout: number,        // handshake timeout in seconds (default: 300)
  readTimeout: number,    // max time between messages (default: 0 = none)
  oauth_token: string,    // Bearer auth token
  basic_auth: { username, password },
  headers: map,           // extra HTTP headers for WS upgrade
  subprotocols: [string]  // Sec-Websocket-Protocol values
}
```

### Key Behaviors

- Client tied to current view — auto-closed when view is popped or session times out
- `read` — blocks until next message; after `closeMsg`, repeated calls return same `closeMsg`
- `readStream` — all messages including final `closeMsg`; `readTextStream` — text only
- `write` — concurrent writes not ordered; **use `writeChannel` for guaranteed order**
- `writeTextChannel` — sends strings as `textMsg`; appends close at end
- `keepAlive` — starts coroutine sending periodic pings
- `close` — low-level TCP shutdown; prefer sending `closeMsg` and letting auto-management handle it
- Locally generated codes: `1005` (bad format), `1006` (socket error), `1015` (TLS failure)
- **Wasm limitations**: no ping/pong; some close codes unavailable; no headers/auth (subprotocols ok)
