# Tech Debt & Known Architectural Limitations

> Notion: [Tech debt notes](https://www.notion.so/11f1d464528f80e09035cac4bb91dc0a)
> Parent: Engineering process > Platform Engineering (Internal) Home

Major long-standing mis-features, load-bearing bugs, and historically-justified but now-awkward designs that are expected to change.

---

## Database / VM: Refs vs. Strings

- Refs have been inconsistently treated as strings vs. their own type since early DB code
- Currently refs are their own type in the VM and Mix JSON/Msgpack encodings, but strings are sometimes accepted in input
- No clean exposure of the ref type in the Mix language
- Tracking: `amp#2328`, `groovebox#304`

## Identity: Emails vs. Federated Identity

- Tokens currently use email as the primary identifier
- Plan: tokens should contain a unique ID only; login info accessible from auth server but not the primary identifier
- This will change user metadata exposed in the Mix environment and DB metadata fields
- Tracking: `amp#2038`

## Builder: No Proper SDLC

- No diff support in the builder
- No merge semantics across different edits of the same app
- Hard to understand change histories, share edits, or review updates before applying

## Deployment: K8s Is Overkill for amp/mixer

- Main value from K8s is easily resizable disks via Persistent Volume Claims
- Now available from EBS volumes in AWS ECS (likely a better fit)
- No cluster value: nothing has more than one node, nodes from different environments don't talk to each other

## Compiler: No AST

- Parser directly emits expressions (no intermediate AST representation)
- Blocks: pretty printing Mix code, code transformations, calling the parser from Mix
- Fix: introduce a separate AST that corresponds to syntax

## Compiler: Incomplete Type Checker

- `+` operator badly typed (PR ~90% complete: `protoquery#1634`)
- Recursive data types not supported (needs redo: `protoquery#930`)
- Better error guidance needed (`protoquery#1814`)

## Compiler: Wasm Backend Needs Rewrite

- Current backend is a proof-of-concept spike; usually not faster than QCode due to simplistic translation
- Cannot emit a wasm library (only standalone executable)
- Plan documented in `protoquery#1797`

## mix-rs Known Issues

- Need to replace `wasm3` runtime
- Need to save FS after write in mixcore
- Tracking: `mix-rs` issues tagged `techdept`

## amp: Crude User/Permission Management

- Only global admins can modify permissions
- No explicit non-admin API to create users or assign permissions
- No way to manage permissions for your own apps without global admin access
- Root cause: permissions are saved directly on user records in admin DB with no management API; the frontend settings app just reads/writes admin app records directly
