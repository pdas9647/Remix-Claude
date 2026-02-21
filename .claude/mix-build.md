# Mix Build — Libraries & Executables

> Source: [Libraries-and-executables](https://www.notion.so/1061d464528f81edb8f2f39d035c38f7)
> Parent: [The Mix Programming Language](https://www.notion.so/259ede6504e34505982dde5dc4b63d10)

---

## Library vs Executable

Both are compiled Mix artifacts stored as DB records. Distinguished by `_rmx_type`:

- `"library"` — reusable compiled module set
- `"executable"` — runnable program (has `isexe` flag in header)

**Library identity**: `name + version (timestamp) + content_hash`. Hash changes when content changes.

**Executable convention**: when referenced generically, name=`"$executable"`, version=`0`, hash=`"x"`.

---

## Library DB Record

```json
{
  "_rmx_type": "library",
  "name": "<name>",
  "lib_version": 20180308123401,
  "lib_hash": "<hash>",
  "lib_stdlib": "<name+version+hash of stdlib>",
  "lib_header": "<JSON compile metadata>",
  "lib_modules": ["mod1", "mod2"],
  "dataRef": { "_rmx_type": "blob", "data": "<binary bytecode>" },
  "project": "{ref}<project-ID>"    // only if part of a project
}
```

- `lib_stdlib` — identifies compatible stdlib; compiler checks this for compatibility
- `lib_header` — subset needed for compiling against the lib (module list, accessible exports); much shorter than `data`; loaded by compiler before compilation begins
- `dataRef.data` — full bytecode; only loaded when executing
- `project` field — present only while source code is in the project; omit for third-party libs

## Executable DB Record

```json
{
  "_rmx_type": "executable",
  "name": "<name>",
  "exe_header": "<JSON metadata>",
  "dataRef": { "_rmx_type": "blob", "data": "<binary bytecode>" },
  "project": "{ref}<project-ID>"
}
```

---

## `mixc` CLI Commands

```bash
# Create a library from .mix files in a directory:
mixc -app myapp -url http://localhost:8001 -mklib mylib -directory dir

# Create an executable from .mix files:
mixc -app myapp -url http://localhost:8001 -directory dir

# List libraries (latest compatible versions):
mixc -app myapp -url http://localhost:8001 -list-libs

# List all libraries including old/incompatible:
mixc -app myapp -url http://localhost:8001 -list-all-libs

# Simulate loading executable (like Linux ldd):
mixc -app myapp -url http://localhost:8001 -ldd

# Clean project (wipes source + project libs, not third-party libs):
mixc -app myapp -url http://localhost:8001 -clean
```

---

## Dependency Rules

- Every library and executable depends on the **stdlib** (built into `mixc`)
- `lib_stdlib` identifies the stdlib version; updating the compiler → rebuild all libs
- When importing a module not in source: compiler searches all compatible libs in DB
    - All libs with same compiler version are searched
    - Higher version number + same name shadows lower versions
    - Ties are undefined (avoid having the same module in two libs)
- Dependencies are recursive but never circular
- Linking: only libs actually referenced in source are linked into executable
    - Use a bare `import M` (without doing anything with it) to force-link a lib
    - `declare` is **not** sufficient to force-link

---

## Key Behaviors

- **Modules of a linked lib are pushable from AMP** (including indirect prerequisites)
- **No module GC**: linking one module from a lib links all modules in the lib
- **Static linking**: format supports it; `amp` can run statically linked executables; compiler can't generate them yet
- **Updating a lib**: rebuild the lib → rebuild all dependents (hash changes → all users must recompile)
- **Stdlib**: version 1; only incremented for grossly incompatible ABI changes; smaller changes = only hash changes

---

## Track API — Creating Libraries

Via `msg_compile_store` (deprecated — see incremental build doc):

```json
{
  "_rmx_type": "msg_compile_store",
  "_rmx_version": "0.0",
  "sources": { "main": { "data": "<mix code>" } },
  "options": { "lib_create": true, "lib_name": "foo", "lib_version": 20180328121314 }
}
```

On success, `_rmx_id` of new library in the `executable` field of the response.

---

## Inspecting lib_header

To find module params from a library:

- Decode `lib_header` string as JSON
- Look in `provides` array: `[{ modiname: "ModName", modiparams: [{ mpname, mptyp, mpserializable }] }]`
- `mptyp` is the Mix type as string; `mpserializable` = true means data param (vs function)
