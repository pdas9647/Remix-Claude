# Mix Standard Library — I/O Modules

> Sources:
> - [http](https://www.notion.so/1061d464528f81b69f0ef59a0c715d69)
> - [file](https://www.notion.so/1061d464528f8136b486db76ee013016)
> - [binary](https://www.notion.so/1061d464528f81e481afc195d8d6d92b)
> Parent: [The Mix Standard Library](https://www.notion.so/1061d464528f8010b0cfc60836c20290)

---

## `http` — HTTP Client

> Source: [http](https://www.notion.so/1061d464528f81b69f0ef59a0c715d69)

### Module Signature

```
module http
  type response = data    // map with keys "status"+"body", or "error" only

  case textMode
  case binaryMode
  type mode = textMode | binaryMode

  // Verb functions — all take (url, configMap) or (url, body, configMap)
  def delete : string -> data -> response
  def get : string -> data -> response
  def patch : string -> string -> data -> response
  def post : string -> string -> data -> response
  def put : string -> string -> data -> response

  // Binary body variants
  def patchBinary : string -> binary -> data -> response   // since PR1440
  def postBinary : string -> binary -> data -> response    // since PR1440
  def putBinary : string -> binary -> data -> response     // since PR1440

  // Generic
  def doRequest : data -> data   // put "method", "url", optionally "body" in config map

  def mediaTypes : map(mode)     // default mediaTypes setting, since PR1547

  def urlComponentEscape : string -> string
  def urlComponentUnescape : string -> string
module end
```

### Config Map Options

All verb functions accept a final `configMap` (can be `null` / empty):

| Key | Type | Description |
|-----|------|-------------|
| `params` | `map` | Query parameters (string→string). Alternative to inline URL params — choose one. |
| `basicAuth` | `{username, password}` | HTTP Basic Auth |
| `oauthToken` | `string` | Sets `Authorization: Bearer <token>`. Accepts plain string or opaque amp token from `get_env().token`. |
| `clientTLS` | `{key, certificate}` | PEM-format client TLS cert. PKCS1/PKCS8/SEC1 EC supported. Not available in client VM. |
| `headers` | `map` | Arbitrary request headers |
| `timeout` | `number` | Seconds before failure. Timed-out response has `error` key only. Not available in client VM. |
| `sendFrom` | `"auto"` / `"server"` / `"client"` | Where to execute request. Default `"auto"` = same side as running VM. |
| `returnBinaryResponseBody` | `bool` | Force `body` as `binary` type (accepts non-UTF-8). |
| `mediaTypes` | `map(mode)` | Fine-grained content-type → text/binary mode map. Wildcard `*` supported. |

**URL restriction:** Must use `https` scheme. Cannot point to local network names (blocks amp admin port / internal Kubernetes). Override with `-dangerous-http-mode` amp flag.

### Response Map

```
{
  "error": string,          // present ONLY on hard error (network failure, timeout) — no other keys
  "status": number,         // HTTP status code (e.g. 200)
  "body": string | binary,  // string by default; binary if returnBinaryResponseBody or binaryMode
  "headers": map,           // canonicalized: "Content-Type", "Authorization", etc.
  "requested_url": string   // effective URL after query param insertion
}
```

### URL Escaping

`urlComponentEscape` — escapes everything except RFC 3896 unreserved chars (`[-_.~A-Za-z0-9]`) plus `[*!()']`. Escapes `/`, `:`, `+` (as `%2B`), space (as `%20`). **Use on URL pieces, not full URLs.**

`urlComponentUnescape` — reverses percent-encoding.

**Caution:** Escape components first, then assemble with delimiters — never escape an already-assembled URL (causes double-encoding of `%` signs).

---

## `file` — Blob File Storage

> Source: [file](https://www.notion.so/1061d464528f8136b486db76ee013016)
> Note: Additional undocumented functions exist.

### Module Signature

```
module file
  type file =
    { app: string,
      path: string,      // full path from root, starts with "/"
      kind: string,      // "persisted" or "client"
      metadata: data,    // {size: int} for persisted; client-defined for client files
      url: string        // local blob URL or stored file URL
    }

  def get : option(string) -> string -> result(string, file)
  def read : option(string) -> string -> result(string, binary)
  def list : option(string) -> string -> result(string, array(file))

  def create : appstate -> option(string) -> string -> binary -> result(string, file)
  def delete : appstate -> option(string) -> string -> result(string, null)
  def rename : appstate -> option(string) -> string -> string -> result(string, null)

  def register : appstate -> option(string) -> string -> string -> data -> result(string, file)
  // register(appstate, db, path, clientUrl, metadata)
module end
```

First arg is always `option(string)` — the DB designator (`null` = current DB).

### File Kinds

- **`"persisted"`** — stored on disk; metadata includes `{size: int}`; URLs are `https://` (web) or `file://` (native) or `blob:HOSTNAME/UUID` (browser WASI)
- **`"client"`** — held client-side only; generated via `URL.createObjectURL`; used for preview before upload

### `file.register` — Client-Side Upload Flow

```
register(appstate, designator, path, clientUrl, metadata)
```

`designator` type:
```
null                      // default = same as workspace_app(env.app)
| some(app)               // backward-compat, same as workspace_app
| workspace_app(app)
| remote_app(t_remote_app)
```

Returns a `file` struct with `kind: "client"` and `metadata._rmx_upload_url` — a signed URL for the client to PUT the file directly to mixer. Example response:
```json
{
  "app": "cloud-agent",
  "kind": "client",
  "metadata": {
    "_rmx_upload_oauth": true,
    "_rmx_upload_url": "http://localhost:2025/v1/ws/local/app/cloud-agent/file-manager/content?path=%2Ftest.txt"
  },
  "path": "/test.txt",
  "url": "blob:http://localhost:3000/6db0b678-20e4-4513-b5db-c854b8dd8b1d"
}
```

After `register`, client executes `remix/upload-file` Client Action with `{file: file, data: data}` to upload and persist the file.

**Why client-side upload?** Avoids routing large files through the app backend. Allows preview (display blob URL) before committing the upload. Upload goes direct to file storage server.

---

## `binary` — Byte Sequence Type

> Source: [binary](https://www.notion.so/1061d464528f81e481afc195d8d6d92b)

### Module Signature

```
module binary
  // Basics
  def empty : binary                                     // since PR1657
  def length : binary -> number
  def sub : number -> number -> binary -> binary         // sub(startByte, endByte, b)
  def concat : binary -> array(binary) -> binary         // concat(separator, byteArrays)
  def concatStream : binary -> stream(binary) -> binary  // since PR1817; optimized for long streams
  def indexOfLeft : binary -> binary -> number           // indexOfLeft(toSearch, buffer)
  def byteAt : number -> binary -> number                // since PR1817; range 0-255
  def fromByte : number -> binary                        // since PR1817; single-byte binary

  // Conversions
  def ofString : string -> binary          // UTF-8 encode, always succeeds
  def toString : binary -> string          // UTF-8 decode, throws on invalid UTF-8
  def toStringResult : binary -> result(string, string)  // since PR1545; error variant
  def ofBase64 : string -> binary          // accepts +/ and -_ variants; padding optional
  def ofBase64Result : string -> result(string, binary)  // since PR1545
  def toBase64 : binary -> string          // standard Base64 (+/), with = padding, no linefeeds
  def ofHex : string -> binary             // since PR1817; accepts a-f and A-F
  def ofHexResult : string -> result(string, binary)     // since PR1817
  def toHex : binary -> string             // since PR1817; lowercase a-f, 2 digits per byte
  def ofToken : token -> binary            // binary representation of a token
  def formatRmxCode : data -> result(string, binary)     // since PR1581; internal runtime encoding
  def parseRmxCode : binary -> result(string, data)      // since PR1581; parse internal encoding
module end
```

### Key Behaviors

- `sub(startByte, endByte, b)` — returns bytes `[startByte, endByte)` as a new binary (no sharing)
- `concat(sep, [b1, b2, ...])` — joins binaries with separator between each pair
- `indexOfLeft(toSearch, buffer)` — returns start position of leftmost match, or `undefined` if not found
- `toString` throws on invalid UTF-8; use `toStringResult` for safe variant
- `toBase64` — single line, no linefeeds, standard `+/` alphabet, `=` padding
- `ofBase64` — liberal: also accepts `-` for `+`, `_` for `/`, padding optional
- `toHex` / `ofHex` — 2-digit lowercase hex per byte; `ofHex` also accepts uppercase
- `formatRmxCode` / `parseRmxCode` — round-trip any Mix `data` value through internal runtime encoding

---

## `crypto` — Cryptographic Hashing & Encryption

> Source: [crypto](https://www.notion.so/1061d464528f81178aafd26d3c8b51ee)

```
module crypto
  type binaryLike = string | binary | token

  def md5 : binaryLike -> binary
  def sha1 : binaryLike -> binary
  def sha256 : binaryLike -> binary
  def sha256_generic : data -> binary                                    // since PR#1822; hashes any data

  def hmacSha1 : binaryLike -> binaryLike -> binary                     // hmacSha1(key, msg)
  def hmacSha1Verify : binaryLike -> binaryLike -> binaryLike -> bool   // (key, msg, signature)
  def hmacSha256 : binaryLike -> binaryLike -> binary                   // hmacSha256(key, msg)
  def hmacSha256Verify : binaryLike -> binaryLike -> binaryLike -> bool // (key, msg, signature)

  def rsaSha256 : binaryLike -> binaryLike -> binary                    // rsaSha256(key, msg); ABI 0.4+

  def aesGcmEncrypt : binaryLike -> binaryLike -> binaryLike -> result(string, binary) // (key, additionalData, text)
  def aesGcmDecrypt : binaryLike -> binaryLike -> binaryLike -> result(string, binary) // (key, additionalData, ciphertext)
  def randomKey32 : null -> binary
module end
```

- `binaryLike` — accepts `binary`, `string` (treated as UTF-8), or `token` (uses embedded key material if permitted)
- `md5` / `sha1` / `sha256` — return raw bytes; encode to base64 for string representation: `msg |> crypto.sha256 |> binary.toBase64`
- `sha256_generic` — hashes any `data` value (not just binary-like); algorithm is implementation-defined
- `hmacSha256Verify` / `hmacSha1Verify` — constant-time signature verification
- `rsaSha256` — PKCS#8 PEM key (header `-----BEGIN PRIVATE KEY-----`); PKCS#1 (`-----BEGIN RSA PRIVATE KEY-----`) must be converted first
- `aesGcmEncrypt` / `aesGcmDecrypt` — AES-256-GCM symmetric encryption; 32-byte key; pass `""` for `additionalData` unless needed; wrap inputs/outputs with `binary.ofBase64` / `binary.toBase64`
- `randomKey32()` — generates a random 32-byte key suitable for AES-GCM

---

## `wasm` — WebAssembly FFI

> Source: [wasm](https://www.notion.so/1061d464528f81228b5cd5c0597d6a3e)
> **Prefer `mixer wasm` CLI** over direct use of this module — it generates typed function tiles for the builder.

```
module wasm
  type ffi

  def load : binary -> ffi                   // load and instantiate a Wasm module
  def shutdown : ffi -> null                 // free the Wasm instance

  // Retrieve typed functions by name (N = 1..9)
  def fun1 : ffi -> string -> type(Arg1) -> type(Res) -> (Arg1 -> Res)
  def fun2 : ffi -> string -> type(Arg1) -> type(Arg2) -> type(Res) -> (Arg1 -> Arg2 -> Res)
  // ... fun3 through fun9 follow the same pattern with N arg types then result type

  // WASI interface
  def wasiLoad : binary -> ffi_wasi
  def wasiCall : ffi_wasi -> string -> map(string) -> array(string) -> [string, string]
  // wasiCall(wasi, stdin, envVars, args) -> [stdout, stderr]
module end
```

**Usage pattern:**
```
import wasm; import resource; import result
def ffi = resource.get(null, "m.wasm") |> result.get |> wasm.load
def f = wasm.fun2(ffi, "f", type(number), type(bool), type(string))
// f : number -> bool -> string
```

- Wasm functions must be annotated with `#[remix::bind]` in Rust (crate `mix-rs`)
- Load once in a code module, not on every call
- `wasiLoad` / `wasiCall` — WASI stdin/stdout/env/args interface; easier for languages not supported by `mixer`

---

## `mime` — MIME Headers & Form Data

> Source: [mime](https://www.notion.so/1061d464528f81348c86faa717c57bbc) — since PR1440

```
module mime
  // Headers
  def formatHeaders : map(array(string)) -> bool -> string   // (hdr, crlf)
  def parseHeaders : string -> map(array(string))

  def formatMediaType : string -> map(string) -> option(string)      // (mainType, params) -> option(string)
  def parseMediaType : string -> option([string, map(string)])        // -> option([mainType, params])

  // multipart/form-data
  type formElement = { value: binary, contentType: option(string), filename: option(string) }
  type formData = map(array(formElement))
  def formatFormData : formData -> bool -> [binary, string]  // (form, crlf) -> [body, boundary]
  def parseFormData : binary -> string -> option(formData)   // (body, boundary)
module end
```

- `formatHeaders` / `parseHeaders` — MIME-style headers as `map(array(string))` (arrays because fields can repeat). `bool` controls CR/LF vs LF line endings. Parser handles both automatically.
- `formatMediaType("text/plain", {charset: "UTF-8"})` → `"text/plain; charset=UTF-8"`. Handles 3 param encodings: unquoted, quoted, RFC2231 (for Unicode). Returns `null` if invalid.
- `parseMediaType` — returns `some([mainType, params])` or `null`. Field names in lowercase.
- `formatFormData` — picks a boundary automatically; returns `[body, boundary]` pair. Use `boundary` in `Content-Type: multipart/form-data; boundary=<boundary>`.
- `parseFormData` — tolerant parser; ignores malformed parts; returns `null` if nothing could be decoded.
