# Remix DBP — Media & Data Visualization

> Parent: [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9)
> Ancestor: Remix Documentation > Remix Catalogs and Toolkits
> See also: [remix-dbp-components.md](./remix-dbp-components.md), [remix-dbp-assembly.md](./remix-dbp-assembly.md)

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
