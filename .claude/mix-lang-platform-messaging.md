# Mix Standard Library — Platform: Messaging, Calendar, Color & Loader

>
Sources: [calendar](https://www.notion.so/1061d464528f81e581d3ddbb419d8df0) | [color](https://www.notion.so/1061d464528f819fa83bd801557d90ca) | [messaging](https://www.notion.so/1061d464528f81af9d26dc798e4f52cd) | [loader](https://www.notion.so/1061d464528f8128ada6eac228486a19)
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-platform-runtime.md](./mix-lang-platform-runtime.md) | [mix-lang-platform-concurrency.md](./mix-lang-platform-concurrency.md)

---

## `calendar` — Date, Time & Calendar Arithmetic

**Types:** `t_date` (whole day), `t_dateTime` (date+time), `t_timestamp` (date+time+tz). `t` = all three; `t_withTime` = dateTime|timestamp. `t_time` = time-of-day. `errorMode` =
`failIfBad | undefIfBad | normIfBad` (normIfBad normalises, e.g. Feb 29 -> Mar 1). Comparisons (`==`,`<`,`<=`) within same type only.

**Constructors:** `makeDate(emode,y,m,d)`, `makeDateTime(emode,y,m,d,h,min,s)`, `makeTimestamp(emode,y,m,d,h,min,s,tzOffset,tzName)` — tzOffset precedence; use `toTimestamp(tzName)` on dateTime for
tz-by-name. `reprTimestamp(seconds,tz)`. `makeTime(h,min,s)`. `importDate/DateTime/Timestamp(emode,{...})`, `importTime(data)`.

**Projections:** `year`, `month`, `day`, `hour`, `minute`, `second`, `tzName`, `tzOffset`. **Conversions:** `toDate`, `toDateTime`, `toTimestamp(tzName,t)`, `toTime`, `toString` (ISO-8601),
`toDBString` (UTC), `parse`, `parseResult`, `export` ({year,month,day,...}), `format(strftimePattern,t)`.

**Updaters:** `setDay/Month/Year/Hour/Minute/Second` (failIfBad). `setTimezone(name,ts)` — adapts offset, same second.

**Arithmetic:** `add/sub` + `Days/Months/Years/Hours/Minutes/Seconds`. `addTimes/subTimes -> [t_time,bool]`. Timestamps preserve tzName, adapt offset. **Diff:**
`diffDays/Months/Years/Hours/Minutes/Seconds` — positive = first later. `diffTimes`. **Helpers:** `isLeapYear`, `numDaysOfYear/Month`, `first/lastDayOfMonth/Year`, `dayOfYear`, `daysSinceEpoch`,
`secondsSinceEpoch`, `now()` (UTC), `localNow()` (client tz), `epoch/epochDate/epochDateTime`. `dayRange(d1,d2) -> t_date*`. Timezones: political names, "UTC", "UTC+/-hh:mm".

**Weeks:** `weekUS` (Sun=0..Sat=6), `weekISO` (Mon=1..Sun=7). `weekDay`, `weekInfoOfDate` -> `{weekDay,weekNumber,weekYear,weekKind}`. `dateOfWeekInfo`, `weeksOfYear`, `diffWeeks`, `weekRangeISO` ->
stream. `weekTable(kind,d1,d2)` -> array of 7-day arrays.

**Periods:** `makePeriod(n,first,anchor)` — n-day recurring. `periodInfoOfDate` -> `{day,year,iteration,iterationOfYear}`. `periodDayOfDate`, `dateOfPeriodInfo`, `first/lastDayOfPeriod`,
`succPeriodInfo`, `periodRange` -> stream.

**Quarters:** `makeQuarter(startMonth)` / `naturalQuarter`. `quarterInfoOfDate` -> `{day,quarter,year,iteration}`. `quarterOfDate`, `first/lastDayOfQuarter`, `numDaysOfQuarter`, `dateOfQuarterInfo`,
`succQuarterInfo`, `diffQuarters`, `quarterRange` -> stream.

---

## `color` — Color Spaces & Manipulation (PR#1085)

**Spaces** (double-precision + alpha): **Linear RGB** (physical 0..1), **SRGB** (gamma ~2.2, web), **HSL** (hue 0..360, sat/light 0..1), **HWB** (hue 0..360, white/black 0..1).

