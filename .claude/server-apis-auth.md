# Server APIs — Auth Integrations

> Source: [Auth integrations](https://www.notion.so/11f1d464528f8070b010f945a0b0e7af) — Studio Server > Auth integrations
> Parent: Remix Documentation > User Guides and Tutorials > Remix Studio User Guide

---

## Supported Auth Plugin Types

| Type         | Description                                           | PKCE |
|--------------|-------------------------------------------------------|------|
| `oauth`      | Generic OAuth 2.0                                     | Yes  |
| `oidc`       | OpenID Connect                                        | Yes  |
| `apple`      | Sign In With Apple                                    | N/A  |
| `sfdc_oauth` | Salesforce OAuth (special handling — no OIDC support) | Yes  |

## Configuration Schema

```json
{
  "type": "oauth|oidc|apple|sfdc_oauth",
  "name": "<plugin_name>",
  "client_id": "<client_id>",
  "client_secret": "<client_secret>",
  "auth_url": "<authorization_endpoint>",
  "token_url": "<token_endpoint>",
  "userinfo_url": "<userinfo_endpoint>",
  "endpoint": "<oidc_discovery_endpoint>",
  "scopes": [
    "openid",
    "email",
    ...
  ],
  "default": false,
  "use_for_amp_login": false,
  "store_token": false,
  "offline": false,
  "domain": "<auto_attach_domain>",
  "token_extras": [
    "instance_url"
  ],
  "token_lifetime": 0,
  "force_refresh_after": 0,
  "auth_url_params": {
    "prompt": "consent"
  },
  "headers": {
    "user-agent": [
      "com.remixlabs.runtime/1.0"
    ]
  },
  "callback": "<override_callback_url>"
}
```

## Key Configuration Fields

| Field                     | Effect                                                                                                                                                |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `default: true`           | Makes this the default auth flow (`/x/auth` redirects here)                                                                                           |
| `use_for_amp_login: true` | Auth flow generates a Remix token (identity = email from userinfo)                                                                                    |
| `store_token: true`       | Saves 3rd-party token encrypted in DB; accessible via `secrets.getToken(name)` and `/x/auth/{name}/originalToken`                                     |
| `offline: true`           | Requests refresh token; adds `access_type=offline` to auth URL                                                                                        |
| `domain`                  | HTTP requests to this domain auto-attach the stored token if no Authorization header present                                                          |
| `token_extras`            | Fields extracted from OAuth token response and stored alongside (e.g. `instance_url` for Salesforce); accessible via `secrets.tokenExtra(tok, "key")` |
| `token_lifetime`          | Remix token lifetime in seconds (default: 90 days / 7,776,000s)                                                                                       |
| `force_refresh_after`     | Seconds after which to refresh access tokens if no expiration was provided by the IDP                                                                 |

## Callback URLs per Surface

| Surface                | Callback URL Pattern                                   |
|------------------------|--------------------------------------------------------|
| **Web (hosted)**       | `https://<env>.remixlabs.com/a/x/auth/<name>/callback` |
| **Desktop**            | `http://localhost:3434/a/x/auth/<name>/callback`       |
| **Mobile / local dev** | `http://localhost:8000/x/auth/<name>/callback`         |

Some IDPs allow multiple callbacks in one config; others require separate integrations per surface.

## Server API — Auth Config Management

**Public configs** (can be distributed to client apps):

| Method             | Endpoint                      | Access             |
|--------------------|-------------------------------|--------------------|
| `GET`              | `/x/authConfig/public`        | Anyone             |
| `GET`              | `/x/authConfig/public/{name}` | Anyone             |
| `POST`             | `/x/authConfig/public`        | Any signed-in user |
| `PUT/PATCH/DELETE` | `/x/authConfig/public/{name}` | Creator or admin   |

**Confidential configs** (server-only):

| Method                      | Endpoint                 | Access     |
|-----------------------------|--------------------------|------------|
| `GET/POST/PUT/PATCH/DELETE` | `/x/authConfig[/{name}]` | Admin only |

## Auth Flow URLs

| Purpose                 | URL                                                                                |
|-------------------------|------------------------------------------------------------------------------------|
| Default sign-in         | `https://<env>.remixlabs.com/a/x/auth?redirect=<url>`                              |
| Named provider          | `https://<env>.remixlabs.com/a/x/auth/<provider>?redirect=<url>`                   |
| Token exchange          | `GET\|POST /x/auth/{handler}/token` (pass `token` or `idToken` param)              |
| Get 3rd-party token     | `GET\|POST /x/auth/{handler}/originalToken` (authenticated; refreshes if possible) |
| Delete stored token     | `secrets.deleteToken(name)` in Mix                                                 |
| Per-app default sign-in | Set `settings.signin-screen` in app metadata                                       |

## auth0 Setup (for username/password flows)

1. Create auth0 tenant `rmx-<customer>` → create application
2. Add `oidc` plugin config with client ID/secret from auth0 app
3. Optional: Google sign-in via auth0 Google connection (needs GCP OAuth app)
4. Integrate email provider for auth0 emails
5. Enable "Password" Grant Type in auth0 app Advanced Settings for Mix login FFIs
6. Set tenant "Default Directory" to `Username-Password-Authentication`