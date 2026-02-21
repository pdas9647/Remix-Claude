# Mix Special Agents

> Sources:
> - [agent-sessionAgent](https://www.notion.so/1061d464528f817d9ae0db9dda6c6b0f)
> - [agent-agentAgent](https://www.notion.so/1061d464528f8183bb27d729af8358b4)
> - [agent-makeAgent](https://www.notion.so/1f11d464528f80d7b7fbfba55aecd426)
> Parent: The Mix Programming Language

Special built-in agents bundled in `static-agents.mixexe` at the amp endpoint
`/x/rcm/lib/wasm32-unknown-unknown/static-agents.mixexe`.

---

## `_rmx_sessionAgent` — Compiler + VM Session Driver

Drives the Mix compiler and feeds a "target" VM with compiled binaries, creating an interactive
session. The agent VM spawns a second "target" VM for user code execution. Currently client-side only.

### Parameters

| Param | Type | Notes |
|---|---|---|
| `app` | string | Name of the target app to run in the target VM |
| `compilerEnabled` | bool | If false, compiler is disabled; `executable` param must be set |
| `compilerOptions` | map | `lib_create`, `lib_name`, `lib_version`, `lang_exposeInternals`, `lang_warnings`, `lang_infos`, `link_static` |
| `database` | map | `{type:"rmx_remote", url, db}` or `{type:"rmx", scope:"global", db}` |
| `variant` | string | `""` for QCode, `"WASM"` for Wasm |
| `fwdOutput` | bool | Forward target VM output to agent VM (for debugging) |
| `extraEnv` | map | Extra env vars for target VM (inherits agent env by default; `isAgent` set to false) |
| `executable` | string | `""` = none, `"AUTO"` = search DB, or `_rmx_id` of executable record |
| `localAgentsConfig` | bool/string | `false` = no agentAgent, `true` = start new one, `"<topic>"` = use existing |
| `onInitEcho` | string | `_rmx_echo` string included in response when payload VM initialises |

### Output

- `stdlibId` — stdlib ID used by the compiler (only if `compilerEnabled`)
- `session` — session ID
- `sessionTopic` — MQTT topic for session control messages
- `machine` — target VM ID
- `machineTopic` — MQTT topic for target VM output

### Lifetime

Agent responds after ~2–3 seconds and prints an idle action:
```
$idle_actions = [ { _rmx_type: "msg_view_invoke", name: "...", env: "..." } ]
```
This action **must be invoked** to make the agent fully responsive. The agent and target VM run until
explicitly terminated. Target VM is auto-stopped when the agent VM terminates.

### Sending Requests

Non-retained messages on `<sessionTopic>/control`:

**`msg_compile_runPhrases`:**
```
{ _rmx_type: "msg_compile_runPhrases",
  phrases: [ "phrase1", "phrase2", ... ]
}
```

### Receiving Responses

Published on `<machineTopic>/output`:
```
{ _rmx_type: "msg_response",
  _rmx_echo: "...",           // if requested
  cellMessages: [ { name, version, value }, ... ],
  logMessages: [ { severity: "info"|"warning"|"error", text: "..." }, ... ],
  machine, session, org, workspace,
  responseTo: "name of last request",
  warnings: [ issue, ... ],  // may be omitted
  errors: issue               // if an error occurred
}
```

**`issue` format** (shared by sessionAgent, makeAgent):
```
{ code: string, class: string, message: string,
  params: map(data),
  location: [ file, start_line, start_col, end_line, end_col ]
}
```

---

## `_rmx_agentAgent` — On-Demand Agent VM Spawner

Spawns sub-VMs running agents on demand. Normally used indirectly — sessionAgent starts it when
`localAgentsConfig` is configured.

### Parameters

| Param | Notes |
|---|---|
| `app` | Target app |
| `localAgentsConfig` | Topic path where to install the service |
| `database` | Remote DB URL (optional) |
| `extraEnv` | Extra env passed to spawned agents |

### Output

None. Prints an idle action:
```
{ _rmx_type: "msg_vm_invoke", name: ..., env: ... }
```
This action must be called after the agentAgent returns to install the service.

---

## `_rmx_makeAgent` — Build Agent

Builds Mix binaries from source code. Replacement for amp's `build` request type. Also included in
`static-agents.mixexe`. Can build libs, main exe, standalone exes, and optionally pack + deploy a
`.remix` file — all in one request.

### General Parameters

| Param | Type | Notes |
|---|---|---|
| `compilerOptions` | map | `lang_exposeInternals`, `lang_warnings`, `lang_infos`, `link_static` |
| `variant` | string | `""` QCode (default), `"WASM"` Wasm |
| `targetLibs` | array | `[]` none, `["*"]` all, `["lib1",...]` named libs |
| `targetExe` | array | `[]` none, `["*"]` all libs, `["lib1",...]` specific libs |
| `targetStandaloneExes` | array | Each entry builds a standalone exe linked with the same-named lib |
| `extraLibs` | array(string) | Extra lib IDs to fetch from host rcm dir before building |
| `stopOnFirstError` | bool | Default: false (continue as far as possible) |
| `clean` | bool | Delete all rules/libs/exes from DB before building |

### Data Source (one required)

- **`database`** — `{type:"rmx", scope:"global", db}` or `{type:"rmx_remote", url, db}` or `{..., token}` — source + output
- **`filestore`** — `{app, prefix, url?, token?}` — source from filesystem, binaries written back
- **`sources`** — `array({name, library, loadAlways: bool, code: string})` — literal source code; binaries NOT written back (use with deploy mode)

### Runtime JSON

- `{}` — not installed, not included in deployment
- `map(data)` — installed as `runtime.json` and included in deployment (if `sources` present: included but not installed)
- `null`/missing — not installed; current DB version included in deployment (if `sources` present: nothing included)

### Deploy Parameters

| Param | Notes |
|---|---|
| `deployMode` | `""` none, `"update"`, `"updateButReplaceCode"` (wipes `/code` first), `"updateExisting"`, `"updateExistingButReplaceCode"`, `"dropAndRecreateApp"` |
| `deployFiles` | Array of file names (from DB/filestore) or `[{path, content: binary}]` |
| `deployRecords` | Array of `_rmx_id` strings (from DB) or record objects |
| `deployAppMeta` | `{}` = exclude, `map` = use this, `null`/missing = take from DB |
| `deployBuilderAssets` | bool — include ALL builder assets (ignores module scope) |
| `deploySources` | bool — include Mix source files |
| `deployToServer` | URL of agent server, or `""` to only create `.remix` without deploying |
| `deployToWorkspace` | Workspace name |
| `deployToApp` | App name in target workspace |
| `storeRemixFile` | bool — leave `.remix` file at `/code/export.remix` in agent's app context |

Note: `storeRemixFile` requires `deployMode` to be set. Set `deployToServer: ""` to get the file
without deploying.

### Outputs

| Field | Notes |
|---|---|
| `logs` | `array({severity: string, text: string})` |
| `warnings` | `array(issue)` |
| `errors` | `map(issue)` — keys: `"lib:"+name`, `"exe"`, `"exe:"+name`, `"other"`. Empty map = no errors. |
| `builtLibs` | `map(string)` — lib name → lib ID |
| `builtExe` | bool |
| `builtStandaloneExes` | `array(string)` |
| `deploymentDone` | bool |
| `remixFile` | file path (when `storeRemixFile: true`) |
