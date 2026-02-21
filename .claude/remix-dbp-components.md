# Remix DBP — Buttons, Forms, Banners & Content Components

> Parent: [Business Profile / About Us Catalog](https://www.notion.so/1051d464528f801e975cc2bf6cd715b9)
> Ancestor: Remix Documentation > Remix Catalogs and Toolkits
> See also: [remix-dbp-media.md](./remix-dbp-media.md), [remix-dbp-assembly.md](./remix-dbp-assembly.md)

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
