# Mix Standard Library — I/O Modules

> Sources: [http](https://www.notion.so/1061d464528f81b69f0ef59a0c715d69), [file](https://www.notion.so/1061d464528f8136b486db76ee013016), [binary](https://www.notion.so/1061d464528f81e481afc195d8d6d92b), [crypto](https://www.notion.so/1061d464528f81178aafd26d3c8b51ee), [wasm](https://www.notion.so/1061d464528f81228b5cd5c0597d6a3e), [mime](https://www.notion.so/1061d464528f81348c86faa717c57bbc)

---

## `http` — HTTP Client

```
module http
  type response = data    // {status, body, headers, requested_url} or {error} only
  case textMode | binaryMode
  type mode = textMode | binaryMode

  def delete/get : string -> data -> response
  def patch/post/put : string -> string -> data -> response
  def patchBinary/postBinary/putBinary : string -> binary -> data -> response
  def doRequest : data -> data   // generic: {method, url, body?} in config
  def mediaTypes : map(mode)
  def urlComponentEscape/urlComponentUnescape : string -> string
module end
```

### Config Map

All verb functions accept a final `configMap` (can be null):

| Key                        | Type                       | Description                                                  |
|----------------------------|----------------------------|--------------------------------------------------------------|
| `params`                   | `map`                      | Query parameters (string→string)                             |
| `basicAuth`                | `{username, password}`     | HTTP Basic Auth                                              |
| `oauthToken`               | `string`                   | Sets `Authorization: Bearer <token>`                         |
| `clientTLS`                | `{key, certificate}`       | PEM client TLS cert (not in client VM)                       |
| `headers`                  | `map`                      | Arbitrary headers                                            |
| `timeout`                  | `number`                   | Seconds before failure (not in client VM)                    |
| `sendFrom`                 | `"auto"/"server"/"client"` | Where to execute. Default `"auto"`                           |
| `returnBinaryResponseBody` | `bool`                     | Force body as binary                                         |
| `mediaTypes`               | `map(mode)`                | Content-type → text/binary mode map. Wildcard `*` supported. |

**URL restriction:** Must use `https`. Cannot point to local network. Override: `-dangerous-http-mode`.

**Response:** `{status, body, headers, requested_url}` on success; `{error}` only on hard error (network/timeout). Headers are canonicalized (e.g. `Content-Type`).

**URL escaping:** Escape components first, then assemble — never escape full URLs (double-encoding). Escapes everything except `[-_.~A-Za-z0-9*!()']`.

---

## `file` — Blob File Storage

> Additional undocumented functions exist.

```
module file
  type file = {app, path, kind, metadata, url}
  // kind: "persisted" (disk, {size:int}) or "client" (browser blob URL)

  def get/read/list : option(string) -> string -> result(...)
  def create : appstate -> option(string) -> string -> binary -> result(string, file)
  def delete/rename : appstate -> option(string) -> string -> ... -> result(...)
  def register : appstate -> option(string) -> string -> string -> data -> result(string, file)
module end
```

First arg `option(string)` = DB designator (`null` = current DB).

### `file.register` — Client-Side Upload

`register(appstate, designator, path, clientUrl, metadata)` → returns `file` with `kind:"client"` and `metadata._rmx_upload_url` (signed URL for direct PUT to mixer). Designator: `null` | `some(app)` | `workspace_app(app)` | `remote_app(t_remote_app)`.

After register, client executes `remix/upload-file` Client Action with `{file, data}` to persist. Direct-to-storage upload avoids routing large files through app backend; allows blob preview before committing.

---

## `binary` — Byte Sequence Type

```
module binary
  def empty/length/sub/concat/concatStream/indexOfLeft/byteAt/fromByte

  // Conversions
  def ofString/toString/toStringResult : string <-> binary   // UTF-8
  def ofBase64/ofBase64Result/toBase64 : string <-> binary   // +/ alphabet, = padding
  def ofHex/ofHexResult/toHex : string <-> binary            // lowercase 2-digit hex
  def ofToken : token -> binary
  def formatRmxCode/parseRmxCode : data <-> binary           // internal runtime encoding
module end
```

**Key:** `sub(start, end_excl, b)` no sharing. `toString` throws on invalid UTF-8 (use `toStringResult`). `toBase64` = single line, `+/` alphabet, `=` padding. `ofBase64` liberal (accepts `-_`, optional padding). `indexOfLeft` returns `undefined` if not found.

---

## `crypto` — Hashing & Encryption

```
module crypto
  type binaryLike = string | binary | token

  def md5/sha1/sha256 : binaryLike -> binary
  def sha256_generic : data -> binary              // hashes any data value

  def hmacSha1/hmacSha256 : binaryLike -> binaryLike -> binary        // (key, msg)
  def hmacSha1Verify/hmacSha256Verify : binaryLike -> binaryLike -> binaryLike -> bool  // constant-time

  def rsaSha256 : binaryLike -> binaryLike -> binary   // PKCS#8 PEM key; ABI 0.4+

  def aesGcmEncrypt/aesGcmDecrypt : binaryLike -> binaryLike -> binaryLike -> result(string, binary)
  // (key, additionalData, text/ciphertext); 32-byte key; pass "" for additionalData
  def randomKey32 : null -> binary
module end
```

**Key:** Hash functions return raw bytes — pipe to `binary.toBase64` for string. `rsaSha256` needs PKCS#8 PEM (`BEGIN PRIVATE KEY`); convert PKCS#1 first. AES-GCM: 32-byte key, wrap with `binary.ofBase64`/`toBase64`.

---

## `wasm` — WebAssembly FFI

> **Prefer `mixer wasm` CLI** — generates typed function tiles for the builder.

```
module wasm
  type ffi
  def load : binary -> ffi
  def shutdown : ffi -> null
  def fun1..fun9 : ffi -> string -> type(Args...) -> type(Res) -> (Args... -> Res)

  // WASI
  def wasiLoad : binary -> ffi_wasi
  def wasiCall : ffi_wasi -> string -> map(string) -> array(string) -> [string, string]
  // wasiCall(wasi, stdin, envVars, args) -> [stdout, stderr]
module end
```

**Usage:** `resource.get(null, "m.wasm") |> result.get |> wasm.load` → `wasm.fun2(ffi, "f", type(number), type(bool), type(string))`. Rust functions need `#[remix::bind]`. Load once in code module, not per call. WASI interface enables non-Rust languages.

---

## `mime` — MIME Headers & Form Data

> Since PR1440. Builder example: `remix.remixlabs.com/e/edit/multipart_example/home`

```
module mime
  def formatHeaders/parseHeaders : map(array(string)) <-> string   // (hdr, crlf)
  def formatMediaType : string -> map(string) -> option(string)    // (mainType, params)
  def parseMediaType : string -> option([string, map(string)])

  type formElement = {value: binary, contentType: option(string), filename: option(string)}
  type formData = map(array(formElement))
  def formatFormData : formData -> bool -> [binary, string]        // -> [body, boundary]
  def parseFormData : binary -> string -> option(formData)
module end
```

**Key:** Headers as `map(array(string))` (arrays for repeated fields). `bool` = CR/LF (true) or LF (false); parsers handle both. `formatMediaType("text/plain", {charset:"UTF-8"})` → `"text/plain; charset=UTF-8"`. Supports 3 param encodings (unquoted, quoted, RFC2231 for Unicode). `formatFormData` auto-picks boundary; use in `Content-Type: multipart/form-data; boundary=<boundary>`. Parser is tolerant; returns null on failure.
