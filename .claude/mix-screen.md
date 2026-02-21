# Mix Special Screens

> Sources:
> - [_rmx_auth](https://www.notion.so/1821d464528f802a877be36fb7b69fdb)
> - [screen-rmx-error](https://www.notion.so/1061d464528f813a889ae582452bbf64)
> - [screen-rmx-entry](https://www.notion.so/1061d464528f813792f3c132856512a5)
> - [screen-rmx-debugShell](https://www.notion.so/1061d464528f81679c6bca4f882f19dd)
    > Parent: The Mix Programming Language

Special built-in screen names used by the Remix runtime for auth, error handling, entry interception, and debug.

---

## `_rmx_auth` — Authentication Screen

Navigated to automatically when:

- The target screen X has metadata `{ authentication: { enable: true } }`, AND
- `_rmx_auth` itself does NOT have that metadata, AND
- The user is not signed in.

Builder auto-adds an invisible `_rmx_auth` to any app that doesn't define its own. Toggle auth requirement via screen context menu: **"make anon"** / **"require auth"**.

**Module signature:**

```
module _rmx_auth(p:data, t:data)
...
module end
```

**Parameters:**

- `p.reqName` — name of the originally requested screen X
- `p.reqParams` — array of module parameters for the request (normally `[paramObj, transitionObj]`)
- `p.authRequest` — the `authentication` object from X's metadata (always `{ enable: true, ... }`)
- `t` — set to empty object `{}`

**Responsibility:** Sign in the user (use "Trigger User Sign-In" action), then navigate back to X with
`p.reqParams`. After sign-in, `env.token()`, `env.user()`, etc. return values for the authenticated user.

See also: [Build your own screen](https://www.notion.so/1741d464528f808cbb5ff371d571c437)

---

## `_rmx_error` / `_rmx_errorFallback` — Error Screens

Pushed when a Mix runtime error occurs. **Not enabled by default** — client must set client variable
`enable_error_page=true`. Auto-disabled in agents and builder REPL.

- `_rmx_error` — user-defined; takes precedence if present. Must be a module (not slate).
- `_rmx_errorFallback` — stdlib built-in; always defined as fallback.

**Opt-out per client:**

- `client_no_rmx_error=true` — disables `_rmx_error`
- `client_no_rmx_errorFallback=true` — disables `_rmx_errorFallback`

**Parameters:**

- `message` — string error message
- `details` — object with:
    - `code` — string error code (often `""`)
    - `message` — error message (repeated)
    - `mixStack` — Mix backtrace as `array` of tuples
    - `goStack` — Go backtrace as `array` of strings

**Recovery patterns:**

- Pop/go back — reveals failing screen as-is
- `viewstack.switchCaller(1, viewstack.getRequest(1))` — recreate failing view with same params
- `viewstack.load(viewstack.getRequest(viewstack.length()-1))` — restart entire app with same params

---

## `_rmx_entry` — External Request Interceptor

When defined, **all external screen requests** are first directed to `_rmx_entry`. Does **not** intercept
agent invocations or internal `viewstack` pushes/switches.

**Module signature:**

```
module _rmx_entry(p:data, t:data)
...
module end
```

**Parameters:**

- `p.reqName` — name of the requested screen
- `p.reqParams` — array of module parameters (normally `[paramObj, transitionObj]`)
- `t` — set to empty object `{}`

The module must switch to the requested screen or a best replacement.

**Programmatic redirect to `_rmx_entry`** (since PR1404):

```
let req =
  viewstack.dynamicAppView(name, [p1, p2])
  |> viewstack.entryRedirection;
viewstack.push(req)
```

Creates a request for `_rmx_entry` wrapping the original request. Only covers new-session external
requests; internal pushes/switches ignore `_rmx_entry`.

---

## `_rmx_debugShell` — Debug Session Starter

*(Since PR 1160.)* A no-op screen used only to start a debug session (e.g. for test automation).
Do not use in user projects.
