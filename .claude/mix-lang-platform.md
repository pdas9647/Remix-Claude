# Mix Standard Library — Platform Modules

> Sources:
> - [agent](https://www.notion.so/1821d464528f8084aa32e0673c64255c)
> - [auth](https://www.notion.so/1061d464528f815482dbfd6f695f4947)
> - [memo](https://www.notion.so/f1456110dc364bdb95ec4171395cf94c)
> - [random](https://www.notion.so/1061d464528f819f9ae1ddb7092be7cc)
> - [util](https://www.notion.so/1061d464528f814eaf75df71f972c907)
> - [calendar](https://www.notion.so/1061d464528f81e581d3ddbb419d8df0)
> - [color](https://www.notion.so/1061d464528f819fa83bd801557d90ca)
> - [env](https://www.notion.so/1061d464528f81099a21eb0df302940c)
> - [secrets](https://www.notion.so/1061d464528f8147ae05db232fdd6db9)
> - [logging](https://www.notion.so/1061d464528f81aab0bbc05142788d0d)
> - [metrics](https://www.notion.so/1061d464528f81e8b4b7dafaca82fac2)
> - [co](https://www.notion.so/1061d464528f8116b68ad82588e6b6c4)
> - [embed](https://www.notion.so/1061d464528f814fbda0e472a2642cd0)
> - [track](https://www.notion.so/0d937a3ad10043c597a3f6b3ef39f3f6)
    > Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `agent` — Agent Descriptors & Invocation

```
module agent
  type agent
  type options = map(data)

  def localAgent : string -> t
  def localPersistentAgent : string -> t
  def remoteAgent : string -> string -> string -> t
  def remotePersistentAgent : string -> string -> string -> t

  case currentToken
  case thisToken(data)
  type credentials = currentToken | thisToken
  def asUser : string -> credentials -> t -> t

  def call : t -> options -> data -> result(string, data)
module end
```

### Agent Types

- **One-shot** (`localAgent(name)`, `remoteAgent(baseURL, app, name)`): session created per request, torn down after result. Cannot persist state between calls (use `db` for that).
- **Persistent** (`localPersistentAgent`, `remotePersistentAgent`): session stays alive; the same spreadsheet handles all requests.

### Identity

`asUser(userName, userCreds, descriptor)` — change caller identity. `userCreds` is `currentToken` (pass through current user token) or `thisToken(token)` (explicit token).

### Call Options

- `enforceSessionIsolation: true` — always run in a separate session (not merged with caller's)
- `enforceServerExecution: true` — force run on amp server (since PR1657; also implies isolation)
- `onceKey: some("key")` — cache result per agent+key (local agents only, since PR1657)

### Agent Module Convention (manual coding)

```
module NAME(p:data)
  var cell m_params = p
  ...
  cell output = ...
module end
```

Cells `m_params` and `output` names are fixed.

### Examples

```
// One-shot local agent
agent.call(agent.localAgent("myTest"), {}, {x:"hello"}) |> result.get

// Call as another user
agent.call(agent.localAgent("myTest") |> agent.asUser("myFriend", agent.thisToken(token)), {}, {x:"hello"}) |> result.get

// Remote agent in same workspace, different app
agent.call(agent.remoteAgent(env.baseURL, "otherApp", "myTest"), {}, {x:"hello"}) |> result.get

// Cloud agent on agt-dev server, remix workspace
agent.call(agent.remoteAgent(env.cloudAgentBaseURLFor("agt-dev", "remix"), "myApp", "myAgent"), {}, {x: "hello"}) |> result.get
```

---

## `auth` — Authentication Helpers

```
module auth
  def emptyToken : null -> token             // since PR 1270
  def signIn : string -> string -> string -> result(data, data)
  def signUp : string -> string -> string -> data -> result(data, data)
  def requestPasswordReset : string -> string -> result(data, string)
module end
```

- `emptyToken()` — substitute token when there is none; `string.ofToken` returns `""` for it.
- `signIn(plugin, username, password)` — in-app username/password flow via auth0.
- `signUp(plugin, username, password, userInfo)` — register with optional user info map.
- `requestPasswordReset(plugin, username)` — trigger password reset.
- First arg is always the auth plugin name (typically `"auth0"`).
- **Note:** Redirecting to a web OAuth flow is generally preferred over in-app username+password.

---

## `env` — Environment & URL Construction

```
module env
  // Runtime detection
  type hostEnvironment_t =
    builderEnvironment | serverEnvironment | webEnvironment | widgetEnvironment
    | agentServerEnvironment | unsetEnvironment | unhandledEnvironment(string)
  def platform_version : version_t    // {remix, abi_major, abi_minor, compiler, amp, ...}
  def hostEnvironment : hostEnvironment_t

  // Identity
  def org : string
  def workspace : string
  def user : string
  def userIsVerified : bool
  def userPseudonym : string
  def app : string
  def isBuilder : bool
  def token : null -> data

  // Env variables
  def get : string -> data
  def getWithDefault : data -> string -> data
  def getNumber : string -> number
  def getNumberWithDefault : number -> string -> number
  def defined : string -> bool
  def keys : null -> array(string)
  def initial : map(data)         // env at session start
  def current : null -> map(data) // env now

  // Base URLs
  def backendBaseURL : string          // e.g. https://remix-dev.remixlabs.com/a
  def localBackendBaseURL : string
  def frontendBaseURL : string         // e.g. https://remix-dev.remixlabs.com
  def agentBaseURL : string
  def localAgentBaseURL : string

  // URL builders
  def runtimeURL : string -> string -> string -> data -> string
  def previewURL : string -> string -> string -> data -> string
  def builderURL : string -> string -> string -> string
  def agentURL : string -> string -> string -> string
  def authRedirectURL : string -> string -> string -> string
  def cloudAgentBaseURLFor : string -> string -> string

  // Client properties (browser/widget context)
  def clientVars : null -> map(data)
  def client_height : null -> number
  def client_width : null -> number
  def client_locale : null -> string
  def client_timezone : null -> string
  def client_platform : null -> string
module end
```

### Key Notes

- `get_env()` in user code is **deprecated**; use `env.get` instead.
- `backendBaseURL` vs `client_backendBaseURL`: the former is the standard public URL; the latter is what this particular client is using (may involve proxying). `client_*` variables require a
  connected browser session.
- `cloudAgentBaseURLFor("agt-dev", "remix")` → `"https://agt-dev.remixlabs.com/run-agent/remix"`.
- `runtimeURL(b, f, app, screen, params)` — adds `base` param if non-current backend, appends `params` as query string. In builder context returns preview URL.
- Auth redirect: `env.authRedirectURL(env.backendBaseURL, "myplugin", redirect)`.

---

## `secrets` — Per-User Encrypted Storage

```
module secrets
  foreign def set : appstate -> string -> string -> result(string, string)
  foreign def get : string -> result(string, string)
  foreign def getToken : string -> result(string, token)
  foreign def list : null -> result(string, array(string))
  foreign def delete : appstate -> string -> result(string, string)
  foreign def deleteToken : appstate -> string -> result(string, string)
  foreign def tokenExtra : string -> token -> result(string, data)
module end
```

- Secrets stored per-user with platform encryption (envelope key approach in admin DB).
- **OAuth tokens**: after a third-party sign-in flow with `store_token: true`, token is auto-stored under the auth plugin name. Retrieve with `secrets.getToken("sfdc")`.
- `tokenExtra(key, token)` — access extra fields from OAuth response (e.g. Salesforce `instance_url`).
- Access tokens are auto-refreshed when needed (including refresh token rotation in secrets store).
- `string.ofToken(tok)` always returns a valid access token; raises runtime error if refresh fails.

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

---

## `calendar` — Date, Time & Calendar Arithmetic

> Source: [calendar](https://www.notion.so/1061d464528f81e581d3ddbb419d8df0)

### Types

- `t_date` — whole calendar day
- `t_dateTime` — date + time (no timezone)
- `t_timestamp` — date + time + timezone
- `t` = `t_date | t_dateTime | t_timestamp`; `t_withTime` = `t_dateTime | t_timestamp`
- `t_time` — time-of-day only
- `errorMode` = `failIfBad | undefIfBad | normIfBad` (normIfBad normalises out-of-range values, e.g. Feb 29 → Mar 1)

### Constructors

```
makeDate(emode, year, month, day)
makeDateTime(emode, year, month, day, hour, minute, second)
makeTimestamp(emode, year, month, day, hour, minute, second, tzOffset, tzName)
  // tzOffset takes precedence; use toTimestamp(tzName) on a dateTime to set tz by name only
reprTimestamp(seconds, timezone)   // from Unix seconds
makeTime(hour, minute, second)
importDate/DateTime/Timestamp(emode, {year,month,day,...})
```

### Key Functions

- **Projections**: `year`, `month`, `day`, `hour`, `minute`, `second`, `tzName`, `tzOffset`
- **Conversions**: `toDate`, `toDateTime`, `toTimestamp(tzName, t)`, `toTime`, `toString` (ISO-8601), `toDBString` (UTC), `parse`, `parseResult`, `export` ({year,month,day,...}),
  `format(strftimePattern, t)`
- **Updaters**: `setDay/Month/Year/Hour/Minute/Second`, `setTimezone(name, ts)` — adapts offset to new zone
- **Addition**: `addDays/Months/Years/Hours/Minutes/Seconds/Times`
- **Subtraction**: `subDays/Months/Years/Hours/Minutes/Seconds/Times`
- **Difference**: `diffDays/Months/Years/Hours/Minutes/Seconds` — positive = first arg is later
- **Helpers**: `isLeapYear(y)`, `numDaysOfYear(y)`, `numDaysOfMonth(y, m)`, `firstDayOfMonth/Year`, `lastDayOfMonth/Year`, `dayOfYear`, `daysSinceEpoch`, `secondsSinceEpoch`, `now()` (UTC timestamp),
  `localNow()` (client timezone), `epoch` / `epochDate` / `epochDateTime` (1970-01-01)
- **Day loop**: `dayRange(d1, d2) → t_date*` stream

### Weeks

- Two kinds: `weekUS` (Sun=0..Sat=6), `weekISO` (Mon=1..Sun=7)
- `weekDay(kind, d)`, `weekInfoOfDate(kind, d)` → `{weekDay, weekNumber, weekYear, weekKind}`
- `weekTable(kind, d1, d2)` → `array(array(t_date))` — outer=weeks, inner=7 days
- `weekRangeISO(d1, d2)` → stream of weekInfo; `diffWeeks(kind, d1, d2)`
- US week 1 = week containing Jan 1; ISO week 1 = week of first Thursday

### Periods (generalised weeks)

`makePeriod(n, first, anchor)` — n-day recurring period, numbered `first..first+n-1`, anchored to a date.
`periodInfoOfDate(p, d)` → `{day, year, iteration, iterationOfYear}`; `periodRange(p, d1, d2)` → stream.

### Quarters

`makeQuarter(startMonth)` / `naturalQuarter` (Jan start); `quarterInfoOfDate(q, d)` → `{day, quarter, year, iteration}`; `quarterRange(q, d1, d2)` → stream; `diffQuarters(q, d1, d2)`.

---

## `color` — Color Spaces & Manipulation

> Source: [color](https://www.notion.so/1061d464528f819fa83bd801557d90ca) (since PR 1085)

### Color Spaces

Four spaces, all with double-precision + alpha: **linear RGB** (physical intensity 0..1), **SRGB** (gamma ~2.2, web standard), **HSL** (hue 0..360, sat/light 0..1), **HWB** (hue 0..360, white/black
0..1).

### Create

```
SRGB_unit(r,g,b) / SRGBA_unit(r,g,b,a)    — values 0..1
SRGB_8bit(r,g,b) / SRGBA_8bit(r,g,b,a)    — values 0..255
LinRGB(r,g,b) / LinRGBA(r,g,b,a)           — values 0..1
HSL(h,s,l) / HSLA(h,s,l,a)                 — hue 0..360, rest 0..1
HWB(h,w,b) / HWBA(h,w,b,a)                 — hue 0..360, rest 0..1
```

### Import / Export

- `importSRGB_unit/8bit/LinRGB/HSL/HWB(data)` — from record with named fields
- `exportSRGB_unit/8bit/LinRGB/HSL/HWB(t)` → `{red,green,blue,alpha}` or `{hue,sat,light,alpha}` etc.

### Convert / Parse / Print

- `toSRGB / toLinRGB / toHSL / toHWB` — convert between spaces
- `parseHex(s)` — accepts 3/4/6/8 hex digits → SRGB
- `formatHex(c)` — 6 digits (no alpha) or 8 digits; converts to SRGB first
- `formatCSS(c)` — e.g. `rgb(10% 20% 70%)`; LinRGB → SRGB first

### Mixing

- `add(c1, c2)` — additive mix (result in c1's space, c1's alpha)
- `sub(c1, c2)` — subtractive mix
- `blend(c1, c2, p1)` — weighted average; `p1=0.5` → 50/50
- `blendMany(array([color, weight]))` — multi-color weighted blend

### Brightness & Tone

- `lighter(n, c)` / `darker(n, c)` / `updateLightness(f, c)` — operate in HSL
- `addWhite/subWhite/addBlack/subBlack/updateWhiteness/updateBlackness` — operate in HWB
- `luminance(c)` → relative luminance 0..1 (SRGB)
- `setLuminance(v, c)` — adjust to target luminance
- `perceivedLightness(c)` → L* value 0..100
- `blackAndWhite(c)` → gray with same luminance
- `contrastRatio(c1, c2)` → WCAG contrast ratio

### Alpha

- `alpha(c)` → 0..1; `updateAlpha(f, c)` — transform alpha value

---

## `messaging` — High-Level Pub/Sub Messaging

> Source: [messaging-pubsub](https://www.notion.so/1061d464528f81af9d26dc798e4f52cd)
> Note: stdlib module name is `messaging`, not `messaging-pubsub`.

```
module messaging
  type topic
  // Topic constructors
  def viewTopic : string -> topic        // current screen only (local, no broker)
  def sessionTopic : string -> topic     // current session
  def appTopic : string -> topic         // any session of the current app
  def userTopic : string -> topic        // current user + current app
  def wsUserTopic : string -> topic      // current user, any app in workspace
  def wsTopic : string -> topic          // all authenticated users in workspace
  def mixEvTopic : string -> topic       // stdlib-internal events (e.g. "db")
  def topicByName : string -> string -> topic

  // Retained ("variable-like") messages
  def set : appstate -> topic -> data -> null
  def get : topic -> data

  // Non-retained ("event-like") messages
  def send : appstate -> topic -> data -> null

  // Subscriptions
  def subscribe : topic -> data -> co.subscription(data)
  def subscribeFG : topic -> data -> co.subscription(data)  // foreground, blocks until msg
  def unsubscribe : co.subscription(_) -> null
  def close : topic -> null

  // Live queries
  def subscribeToQuery : db.query -> data -> co.subscription(data)

  // Remote org topics (PR1351)
  def remoteWsTopic : {org, ws, key} -> topic
  def remoteWsUserTopic : {org, ws, user, key} -> topic
  def remoteAppTopic : {org, ws, app, key} -> topic
  def remoteUserTopic : {org, ws, app, user, key} -> topic
  def mountOrg : string -> map(data) -> null

  // Cloud topics (PR1860; publish from cloud agents, subscribe from client apps)
  def cloudAppUserTopic : string -> topic    // short path: $cloudAppUser/<path>
  def cloudAppSharedTopic : string -> topic  // short path: $cloudAppShared/<path>
  def cloudAppPublicTopic : string -> topic  // short path: $cloudAppPublic/<path>
  def remoteCloudAppSharedTopic : {host, ws, app, key} -> topic
  def remoteCloudAppPublicTopic : {host, ws, app, key} -> topic
  def remoteCloudAppUserTopic : {host, ws, app, user, key} -> topic
module end
```

### Retained vs Non-Retained

- **Retained** (`set`/`get`): last message stored in broker; new subscribers receive it immediately. Behaves like a global observable variable.
- **Non-retained** (`send`): broker does not store; only live subscribers receive it. Use for event/protocol messaging.

### Subscribing

- `subscribe(topic, defaultVal)` → `co.subscription` — returns default if nothing published yet.
- `co.pubsubGetStream(sub)` — stream of retained updates; `co.pubsubRecvStream(sub)` — stream of non-retained.
- `subscribeFG(topic, defaultVal)` — foreground subscription; blocks coroutine until messages arrive; max one per topic; must terminate with `close`.
- `co.pubsubUnsubscribe(sub)` or `messaging.unsubscribe(sub)` — stop subscription.

### Live Queries

`subscribeToQuery(q, {watchOtherSessions, sessionScope})`:

- Emits `true` when the query result set changes.
- `watchOtherSessions: true` — also detect changes from other sessions (default: only own session).
- `sessionScope: true` — subscription lives for session (default: view lifetime).

### Remote Orgs

```
messaging.mountOrg("my-org", {url: "wss://remix-dev.remixlabs.com/a/x/broker.ws", user: "...", token: ...})
```

Mounts remote org topics for the session lifetime. Then use `remoteWsTopic` etc. to construct topic paths.

### Cloud Topics

Published from cloud agents using short paths (`$cloudAppUser/<path>`, `$cloudAppShared/<path>`, `$cloudAppPublic/<path>`). Subscribed from client apps using absolute paths:

- `/cloud/<host>/ws/<ws>/app/<app>/user/<user>/<path>` — user-scoped
- `/cloud/<host>/ws/<ws>/app/<app>/shared/application/<path>` — app-shared (any user)
- `/cloud/<host>/ws/<ws>/app/<app>/public/application/<path>` — no auth required

### Topic Path Rules

- No leading/trailing slashes; no `//`; no `+` or `#` characters (MQTT reserved).
- Topics are translated to MQTT paths internally, e.g. `userTopic("foo")` → `/org/staging/ws/staging/app/sample/user/gerd/application/foo`.

---

## `loader` — Dynamic Library Loading

> Source: [loader](https://www.notion.so/1061d464528f8128ada6eac228486a19) (since PR 1213)

```
module loader
  case exposeAs(string)
  case exposeWithDifferentPrefix([string, string])
  type exposeOption = null | exposeAs | exposeWithDifferentPrefix

  def load : appstate ->
             array(data) ->              // sources
             array(string) ->            // libraries
             array([modUsage, string, exposeOption]) ->  // exposedObjects
             map(data) ->                // options
             result(string, null)
module end
```

### Usage

```
loader.load(appstate, sources, libraries, exposedObjects, options) |> result.get
```

### Sources

List of DB references (same format as `db` module):

```
[ {"type": "rmx", "scope": "global", "db": "myapp"} ]
```

- Special `{"type": "loaded"}` — refers to already-loaded libraries (use to pin versions, no newer loading).
- **Only regular global databases** (no in-mem or special DBs) supported currently.

### Libraries

- By name: `"person"` — loads newest version from sources.
- By full ID: `"list-20220405@abcd"` — loads exact version.
- Dependencies resolved automatically.
- Loaded Mix modules appear in `reflect.modules()` (prefixed by library ID) even if not exposed.

### Exposed Objects

List of triples `[type, name, option]`:

- **type**: `global.modScreen`, `global.modAgent`, `global.modComponent`
- **name**: plain name or `"*"` to expose all of that type
- **option**: `null` (keep name) | `exposeAs(newName)` | `exposeWithDifferentPrefix([oldPrefix, newPrefix])`
- If a same-named object already exists, it is **overridden**.
- Exposed screens → navigable + in `reflect.screens()`; agents → invocable + in `reflect.agents()`; components → usable by name + in `reflect.components()`.

### Screen Meta Declarations (PR1406)

Screens can pre-declare components to load in `_meta_`:

```
_meta_ { modType: "screen",
  loadComponents: [
    [ [{type:"rmx", scope:"global", db:"designlib"}],   // sources
      ["formfields", "buttons"],                          // libraries
      ["comp_textField", "comp_OKbutton"],                // exposed component names
      {}                                                  // options
    ]
  ]
}
```

- **Screens only** (not components). Screen must declare all components recursively.
- **Currently only components** can be loaded this way (not agents).

### Dynamic Import (PR1386)

Modules with `_dynamicExport_` directive can be imported at runtime:

```
_dynamicImport_(modSpace, name, type(P1 -> P2 -> {field: Type, ...}))
```

- `modSpace`: `global.modPhysical` (by `"libID.ModName"`) or `global.modComponent` (by exposed name)
- Returns a function; call it to instantiate the module.

Dynamic component import:

```
_dynamicComponentImport_(exposedName, {inputs: ["DisplayName1", ...], outputs: ["cellName1", ...]})
```

Returns `link(data) -> data -> {output: link(data), _content_: link(data)}` (or as declared).

### Scoping & Restrictions

- Multiple Mix module versions can coexist; each version's code uses its own compile-time references.
- Screens/agents/components have a **flat namespace** — only one instance per name allowed.
- Cannot load libraries compiled against a different stdlib version.
- Debugging: `reflect.units`, `reflect.modules`, `reflect.screens`, `reflect.agents`, `reflect.components`.
