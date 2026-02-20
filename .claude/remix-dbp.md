# Remix DBP — Digital Business Profile / About Us Catalog

> Parent: [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9)
> Ancestor: Remix Documentation > Remix Catalogs and Toolkits

---

## Overview

The "About Us" catalog provides reusable media and content components for building Digital Business Profile (DBP) websites. All assets are also available in the **About Us design library** on `remix-india-beta.remixlabs.com/e/edit/about_us/`.

---

## Media Assets

> Source: [Media](https://www.notion.so/500fd31c62d5457ca4044a26782adda5)

Container page for media-related DBP components. Sub-pages: Video Players, List of videos, Image galleries, List of images with text, images (unfetched).

---

### Video Players

> Source: [Video Players](https://www.notion.so/ec18a532129e4dd6a4f37ae9d445387d)
> Developers: padmanabha@remixlabs.com, riju@remixlabs.com

Two types of video players:

| Player | Description | Dependency |
|--------|-------------|------------|
| **Remix video player** | Plays videos stored on website/cloud/Remix workspace. Requires thumbnail. | ~~`rmx-video.js` in Files~~ (no longer needed as of R3.1) |
| **Embedded video player** | Supports YouTube/Vimeo embedded videos. No thumbnail needed. | None |

Both have **overlay variations** (full-screen overlay playback).

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/image_and_video`

| Release | Date | Notes |
|---------|------|-------|
| R1.0 | 23 Jul 2024 | First release |
| R1.1 | 04 Sep 2024 | UI consistency improvements |
| R2.0 | 09 Sep 2024 | Split into two players (workspace vs embedded) |
| R3.0 | 03 Oct 2024 | Overlay variants added |
| R3.1 | 14 Oct 2024 | No resource file dependency needed |

---

### List of Videos

> Source: [List of videos](https://www.notion.so/a96d7f9e95a545b080d788ff5d40d8e2)
> Developer: riju@remixlabs.com

Video list components — add multiple video links with thumbnails. Two variants:

| Variant | Description | Dependency |
|---------|-------------|------------|
| **Embedded list of videos** | YouTube/Vimeo embedded videos | None |
| **List of videos (Remix player)** | Videos stored in Remix workspace | Remix video player (rmx-video.js) |

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`

| Release | Date | Notes |
|---------|------|-------|
| R1.0 | 13 Sep 2024 | First release |
| R1.1 | 14 Oct 2024 | No resource file dependency needed |
| R1.2 | 15 Oct 2024 | Bug fix: configurator not working if asset already in project |

---

### Image Galleries

> Source: [image galleries](https://www.notion.so/7c82e8be64384df7b6277edec1f14df7)
> Developer: riju@remixlabs.com

Display images as a catalog/gallery. Configurable: image list, images per row (`how_many_per_row` binding), optional full-screen screen on tap.

Three gallery types:

| Type | Description |
|------|-------------|
| **Image gallery with hero** | First image displayed larger (hero), rest in grid |
| **Image gallery without hero** | All images in uniform grid |
| **Single row image gallery** | One row only, shows up to `how_many_per_row` images |

Also includes **image full screen overlay** variant.

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`

| Release | Date | Notes |
|---------|------|-------|
| R1.0 | 11 Sep 2024 | First release |
| R2.0 | 04 Oct 2024 | Image overlay variant added |

---

### List of Images with Text

> Source: [List of images with text](https://www.notion.so/d09fa785ff5541df9ab50b74500cc93e)
> Developer: riju@remixlabs.com

Similar to image gallery but includes **two lines of text per image** — suitable for:
- Listing items with names and descriptions
- People listings with profile image, name, and title

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`

| Release | Date | Notes |
|---------|------|-------|
| R1.0 | 25 Sep 2024 | First release |

---

## List of Texts

> Source: [List of texts](https://www.notion.so/417c0e30d8044b659221751a4337088b)
> Developer: riju@remixlabs.com

Text/paragraph list components. Three assets:

| Asset | Description |
|-------|-------------|
| **Points/steps with header and paragraph** | List of numbered points, each with a header and paragraph |
| **Simple paragraph text with header** | Similar but without point headers |
| **Simple bullet text with header** | Bullet points under a single header |

**Dependencies:** None
**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/lists`
**Release:** R1.0 — 18 Sep 2024

---

## Review Assets

> Source: [Review assets](https://www.notion.so/7c3dabb3f1644d6c8bd6b5c749ed163f)
> Developer: riju@remixlabs.com

Display customer reviews with rating stars, reviewer name, optional image, and review text. Two variants:

| Variant | Description | Dependencies |
|---------|-------------|--------------|
| **Review with Yelp** | Pulls reviews via Yelp API using business ID (get via `get_yelp_id` agent). Configure: database, orgAmpPrefix, token. | `get_yelp_reviews` cloud agent in workspace |
| **Review with Clipper** | Hardcoded reviews or reviews clipped via Remix Chrome Extension. Fields: name, image_url (optional), rating, text. | Chrome Extension (optional) |

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

| Release | Date | Notes |
|---------|------|-------|
| R1.0 | 23 Jul 2024 | First release |
| R1.1 | 04 Sep 2024 | UI consistency improvements |
| R1.2 | 30 Sep 2024 | Performance: latest input field components |
| R2.0 | 04 Oct 2024 | Overlay variant added |

### Simple Contact Form

> Source: [Simple Contact Form](https://www.notion.so/1d071e04ab5b49f48f28a312adb025c2)
> Developer: riju@remixlabs.com

Captures contact info: **name, email, phone number**. For use in DBP flows.

- Customizable: button color, success toast text
- Has **overlay variation**
- **Dependency:** Remix server agent for submission
- **Related:** Inbound Contact Requests asset

**Design library:** `remix-india-beta.remixlabs.com/e/edit/about_us/forms`

| Release | Date | Notes |
|---------|------|-------|
| R1.0 | 20 Sep 2024 | First release |
| R1.1 | 30 Sep 2024 | Performance: latest input field components |
| R2.0 | 04 Oct 2024 | Overlay variant added |

---

## Banners

> Source: [Banners](https://www.notion.so/cda4a08e28874f42987417dd4f79f2e5)

Container page for banner components. All banners configurable via configurator or in-bindings (height, image URL, text). All are in the About Us design library at `remix-india-beta.remixlabs.com/e/edit/about_us/banners`.

### Banners with Back Buttons

> Source: [Banners with back buttons](https://www.notion.so/6336070433614ec389f973ce7bb6fa3f)
> Developer: riju@remixlabs.com

For **non-home screens** (sub-pages). Background image + 1 or 2 lines of text + back button.

| Variant | Description |
|---------|-------------|
| Banner with back button + 1 line | Single text line |
| Banner with back button + 2 lines | Two text lines |

**Dependencies:** None | **Release:** R1.0 — 18 Sep 2024

### Banners with Logos and Buttons

> Source: [Banners with logos and buttons](https://www.notion.so/969074b8bbaa45a78591bed8f6a25ec5)
> Developer: riju@remixlabs.com

For **home/main screens**. Background image + logo + icon button + 1 or 2 lines of text. Configurable: logo, logo background color, background image, icon, button text, button navigation screen, button color.

| Variant | Description |
|---------|-------------|
| Splash banner curved + button + 1 line | Single text line with CTA |
| Splash banner curved + button + 2 lines | Two text lines with CTA |

**Dependencies:** None | **Release:** R1.0 — 18 Sep 2024

### Profile Page Banners

> Source: [Profile page banners](https://www.notion.so/bc6c2faf2fe74ee8be6f3a794e9a744f)
> Developer: riju@remixlabs.com

For **personal/business profile pages** (non-home screens). Background image + name + title (job title / tagline) + back button.

| Variant | Description |
|---------|-------------|
| Profile page banner | Standard profile header |
| Profile page banner with LinkedIn | Adds LinkedIn icon linking to a URL |

**Dependencies:** None | **Release:** R1.0 — 20 Sep 2024

### Banners with Home Button

> Source: [Banners with home button](https://www.notion.so/3952d0976c014436bb17ee8a3565f1f9)
> Developer: riju@remixlabs.com

For **home/splash/contact screens** (also usable in admin flows). Background image + 1 or 2 lines of text + home button.

| Variant | Description |
|---------|-------------|
| Banner with home button + 1 line | Single text line |
| Banner with home button + 2 lines | Two text lines |

**Dependencies:** None | **Release:** R1.0 — 21 Oct 2024

---

## Unfetched Sub-pages

- [images](https://www.notion.so/e1816e3726314bf49950c736b82af61f) — not yet fetched (under Media)
- [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9) — parent hub page, not yet fetched
