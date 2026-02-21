# Remix DBP — Digital Business Profile / About Us Catalog

> Parent: [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9)
> Ancestor: Remix Documentation > Remix Catalogs and Toolkits

---

## Overview

The "About Us" catalog provides reusable media and content components for building Digital Business Profile (DBP) websites. All assets are also available in the **About Us design library** on
`remix-india-beta.remixlabs.com/e/edit/about_us/`.

---

## Media Assets

> Source: [Media](https://www.notion.so/500fd31c62d5457ca4044a26782adda5)

Container page for media-related DBP components. Sub-pages: Video Players, List of videos, Image galleries, List of images with text, images.

---

### Video Players

> Source: [Video Players](https://www.notion.so/ec18a532129e4dd6a4f37ae9d445387d)
> Developers: padmanabha@remixlabs.com, riju@remixlabs.com

Two types of video players:

| Player                    | Description                                                               | Dependency                                                |
|---------------------------|---------------------------------------------------------------------------|-----------------------------------------------------------|
| **Remix video player**    | Plays videos stored on website/cloud/Remix workspace. Requires thumbnail. | ~~`rmx-video.js` in Files~~ (no longer needed as of R3.1) |
| **Embedded video player** | Supports YouTube/Vimeo embedded videos. No thumbnail needed.              | None                                                      |

Both have **overlay variations** (full-screen overlay playback).

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/image_and_video`

| Release | Date        | Notes                                          |
|---------|-------------|------------------------------------------------|
| R1.0    | 23 Jul 2024 | First release                                  |
| R1.1    | 04 Sep 2024 | UI consistency improvements                    |
| R2.0    | 09 Sep 2024 | Split into two players (workspace vs embedded) |
| R3.0    | 03 Oct 2024 | Overlay variants added                         |
| R3.1    | 14 Oct 2024 | No resource file dependency needed             |

---

### List of Videos

> Source: [List of videos](https://www.notion.so/a96d7f9e95a545b080d788ff5d40d8e2)
> Developer: riju@remixlabs.com

Video list components — add multiple video links with thumbnails. Two variants:

| Variant                           | Description                      | Dependency                        |
|-----------------------------------|----------------------------------|-----------------------------------|
| **Embedded list of videos**       | YouTube/Vimeo embedded videos    | None                              |
| **List of videos (Remix player)** | Videos stored in Remix workspace | Remix video player (rmx-video.js) |

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`

| Release | Date        | Notes                                                         |
|---------|-------------|---------------------------------------------------------------|
| R1.0    | 13 Sep 2024 | First release                                                 |
| R1.1    | 14 Oct 2024 | No resource file dependency needed                            |
| R1.2    | 15 Oct 2024 | Bug fix: configurator not working if asset already in project |

---

### Image Galleries

> Source: [image galleries](https://www.notion.so/7c82e8be64384df7b6277edec1f14df7)
> Developer: riju@remixlabs.com

Display images as a catalog/gallery. Configurable: image list, images per row (`how_many_per_row` binding), optional full-screen screen on tap.

Three gallery types:

| Type                           | Description                                         |
|--------------------------------|-----------------------------------------------------|
| **Image gallery with hero**    | First image displayed larger (hero), rest in grid   |
| **Image gallery without hero** | All images in uniform grid                          |
| **Single row image gallery**   | One row only, shows up to `how_many_per_row` images |

Also includes **image full screen overlay** variant.

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`

| Release | Date        | Notes                       |
|---------|-------------|-----------------------------|
| R1.0    | 11 Sep 2024 | First release               |
| R2.0    | 04 Oct 2024 | Image overlay variant added |

---

### List of Images with Text

> Source: [List of images with text](https://www.notion.so/d09fa785ff5541df9ab50b74500cc93e)
> Developer: riju@remixlabs.com

Similar to image gallery but includes **two lines of text per image** — suitable for:

- Listing items with names and descriptions
- People listings with profile image, name, and title

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`

| Release | Date        | Notes         |
|---------|-------------|---------------|
| R1.0    | 25 Sep 2024 | First release |

---

### Images

> Source: [images](https://www.notion.so/e1816e3726314bf49950c736b82af61f)
> Developer: riju@remixlabs.com

Two image display assets:

| Asset                  | Description                                   |
|------------------------|-----------------------------------------------|
| **Image full screen**  | Displays an image without crop in full screen |
| **Image with caption** | Displays an image without crop with a caption |

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/image_and_video`
**Release:** R1.0 — 11 Sep 2024

---

## Data Visualization

> Source: [Data visualization](https://www.notion.so/34977c23361e4959bb9761a1fc808715)

Container page for data visualization components. Sub-pages: Quote Component, Percentage Visualizer, Bar Graph. All in design library at
`remix-india-beta.remixlabs.com/e/edit/about_us/data_visualizer`.

### Quote Component

> Source: [Quote Component](https://www.notion.so/8e60cce591b0457baf0a1363290170f5)
> Developer: riju@remixlabs.com

Display card for quotes. In-bindings: quote text, person name, job title/description, color. Usable as single card or list.

**Dependencies:** None

| Release | Date        | Notes             |
|---------|-------------|-------------------|
| R1.0    | 02 Sep 2024 | First release     |
| R1.1    | 04 Sep 2024 | Smaller font size |

### Percentage Visualizer

> Source: [Percentage Visualizer](https://www.notion.so/fd575a58f204424fadcbe26fcee81ecc)
> Developer: riju@remixlabs.com

Displays key stats/results with dynamic styling. In-bindings: value, label, description, increase/decrease toggle. Background color darkens and chevron count increases with higher values. Two design
variants: 1-line and 2-line text. Usable as single card or list.

**Dependencies:** None

| Release | Date        | Notes                                                         |
|---------|-------------|---------------------------------------------------------------|
| R1.0    | 02 Sep 2024 | First release                                                 |
| R2.0    | 09 Sep 2024 | Two designs (1-line/2-line); configurator list support fixed  |
| R2.1    | 15 Oct 2024 | Bug fix: configurator not working if asset already in project |

### Bar Graph

> Source: [Bar Graph](https://www.notion.so/fcd1254f1b464ecd9803b8dc8a7f593a)
> Developer: riju@remixlabs.com

Configurable horizontal bar graph. In-bindings: values, labels, optional color scheme (default: blue/primary), percentage toggle, total (0 = sum of values), optional unit.

**Dependencies:** None
**Release:** R1.0 — 11 Sep 2024

---

## Buttons

> Source: [Buttons](https://www.notion.so/a9628c8c35ac4026a77cccd7bfebd3fa)

Container page for button components. All in design library at `remix-india-beta.remixlabs.com/e/edit/about_us/buttons`.

### Complex/Multi Buttons

> Source: [Complex/multi buttons](https://www.notion.so/ad588615b7b64d439d68c54605443975)
> Developer: riju@remixlabs.com

Three variants with header text and additional elements:

| Variant                                         | Description                                                         |
|-------------------------------------------------|---------------------------------------------------------------------|
| **Icon button with share clip option + header** | Header + icon button (customizable color/icon) + share appclip URL  |
| **3 icon buttons with header**                  | Header + 3 icon buttons (no share clip). Each has event out-binding |
| **Icon button with 3 lines of text**            | 3 lines of text + icon button. Has **overlay variation**            |

**Dependencies:** None

| Release | Date        | Notes                                             |
|---------|-------------|---------------------------------------------------|
| R1.0    | 23 Sep 2024 | First release                                     |
| R1.1    | 24 Sep 2024 | Icon color matches button color in 3-line variant |
| R2.0    | 04 Oct 2024 | Icon button with 3 lines overlay variant added    |

### Simple Buttons

> Source: [Simple buttons](https://www.notion.so/a93a998c46d64a489860af1839f716f0)
> Developer: riju@remixlabs.com

Three variants of single/dual buttons:

| Variant                                | Description                                                       |
|----------------------------------------|-------------------------------------------------------------------|
| **Icon button with dark custom color** | Customizable color, label, icon. Event out-binding for navigation |
| **Icon button white**                  | White background, icon color customizable only                    |
| **Two horizontal buttons**             | Two side-by-side buttons, each with customizable color and label  |

**Dependencies:** None

| Release | Date        | Notes                                          |
|---------|-------------|------------------------------------------------|
| R1.0    | 23 Sep 2024 | First release                                  |
| R1.1    | 24 Sep 2024 | Icon color customization added to dark variant |

### Appclip Share

> Source: [Appclip Share](https://www.notion.so/c6eba32b63484d60abd0b373cfdf96da)
> Developer: riju@remixlabs.com

Opens a native share sheet for end users to share a URL (typically an appclip link). Related: the `icon button with share clip option with header` complex button also includes this functionality.

**Dependencies:** None
**Release:** R1.0 — 24 Sep 2024

### Email and Phone Buttons

> Source: [Email and Phone buttons](https://www.notion.so/608e91e4cd59454286402090df06c4f7)
> Developer: riju@remixlabs.com

Two variants for contact actions:

| Variant                                | Description                                              |
|----------------------------------------|----------------------------------------------------------|
| **Email and phone button with header** | Email + phone buttons; opens mail/phone app respectively |
| **Email button with header**           | Email button only; opens mail app                        |

In-bindings: email address, phone number, header, icon color, icons.

**Dependencies:** None
**Release:** R1.0 — 24 Sep 2024

---

## Overlay Modal

> Source: [Overlay Modal](https://www.notion.so/1aa20ff3c50e4569b08f19c74069ea7a)
> Developer: riju@remixlabs.com

Generic modal container for overlay screens. Cross/close button is outside the main container for a cleaner look. Mainly intended for video players but reusable for anything needing a modal overlay.

**Dependencies:** Requires a card to be placed inside (no purpose on its own)
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/modals`

| Release | Date        | Notes                                                  |
|---------|-------------|--------------------------------------------------------|
| R1.0    | 20 Sep 2024 | First release                                          |
| R1.1    | 03 Oct 2024 | Back button navigation pre-configured inside component |

---

## List of Texts

> Source: [List of texts](https://www.notion.so/417c0e30d8044b659221751a4337088b)
> Developer: riju@remixlabs.com

Text/paragraph list components. Three assets:

| Asset                                      | Description                                               |
|--------------------------------------------|-----------------------------------------------------------|
| **Points/steps with header and paragraph** | List of numbered points, each with a header and paragraph |
| **Simple paragraph text with header**      | Similar but without point headers                         |
| **Simple bullet text with header**         | Bullet points under a single header                       |

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`
**Release:** R1.0 — 18 Sep 2024

---

## Review Assets

> Source: [Review assets](https://www.notion.so/7c3dabb3f1644d6c8bd6b5c749ed163f)
> Developer: riju@remixlabs.com

Display customer reviews with rating stars, reviewer name, optional image, and review text. Two variants:

| Variant                 | Description                                                                                                           | Dependencies                                |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------|---------------------------------------------|
| **Review with Yelp**    | Pulls reviews via Yelp API using business ID (get via `get_yelp_id` agent). Configure: database, orgAmpPrefix, token. | `get_yelp_reviews` cloud agent in workspace |
| **Review with Clipper** | Hardcoded reviews or reviews clipped via Remix Chrome Extension. Fields: name, image_url (optional), rating, text.    | Chrome Extension (optional)                 |

**Limitations:** Yelp API returns max 3 reviews per call. Does not work in Chrome browsers (works in runtime, App Clips, .remix files).

Star colors configurable via accent color binding.

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/reviews`
**Release:** R1.0 — 11 Sep 2024

---

## Forms

> Source: [Forms](https://www.notion.so/2a835f37d2874066accbb2d4a02f7cbf)

Container page for form assets used in DBP flows.

### Lead Form

> Source: [Lead form](https://www.notion.so/f99a82543fd74668928f01fffdd29df6)
> Developer: riju@remixlabs.com

Captures lead information: **name, email, company, title**. For use in DBP flows.

- Customizable: button color, success toast text
- Has **overlay variation**
- **Dependency:** Remix server agent for submission
- **Related:** Inbound Contact Requests asset (for browsing submitted leads)

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/forms`

| Release | Date        | Notes                                      |
|---------|-------------|--------------------------------------------|
| R1.0    | 23 Jul 2024 | First release                              |
| R1.1    | 04 Sep 2024 | UI consistency improvements                |
| R1.2    | 30 Sep 2024 | Performance: latest input field components |
| R2.0    | 04 Oct 2024 | Overlay variant added                      |

### Simple Contact Form

> Source: [Simple Contact Form](https://www.notion.so/1d071e04ab5b49f48f28a312adb025c2)
> Developer: riju@remixlabs.com

Captures contact info: **name, email, phone number**. For use in DBP flows.

- Customizable: button color, success toast text
- Has **overlay variation**
- **Dependency:** Remix server agent for submission
- **Related:** Inbound Contact Requests asset

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/forms`

| Release | Date        | Notes                                      |
|---------|-------------|--------------------------------------------|
| R1.0    | 20 Sep 2024 | First release                              |
| R1.1    | 30 Sep 2024 | Performance: latest input field components |
| R2.0    | 04 Oct 2024 | Overlay variant added                      |

---

## Banners

> Source: [Banners](https://www.notion.so/cda4a08e28874f42987417dd4f79f2e5)

Container page for banner components. All banners configurable via configurator or in-bindings (height, image URL, text). All are in the About Us design library at
`remix-india-beta.remixlabs.com/e/edit/about_us/banners`.

### Banners with Back Buttons

> Source: [Banners with back buttons](https://www.notion.so/6336070433614ec389f973ce7bb6fa3f)
> Developer: riju@remixlabs.com

For **non-home screens** (sub-pages). Background image + 1 or 2 lines of text + back button.

| Variant                           | Description      |
|-----------------------------------|------------------|
| Banner with back button + 1 line  | Single text line |
| Banner with back button + 2 lines | Two text lines   |

**Dependencies:** None | **Release:** R1.0 — 18 Sep 2024

### Banners with Logos and Buttons

> Source: [Banners with logos and buttons](https://www.notion.so/969074b8bbaa45a78591bed8f6a25ec5)
> Developer: riju@remixlabs.com

For **home/main screens**. Background image + logo + icon button + 1 or 2 lines of text. Configurable: logo, logo background color, background image, icon, button text, button navigation screen,
button color.

| Variant                                 | Description               |
|-----------------------------------------|---------------------------|
| Splash banner curved + button + 1 line  | Single text line with CTA |
| Splash banner curved + button + 2 lines | Two text lines with CTA   |

**Dependencies:** None | **Release:** R1.0 — 18 Sep 2024

### Profile Page Banners

> Source: [Profile page banners](https://www.notion.so/bc6c2faf2fe74ee8be6f3a794e9a744f)
> Developer: riju@remixlabs.com

For **personal/business profile pages** (non-home screens). Background image + name + title (job title / tagline) + back button.

| Variant                           | Description                         |
|-----------------------------------|-------------------------------------|
| Profile page banner               | Standard profile header             |
| Profile page banner with LinkedIn | Adds LinkedIn icon linking to a URL |

**Dependencies:** None | **Release:** R1.0 — 20 Sep 2024

### Banners with Home Button

> Source: [Banners with home button](https://www.notion.so/3952d0976c014436bb17ee8a3565f1f9)
> Developer: riju@remixlabs.com

For **home/splash/contact screens** (also usable in admin flows). Background image + 1 or 2 lines of text + home button.

| Variant                           | Description      |
|-----------------------------------|------------------|
| Banner with home button + 1 line  | Single text line |
| Banner with home button + 2 lines | Two text lines   |

**Dependencies:** None | **Release:** R1.0 — 21 Oct 2024

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
