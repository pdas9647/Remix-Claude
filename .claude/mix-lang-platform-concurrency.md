# Mix Standard Library — Platform Modules: Concurrency & Diagnostics

> Sources:
> - [logging](https://www.notion.so/1061d464528f81aab0bbc05142788d0d)
> - [metrics](https://www.notion.so/1061d464528f81e8b4b7dafaca82fac2)
> - [co](https://www.notion.so/1061d464528f8116b68ad82588e6b6c4)
> - [embed](https://www.notion.so/1061d464528f814fbda0e472a2642cd0)
> - [track](https://www.notion.so/0d937a3ad10043c597a3f6b3ef39f3f6)
> - [memo](https://www.notion.so/f1456110dc364bdb95ec4171395cf94c)
> - [random](https://www.notion.so/1061d464528f819f9ae1ddb7092be7cc)
> - [util](https://www.notion.so/1061d464528f814eaf75df71f972c907)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-platform-runtime.md](./mix-lang-platform-runtime.md) | [mix-lang-platform-messaging.md](./mix-lang-platform-messaging.md)

---

## `logging` — Log Messages

```
module logging
  def info : string -> null
  def warning : string -> null
  def error : string -> null
module end
```

Messages are forwarded to the client console. Severities: `info`, `warning`, `error`.

---

## `metrics` — Timers, Profiling & Tracing

```
module metrics
  // Timers (0–4 user-free, ≥5 system)
  def startTimer : number -> null
  def stopTimer : number -> null
  def getTimer : number -> number

  // Profiling
  def enableProfile : appstate -> bool -> null
  def getProfile : null -> data         // {cells:[{name,count,time}], views:[...], actions:[...]}
  def saveProfile : appstate -> string  // saves to db as {_rmx_type:"profile"}
  def clearProfile : null -> null

  // Tracing
  def spreadsheetTracing : number
  def actionTracing : number
  def pubsubTracing : number
  def trace : number -> data -> null
  def setTraceLevels : appstate -> array(number) -> null
  def getTraceLevels : null -> array(number)
  def isTraceLevelEnabled : number -> bool
  def enableTrace : appstate -> bool -> null

  // Statistics (always on, no timing)
  def statsClear : null -> null
  def statsGet : null -> data

  // Mix event log level
  def setMixEventLogLevel : appstate -> string -> null   // "info"|"warning"|"error"
module end
```

### Track API (msg_introspect_debug)

Commands sent via `{ _rmx_type: "msg_introspect_debug", session: "...", command: {name: "..."} }`:

- `profile_enable/disable/clear/get/save/status`
- `trace_enable/disable/status/setLevels/getLevels`
- `stats_clear/get/get_clear`
- `mixEventLogging_set` / `mixEventLogging_status`

Response comes back as `$debugResponse` cell message.

---

## `co` — Coroutines, Channels & Pub/Sub

Single-threaded cooperative concurrency. Coroutines must explicitly yield control.

### Coroutines

```
def create : (routine -> null) -> routine    // background
def createFG : (routine -> null) -> routine  // foreground
def exit : null -> null
def yield : null -> null
```

A coroutine exits on: function return, `co.exit()`, runtime error, `fail(msg)`, dead channel write, or waiting on owned-by-other channel.

### Channels

```
def channelCreate : null -> channel(T)
def channelEmit : channel(T) -> T -> null
def channelEmitEnd : channel(T) -> null
def channelToStream : channel(T) -> T*
def channelFromStream : T* -> channel(T)
def channelOwn : channel(T) -> null
def channelFrom : ((T -> null) -> null) -> channel(T)      // control flow inversion
def channelFromEH : ((T -> null) -> null) -> channel(result(rawErrorObject,T))
```

Ownership: a coroutine owns a channel while waiting on it. When owner exits, channel dies; producers writing to dead channels also exit — enables automatic pipeline cleanup.

### Error Handling

```
def onError : (string -> routine -> null) -> routine -> routine
def apply : (S -> T) -> S -> result(T, string)
def fail : string -> A

// Advanced (since PR1403)
def onErrorRaw / onErrorDebug / applyRaw / applyDebug / failAgain
```

`apply` runs `f(arg)` in a new coroutine; returns `ok(result)` or `error(msg)`. `failAgain` re-throws combining backtraces.

### Time

```
def sleep : number -> null
def applyWithTimeout : number -> (S -> T) -> S -> (result(string,T) | timeout | terminated)
```

### Mutexes

```
def mutexCreate : null -> mutex
def mutexLock / mutexUnlock : mutex -> null
def critical : mutex -> (null -> T) -> T
```

### Live Cells

```
def consume : T* -> T
```

In a `live cell`, re-evaluates each time the stream produces a new value.

### Pub/Sub

```
def pubsubCreate : T -> pubsub(T)
def pubsubConnect : pubsub(data) -> string -> null    // connect to broker topic
def pubsubSubscribe : pubsub(T) -> subscription(T)
def pubsubUnsubscribe : subscription(T) -> null
def pubsubSend : pubsub(T) -> T -> null               // non-retained send
def pubsubSet : pubsub(T) -> T -> null                // retained (mutable variable semantics)
def pubsubRecvStream : subscription(T) -> T*          // non-retained stream
def pubsubGetStream : subscription(T) -> T*           // retained stream
def pubsubRecvLength : subscription(T) -> number
def pubsubGotUpdate : subscription(T) -> bool
def pubsubGet : pubsub(T) -> T
def pubsubClose : pubsub(T) -> null
def pubsubOfSubscription : subscription(T) -> pubsub(T)  // since PR1956
```

**Topic path prefixes** (used with `pubsubConnect`):

- `$workspace/<sub>` — any authenticated user, any app in workspace
- `$wsUser/<sub>` — current user, any app in workspace
- `$app/<sub>` — any user allowed to open the current app
- `$user/<sub>` — current user, current app
- `$session/<sub>` — current session only; auto-deleted when session ends

See also: `messaging` module (higher-level wrapper over pubsub).

---

## `embed` — Named Embedded Screens

```
module embed
  def show : appstate -> string -> string -> appViewRequest -> (data -> null) -> data*
  def hide : appstate -> string -> null
  def toggle : appstate -> string -> string -> appViewRequest -> (data -> null) -> option(data*)
  def placeholderStream : null -> data*
module end
```

Extends `viewstack`. Manages embedded screens by name, enforcing one-visible-per-group semantics.

- **group** — at most one instance visible at a time (e.g. one open dropdown)
- **instance** — identifier for a specific invocation within a group
- **request** — screen name + params; create with `appView S(p1,p2)` or `viewstack.dynamicAppView(name, [p1,p2])`
- `show(appstate, group, instance, req, onclose)` → `data*` stream of DOM from embedded screen. Hides any other screen in the same group.
- `hide(appstate, group)` — terminates viewstack; `onclose` is **not** called.
- `toggle(...)` → `option(data*)` — shows if hidden, hides if same instance is visible, or replaces if different instance.
- `placeholderStream()` — use while embedded screen is hidden to keep a live cell active.

**Typical usage pattern** (dropdown/tabbed):

1. Create embedded screen that calls `viewstack.close(retval)` on user action
2. In main screen, use `co.consume` / `live cell` to consume DOM stream
3. Pass DOM into a container card at overlay position (`position:absolute; zIndex:1`)
4. On `onclose`, receive return value and react (e.g. invoke a set-value action tile)

---

## `track` — Amp Session Control Protocol

Low-level protocol to create and control amp server sessions via WebSocket.

```
module track
  type sid = string
  def createRemote : data -> client   // {baseURL, app, token}
  def close : client -> null
  def write : client -> data -> null
  def writeClose : client -> null
  def read : client -> option(data)

  // Request message builders
  def msgSessionCreate_empty / msgSessionCreate_thisExe / msgSessionCreate_searchExe
  def msgCompileCreate / msgSessionShare / msgSessionDestroy
  def msgSessionExtend_thisExe / msgSessionExtend_sendCode
  def msgAbout / msgCompileModules / msgCompileRunPhrases
  def msgCompileStore / msgCompileStore_sessionless
  def msgViewLoad / msgViewPush / msgViewPop / msgViewReset / msgViewReload
  def msgViewInvoke / msgViewInput / msgIntrospectionSpreadsheet / msgSessionPoll

  // Response extractors
  def sessionOfResponse : data -> sid
  def newSessionOfResponse : data -> sid        // after msgSessionShare
  def executableOfResponse : data -> string     // after msgCompileStore*
  def errorsOfResponse : data -> data
  def cellMessagesOfResponse : data -> data     // [{name, value}]
  def logMessagesOfResponse : data -> data      // [{severity, text}]
module end
```

### Key Concepts

- `createRemote({baseURL, app, token})` — `baseURL` is the amp workspace HTTP URL, e.g. `"https://staging.remixlabs.com/staging/amp"`.
- `close` closes the connection but **does not destroy the session** (sessions time out automatically; prefer `msgSessionDestroy`).
- `writeClose` sends an orderly close; wait for response close to confirm.
- Sessions: one-shot (`msgSessionCreate_thisExe`) or compiler sessions (`msgCompileCreate`).
- `msgCompileStore_sessionless(options, modObj)` — creates compiler session, compiles, stores, destroys in one call.
- `msgViewInvoke(sid, name, env, arg, nextActions)` — invoke an action closure.
- Response `msg_response` has: session ID, errors, cell messages, log messages.
- Full protocol doc: https://docs.google.com/document/d/1Q8DZsrVMceeDQ4hs2RNJP1FQoqlwDhz4kyhkwOVaDQc/

---

## `memo` — Mutable Add/Remove Container

> Source: [memo](https://www.notion.so/f1456110dc364bdb95ec4171395cf94c) (since PR 1020)

```
module memo
  type t(A)
  private case handle(string)   // unique ordered handle (later > earlier)

  def create : null -> t(A)
  def add : t(A) -> A -> handle(string)
  def addBy : (handle(string) -> A) -> t(A) -> [handle(string), A]
  def remove : t(A) -> handle(string) -> null
  def get : t(A) -> handle(string) -> A
  def keys : t(A) -> array(handle(string))
  def values : t(A) -> array(A)
  def bindings : t(A) -> array([handle(string), A])   // since PR1102
module end
```

- Add/remove elements only — **no replacement**. A handle always refers to the same element version.
- `addBy(f, container)` — `f` receives the handle before the element is created (useful for self-referential values).
- Handles are ordered: later-generated handle > earlier-generated handle.

---

## `random` — Random Number & String Generation

> Source: [random](https://www.notion.so/1061d464528f819f9ae1ddb7092be7cc) (since PR 363)

```
module random
  // Predefined alphabets
  def alphabetUpperCase : string   // "ABCDE...Z"
  def alphabetLowerCase : string   // "abcde...z"
  def digits : string              // "0123456789"
  def hexDigits : string           // "0123456789abcdef"
  def alphabetBase64 : string      // upper+lower+digits+"+/"
  def alphabetBase64URL : string   // upper+lower+digits+"-_"

  def pseudoRandomInt : number -> number -> number     // min..max inclusive
  def pseudoRandomFloat : null -> number               // 0..1 exclusive
  def pseudoRandomString : string -> number -> string  // seeded at session start
  def cryptoRandomString : string -> number -> string  // crypto-secure
module end
```

- `pseudoRandomInt(min, max)` — inclusive range.
- `pseudoRandomFloat()` — 0 to 1 exclusive.
- `pseudoRandomString(alphabet, length)` — seeded at session startup (not secure).
- `cryptoRandomString(alphabet, length)` — suitable for session IDs; **not** for long-term secret keys.

---

## `util` — Default & Definedness Helpers

> Source: [util](https://www.notion.so/1061d464528f814eaf75df71f972c907)

```
module util
  def withDefault : A -> A -> A      // x |> withDefault(d)
  def noDefault : A -> A             // x |> noDefault  (error if undefined)
  def isDefined : A -> bool          // same as x? or x !== undefined
  def isUndefined : A -> bool        // same as not x? or x === undefined
  def ifThenElse : bool -> A -> A -> A
module end
```

- `withDefault` and `noDefault` are also available **without importing `util`** (global functions).
- `ifThenElse(c, x, y)` — like `if (c) x else y` but **evaluates both branches** (no shortcut evaluation).
