# Mix Standard Library — Platform Modules: Messaging, Data & Loader

> Sources:
> - [calendar](https://www.notion.so/1061d464528f81e581d3ddbb419d8df0)
> - [color](https://www.notion.so/1061d464528f819fa83bd801557d90ca)
> - [messaging](https://www.notion.so/1061d464528f81af9d26dc798e4f52cd)
> - [loader](https://www.notion.so/1061d464528f8128ada6eac228486a19)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-platform-runtime.md](./mix-lang-platform-runtime.md) | [mix-lang-platform-concurrency.md](./mix-lang-platform-concurrency.md)

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

Four spaces, all with double-precision + alpha: **linear RGB** (physical intensity 0..1), **SRGB** (gamma ~2.2, web standard), **HSL** (hue 0..360, sat/light 0..1), **HWB** (hue 0..360, white/black 0..1).

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
