# Remix DBP — Layout, Footers, Admin & Assembly Tutorial

> Parent: [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9)
> Ancestor: Remix Documentation > Remix Catalogs and Toolkits
> See also: [remix-dbp-media.md](./remix-dbp-media.md), [remix-dbp-components.md](./remix-dbp-components.md)

---

## Layout

> Source: [Layout](https://www.notion.so/ba77f5d722d143298f3b10a28a0acdd8)
> Developer: riju@remixlabs.com

Screen layout templates for About Us clips. Two variants:

| Variant                       | Description                                        |
|-------------------------------|----------------------------------------------------|
| **Default layout**            | For screens requiring a clear header + back button |
| **Splash/home screen layout** | For screens with a banner                          |

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/modals`
**Release:** R1.0 — 21 Oct 2024

---

## Basic Info Data Tile

> Source: [Basic info data tile](https://www.notion.so/0b4f7aec91804465b8228290035bfd44)
> Developer: riju@remixlabs.com

Data tile for centralizing About Us clip information. Add to the Constants screen, connect outbinding to the constant, then use "bind a constant value" in in-bindings of display assets. Configurable
via configurator or in-bindings.

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/data`
**Release:** R1.0 — 21 Oct 2024

---

## Footers

> Source: [Footers](https://www.notion.so/bcb639d07b4c4f17922c11d077714f09)

Container page for footer components. All in design library at `remix-india-beta.remixlabs.com/e/edit/about_us/footers`.

### Simple Footers

> Source: [Simple Footers](https://www.notion.so/3595525273514f18891ae5bab011c1d6)
> Developers: arvind@remixlabs.com, riju@remixlabs.com

Recommended for splash screens. Three variants:

| Variant                      | Description           |
|------------------------------|-----------------------|
| **Simple footer**            | Basic footer          |
| **Simple footer with image** | Footer with an image  |
| **Powered by Remix**         | Remix branding footer |

**Dependencies:** None

| Release | Date        | Notes                         |
|---------|-------------|-------------------------------|
| R1.0    | 02 Sep 2024 | First release                 |
| R1.1    | 09 Sep 2024 | Added powered by Remix footer |

### Bottom Navigation Bar

> Source: [Bottom navigation bar](https://www.notion.so/d047130659ba4dab8cf1a759c1b1d7db)
> Developer: riju@remixlabs.com

Screen navigation bar for switching screens in clips/.remix files. Configure via table in configurator: `value` (screen name), `label` (display name), `icon`. Customizable primary color. Automatically
highlights current screen if screen name matches table entry. Override with `selected_tab` in-binding.

**Dependencies:** Target screens must exist before configuring
**Release:** R1.0 — 21 Oct 2024

---

## Admin Contact Details

> Source: [Admin Contact details](https://www.notion.so/d4209d1c03534a86ae037b5a0bfc2e9d)

Container page for admin-side contact management assets. All in design library at `remix-india-beta.remixlabs.com/e/edit/about_us/lists`.

### Contact Submission List

> Source: [Contact Submission list](https://www.notion.so/d8c09a0c58174efeb19e1db9c454ba8b)
> Developer: riju@remixlabs.com

Admin asset showing the list of contact form submissions from appclips. Configure: detail screen name (screen to navigate for full details), color.

**Dependencies:** Cloud agent setup required; agent details must be in admin project constants. Follow the [Assemble About Us](https://www.notion.so/b2ad31b63fe8415382958c4e14b08b8e) guide.
**Release:** R1.0 — 21 Oct 2024

### Personal Detail Card

> Source: [Personal Detail card](https://www.notion.so/115057feedc84691ab67df99c0d9ce52)
> Developer: riju@remixlabs.com

Admin asset showing full details of a single contact submission. The screen hosting this asset must have **`ref_id`** in its In Params, connected to the asset's `ref_id` in-binding. Color customizable
via configurator.

**Dependencies:** Requires Contact Submission List asset
**Release:** R1.0 — 21 Oct 2024

---

## Beacon Setup

> Source: [Beacon setup](https://www.notion.so/ef2d07acb85140139ac8925087e571ad)

Container page for analytics beacon assets. Tracks appclip statistics (installs, screen opens, conversion rates).

### Beacon

> Source: [Beacon](https://www.notion.so/2cd954d02a8d41f49bcbd64790199fb7)
> Developers: john@remixlabs.com, riju@remixlabs.com

Stats beacons for tracking appclip usage. Just add to a screen to start tracking. Two variants:

| Variant                | Description                |
|------------------------|----------------------------|
| **Home screen beacon** | For the home/splash screen |
| **Default beacon**     | For all other screens      |

**Dependencies:** Cloud agent setup required; agent details must be in project constants
**Design library:** `https://remix-india-beta.remixlabs.com/e/edit/about_us/beacon`
**Release:** R1.0 — 20 Sep 2024

### Fetch Beacon Data

> Source: [Fetch beacon data](https://www.notion.so/606a224c224342f0bb1c974ad4ecf806)
> Developer: riju@remixlabs.com

Admin asset for visualizing all beacon data/statistics.

**Dependencies:** Cloud agent setup required; agent details must be in admin project constants
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/data_visualizer`
**Release:** R1.0 — 21 Oct 2024

---

## Assemble About Us — Tutorial

> Source: [Assemble About Us](https://www.notion.so/b2ad31b63fe8415382958c4e14b08b8e)
> Catalog: `remix-india-beta.remixlabs.com/e/`

### Workspace Setup

Add cloud agents based on features needed, then publish and deploy to workspace:

| Feature      | Agents                                                                                                           |
|--------------|------------------------------------------------------------------------------------------------------------------|
| Contact form | `cloud_save`, `cloud_delete`, `cloud_query`, `cloud_record_info`, `cloud_save_multiple`, `cloud_delete_multiple` |
| Beacon       | `get_events`, `log`                                                                                              |

**Important:** `cloud_save`, `cloud_save_multiple`, `log` → set to **anonymous access** (required for appclip).

### Constants Setup

- Add `constants` field from design library to the Constants screen
- **Optional:** Add `basic_info` data tile from About Us design library to Constants screen — pre-populates basic info for smoother assembly

### Screen Assembly

**Beacons:** Add `home beacon` to home screen, `beacon` to all other screens.

**Home screen:**

1. Search "Banner" in Find → select from About Us section
2. Search "Button" (e.g. icon button with share clip option)
3. Search "Footer" → add to flow
4. Set in-bindings (title, icon, color, clip URL for share button, etc.)
5. Navigation: use "Push to Screen"; video navigation: add screen name to `button_navigation_screen` binding of banner

**Video screen:**

- "Embedded video player" for YouTube/Vimeo; "Remix video player" for workspace-hosted files
- Add thumbnail and video URL in-bindings
- Add "Modal" overlay from About Us section → connect video → connect modal to screen

**Contact screen:**

- Add Profile banner (e.g. with LinkedIn option) → set LinkedIn URL in-binding
- Add email/phone button → set email address
- Optional: add Button for Contact Us form navigation
- Add Footer

**Contact Us Form screen:**

- Add "Lead form" or "Simple contact form"

### Publishing

Publish project → Host on GCS → create Appclip if required.