**Create:** `SRGB_unit/SRGBA_unit` (0..1), `SRGB_8bit/SRGBA_8bit` (0..255), `LinRGB/LinRGBA`, `HSL/HSLA`, `HWB/HWBA`. **Import/Export:** `import/exportSRGB_unit/8bit/LinRGB/HSL/HWB` — from/to records.
**Convert:** `toSRGB/LinRGB/HSL/HWB`. **Parse/Print:** `parseHex` (3/4/6/8 hex), `formatHex`, `formatCSS`.

**Mixing:** `add` (additive), `sub` (subtractive), `blend(c1,c2,p1)`, `blendMany(array([color,weight]))`. Result in c1's space/alpha.

**Brightness:** `lighter/darker(n,c)`, `updateLightness(f,c)` (HSL). `addWhite/subWhite/addBlack/subBlack/updateWhiteness/updateBlackness` (HWB). Auto-converts cross-space. `luminance` 0..1,
`setLuminance`, `perceivedLightness` L* 0..100, `blackAndWhite`, `contrastRatio` WCAG (PR#1107). **Alpha:** `alpha(c)`, `updateAlpha(f,c)`.

---

## `messaging` — High-Level Pub/Sub

Module name is `messaging`. Topics map to MQTT paths internally.

**Topics** (scope hierarchy): `viewTopic` (screen-local), `sessionTopic`, `appTopic` (any session), `userTopic` (user+app), `wsUserTopic` (user, any app), `wsTopic` (all auth users), `mixEvTopic` (
stdlib events, e.g. "db"). `topicByName(type,key)`. Path rules: no leading/trailing `/`, no `//`, no `+`/`#`.

**Retained** (`set`/`get`): last msg stored in broker; new subscribers get it. **Non-retained** (`send`): only live subscribers see it.

**API:** `set(appstate,topic,data)`, `get(topic)`, `send(appstate,topic,data)`. `subscribe(topic,defaultVal)` -> `co.subscription`. `subscribeFG` — foreground, blocks; one per topic; end with `close`.
`unsubscribe(sub)`, `close(topic)`. Streams: `co.pubsubGetStream(sub)` (retained), `co.pubsubRecvStream(sub)` (non-retained).

**Live queries:** `subscribeToQuery(query, {watchOtherSessions, sessionScope})` — emits `true` on result-set change.

**Remote orgs (PR#1351):** `mountOrg(name, {url,user,token})` for session. `remoteWs/WsUser/App/UserTopic({org,ws,[app,][user,]key})`.

**Cloud topics (PR#1860):** published from cloud agents, subscribed from clients. User: short `$cloudAppUser/<path>`, abs `/cloud/<host>/ws/<ws>/app/<app>/user/<user>/<path>` (auth). Shared:
`$cloudAppShared/...` abs `.../shared/application/<path>` (any auth). Public: `$cloudAppPublic/...` abs `.../public/application/<path>` (no auth). Constructors:
`cloudApp[User|Shared|Public]Topic(key)`, `remoteCloudApp[User|Shared|Public]Topic({host,ws,app,[user,]key})`.

---

## `loader` — Dynamic Library Loading (PR#1213)

`loader.load(appstate, sources, libraries, exposedObjects, options) -> result(string, null)`

**Sources:** DB refs (`{type:"rmx", scope:"global", db:"name"}`). `{type:"loaded"}` = pin already-loaded. Only regular global DBs. **Libraries:** by name (newest) or full ID (exact). Deps
auto-resolved. Multiple versions coexist (except stdlib). Modules in `reflect.modules()` prefixed by library ID.

**Exposed objects:** `[type, name, option]`. Types: `global.modScreen/modAgent/modComponent`. `"*"` = all. Options: `null`, `exposeAs(newName)`, `exposeWithDifferentPrefix([old,new])`. Same-named =
overridden. Flat namespace for screens/agents/components.

**Screen meta (PR#1406):** `_meta_ { loadComponents: [[sources, libs, names, opts], ...] }`. Screens only; must list all components recursively. Only components (not agents). Names: strings or
`{name, exposeAs}`.

**Dynamic import (PR#1386):** requires `_dynamicExport_` directive. `_dynamicImport_(modSpace, name, type(P1->...->{ field:Type }))` — modSpace: `modPhysical` (by "libID.ModName") or `modComponent` (
by exposed name). `_dynamicComponentImport_(name, {inputs:[displayNames], outputs:[cellNames]})`.

**Debug:** `reflect.units/modules/screens/agents/components`.