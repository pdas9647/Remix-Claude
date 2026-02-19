# Remix Database — Records, Indexing, and Querying

> Notion: [Remix Database](https://www.notion.so/1061d464528f80248ae6cfe395ee6a9c)
> Related: [Queries](https://www.notion.so/1051d464528f80dcb464fae23efefcf2), [Query Syntax](https://www.notion.so/205ca411bf684f5593c3469e39de2af8)

---

## Data Model

- **Linear Record Store**: An append-only, history-preserving store of records.
- **Untyped Records**: Records can contain any serializable Mix `data` (null, bool, number, string, array, map, refs, calendar types).
- **Metadata Fields**:
    - `_rmx_id`: Unique identifier for a record across versions.
    - `_rmx_last_modified_at`: Timestamp of the version.
    - `_rmx_deleted`: Boolean flag for tombstones.
    - `_rmx_previous`: Reference to the previous version of the record.

## Indexing

- **Dynamic & Automatic**: No manual index configuration.
- **Universal Indexing**: Every indexable field is automatically indexed.
- **Indexable Types**: Strings, refs, booleans, `null`, and specific stdlib types (e.g., `calendar` types with `_rmx_index` field).

---

## Querying Logic

Queries are expressed as AST (Abstract Syntax Tree) and executed in layers:
1. **Indexed Filter**: Computed first using the index, yielding database rows.
2. **Record Inspection**: Non-indexed parts of the query are performed on materialized records (lazy loading).
3. **Projections**: Modifications to the record stream (e.g., selecting fields, renaming).
4. **Post-processing**: Arbitrary filters/maps executed in the Mix VM (rather than the DB engine).

### Query Builder 2.0 (QB2)
- Focuses on **Rich Projections** to fetch all data in a single round-trip.
- Removes post-processing from the DB engine for maximum speed (users must handle post-processing in Mix).

---

## Query Syntax (Mix & HTTP)

### Basic Syntax
- `filter | projection`
- Example: `"entity" == "person" | { first_name, last_name, name: first_name + " " + last_name }`

### Performance Tiers
1. **Index Operations**: Comparisons on strings/refs/bools, prefix tests, string contains, `in`/`all in`, boolean combinations.
2. **Engine Operations**: Comparisons on numbers, lists, maps, arbitrary case types.
3. **Mix VM Operations**: Post-processing (`db.postprocess`), fallbacks (`db.filterWithFallback`).

---

## History & Deletions

- **Superseded Records**: Old versions of a record (same `_rmx_id`).
- **Tombstones**: Records marking deletion.
- **Including History**:
    - HTTP: `includeHistory=true`, `includeDeleted=true`.
    - Mix: `db.all()`, `db.includeSuperseded(true)`, `db.includeDeleted(true)`.
- **Querying Tombstones**: Since tombstones lack content, use `_rmx_id` or follow `_rmx_previous`.
    - Example: `db.filter(._rmx_deleted == true && ._rmx_previous.entity == "person")`
