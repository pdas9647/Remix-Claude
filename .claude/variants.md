# Variants (Config Assets)

> Source: [Variants](https://www.notion.so/1361d464528f801db88efc7b8a4bc063)
> Original design proposal (complete): [Design Proposal - Config assets](https://www.notion.so/12a1d464528f8071ad0ce25f9f53320e)
> Parent: Remix Documentation > User Guides and Tutorials > Topics > Catalog and Collections

---

## Problem

Many catalog assets are just instances of an existing base asset with different binding values. Instead of duplicating the full base asset for each configuration (e.g. repeating a button card for
every color), a single base is kept with multiple configured instances (variants).

## Definitions

- **Base** — The main asset defining the nodes used to create the variant
- **Config** — A set of bindings used to customize the base asset
- **Variant / Instance** — A reference to the base asset with a config

Example: A single picker component (base) can have variants like "month picker" (config: Jan, Feb, ...) or "US state picker" (config: Alabama, Alaska, ...).

## Lifecycle

### Create

- In the sync footer menu of a catalog asset, select `Convert to Variant`
- Visual indicators: yellow indicator on top-right, yellow sync menu (blue = catalog asset, yellow = variant)

### Edit

- Variants are **NOT editable** — no `Edit` footer menu item
- Only bindings (config values) can be modified
- To see/edit the underlying base, use `Get Base` to materialize it into the nodegraph

### Publish

- Variants can be published to a catalog like any other asset
- After publishing, the sync menu shows two locations:
    - **Base location** — where the base asset is published
    - **Variant location** — where the variant is published
- Tooltip on the variant icon in node header also shows both locations

### Publish As...

- Publishes an existing variant to a different catalog/library (like "Save as...")
- Use after modifying a variant you want to publish without changing the original

### Sync (Deep Sync / Dual Sync)

- Base and variants have independent lifecycles
- Syncing a variant does a **Deep Sync**:
    1. Base asset synced — updates the underlying node
    2. Variant synced — updates the binding values (config)
- If a variant is not published (local to nodegraph), only the base needs syncing

## Storage Format (L2SourceNode)

Extends the existing `L2SourceNode` clipboard format with `variant` and `base` fields:

```json
{
  "type": "L2SourceNode",
  "val": {
    "variant": true,
    "source": {
      ...
    },
    "base": {
      ...
    },
    "bindings": {
      ...
    },
    "name": "<optional variant node name>"
  }
}
```

- `variant: true` — signals this is a variant (vs inserting the base component)
- `base` — provenance of the base asset (for dual sync)
- `source` — provenance of the variant config itself
- `bindings` — the config values

## Configurator Integration

Component configurators should create variants when showcasing a component. In the configurator JSON, pass `"variant": true` to insert a variant rather than the base.

## When to Use Variant vs Base Asset

- **Variant** — When the component is used as-is and configured only via bindings. Protects against accidental modification, maintains single source of truth for the underlying component.
- **Base asset** — When the asset is meant to be modified (not fully configurable via bindings alone).

## Preview Considerations (from design proposal)

- Previews for variants are problematic: if base updates, recomputing all variant previews is infeasible (no central registry, variants may be on private/federated servers)
- Externally-generated configs may not have previews in search
- Users can manually update previews: sync the variant, then publish the updated preview back to the catalog
