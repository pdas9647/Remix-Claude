# Mix Standard Library — Platform Modules: Runtime & Identity

> Sources:
> - [agent](https://www.notion.so/1821d464528f8084aa32e0673c64255c)
> - [auth](https://www.notion.so/1061d464528f815482dbfd6f695f4947)
> - [env](https://www.notion.so/1061d464528f81099a21eb0df302940c)
> - [secrets](https://www.notion.so/1061d464528f8147ae05db232fdd6db9)
>
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

See also: [mix-lang-platform-concurrency.md](./mix-lang-platform-concurrency.md) | [mix-lang-platform-messaging.md](./mix-lang-platform-messaging.md)

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