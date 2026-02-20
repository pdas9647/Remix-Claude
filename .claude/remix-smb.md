# Remix SMB Catalog

> Source: [SMB catalog](https://www.notion.so/1051d464528f80a09405d83aec33ffb9)
> Parent: Remix Documentation > Remix Catalogs and Toolkits

---

## Overview

The Remix SMB Catalog provides assets for assemblers to build applications for small to medium businesses. Goal: a partner can assemble and deploy an app for an SMB vertical **in one day**.

- Target verticals: restaurants, dentists/doctors, hair salons, grocery stores, gardening services, plumbers, auto repair, small hotels, bookstores, etc.
- Assets include: components, cloud agents, UI layouts, cards — all built on a shared data model
- Integrations: Stripe (payments), getaddress.io (UK address lookup), HubSpot
- All UI assets available in the `_rmx_drafts` library

### Navigation Guide

1. Understand deployment footprints (workspaces, databases, cloud agents, end-user apps)
2. Learn the data model (entities used by catalog assets)
3. Browse live applications and demo videos
4. Explore individual catalog asset pages

---

## Transaction Models

> Source: [Transaction models for SMBs](https://www.notion.so/1291d464528f8077b390e89eec31115d)

Three transaction models supported:

| # | Model                                  | Example                                    |
|---|----------------------------------------|--------------------------------------------|
| 1 | **Ordering from a catalog**            | Ordering from a restaurant menu            |
| 2 | **Booking services provided by staff** | Hair salon appointment, dental appointment |
| 3 | **Booking resources**                  | Hotel room, restaurant table               |

---

## Data Model

> Source: [Data model for a small to medium business](https://www.notion.so/10d1d464528f8055b80bc699da804886)

### Core Entities

- **`business`** — Main entity. Has one or more `venues`.
- **`venue`** — A branch/location of the business. Contains:
    - `business_profile` — name, email, phone, social media handles
    - `address`
    - `settings` — timezone, currency, date/time format
    - `business_hours` and `holiday` schedule
    - `delivery_setup` (if delivery supported)
    - `payments` information
    - `printer_list` (optional)

### Catalog Data Model

> Source: [Data model: Catalog](https://www.notion.so/88f26fb868cb4724ab86f0b0ccf502c6)

Hierarchical structure (all sorted lists):

```
Catalog
  └── Category (0..N)
        └── Item (0..N, belongs to exactly one Category)
              ├── Item Variant (0..N, not reusable across Items)
              │     └── Option Group (0..N)
              └── Option Group (0..N, not reusable — copied)
                    └── Option (0..N, can appear in multiple Option Groups)
```

**Design decisions:**

- Item Variants are NOT reusable across Items
- Option Groups are NOT reusable — they are copied
- Single Item cannot map to multiple Categories
- Item Variant vs Option Group: modeling choice (e.g. "Chicken Curry or Korma" can be modeled either way)

### Booking Services Data Model

> Source: [Booking services provided by staff](https://www.notion.so/1291d464528f80429975c270abe1b80c)

For service-based SMBs (hair salons, dental offices):

- A `business` has `venues`
- A `venue` is open for `business_hours`
- A `venue` has `staff` (employees or contractors)
- `staff` working hours defined by `staff_shift`
- Products/services defined in a `catalog` (items grouped by categories)
- `staff` offer certain `items` within their `staff_shifts`
- Bookings and staff shifts managed through a `business_calendar`
- A `booking` is made by a `customer` who picks a `staff` and a set of `items` fitting within availability

---

## Order Data Model

> Source: [Data model: Order](https://www.notion.so/5cb5cb553449403cad142247d3bb9acf)
> JSON example: [JSON for an Order](https://www.notion.so/11c1d464528f803ea04ff017e590c4c3)

### Order Lifecycle — Dual Status Model

Orders have two status attributes:

**Payment Status:**

- `Paid` — Payment received
- `Authorized` — Credit card authorized by processor (e.g. Stripe)
- `Unpaid` — Payment not yet received (cash orders)
- `Refunded` — Payment refunded (credit card: authorization released, not actually charged)

**Order Status:**

- `Pending` — Received, not yet reviewed
- `Processing` — Accepted, being fulfilled
- `Rejected` — Rejected by business
- `Expired` — Not reviewed within time window
- `Canceled` — Canceled after acceptance
- `Completed` — Fulfilled and delivered

### Valid Status Combinations

**Cash orders:**

| Combination       | Meaning                         |
|-------------------|---------------------------------|
| Unpaid-Pending    | Placed, not yet reviewed        |
| Unpaid-Processing | Accepted, being fulfilled       |
| Unpaid-Rejected   | Rejected, never paid            |
| Unpaid-Canceled   | Canceled post-acceptance        |
| Unpaid-Expired    | Not reviewed in time            |
| Paid-Processing   | Payment made, not yet fulfilled |
| Paid-Completed    | Done                            |

**Credit card orders:**

| Combination        | Meaning                                        |
|--------------------|------------------------------------------------|
| Unpaid-Expired     | Buyer abandoned Stripe session                 |
| Authorized-Pending | Placed with card auth, not yet reviewed        |
| Paid-Processing    | Accepted (acceptance triggers payment capture) |
| Refunded-Rejected  | Rejected, authorization released               |
| Refunded-Canceled  | Canceled, payment refunded                     |
| Refunded-Expired   | Expired, payment refunded                      |
| Paid-Completed     | Done                                           |

### Order Attributes

| Attribute           | Type     | Notes                               |
|---------------------|----------|-------------------------------------|
| `entity`            | string   | Always `"order"`                    |
| `order_number`      | string   | e.g. `"0998"`                       |
| `status`            | string   | Order status (see above)            |
| `payment_status`    | string   | Payment status (see above)          |
| `payment_method`    | string   | `"cash"`, `"card"`, etc.            |
| `filter_status`     | string   | Combined e.g. `"processing-unpaid"` |
| `currency_code`     | string   | e.g. `"GBP"`                        |
| `fulfilment_method` | string   | e.g. `"takeaway"`                   |
| `fulfilment_time`   | dateTime | Expected fulfilment time            |
| `ordered_at`        | dateTime | When order was placed               |
| `customer`          | object   | `{name, phone, email, address}`     |
| `line_items`        | array    | Array of line items                 |
| `sub_total`         | number   | In minor units (pence/cents)        |
| `tax_total`         | number   | In minor units                      |
| `delivery_fee`      | number   | In minor units                      |
| `total`             | number   | In minor units                      |
| `discounts`         | array    | Applied discounts                   |
| `codes`             | array    | Promo/voucher codes                 |
| `campaign_id`       | string   | Marketing campaign reference        |

### Line Item Attributes

| Attribute        | Type        | Notes                                                           |
|------------------|-------------|-----------------------------------------------------------------|
| `item_id`        | _rmx_id ref | Reference to Catalog Item                                       |
| `name`           | string      | Item name                                                       |
| `category`       | object      | `{entity, id, name}`                                            |
| `quantity`       | number      |                                                                 |
| `price`          | number      | In minor units                                                  |
| `original_price` | number      | Pre-discount price                                              |
| `variant`        | object      | `{entity, id, name, price}` — empty if no variant               |
| `options`        | array       | Selected option groups with `option_group` + `selected_options` |
| `discounts`      | array       | Line-level discounts                                            |
| `description`    | string      | Auto-generated summary of variant + options                     |
| `image`          | string      | Item image URL                                                  |

**Note:** Prices in minor units (e.g. 490 = GBP 4.90). DateTime values use Remix tagged type `{tag}case:calendar.dateTime`.

---

## SMB Catalog Assets

> Source: [SMB catalog assets](https://www.notion.so/1521d464528f8074aa60c259b498851b)

All UI assets are in the `_rmx_drafts` library.

### UI Assets

| Category           | Asset                             | Description                                         | Status    |
|--------------------|-----------------------------------|-----------------------------------------------------|-----------|
| **Business Setup** | Business profile                  | Edit/view business name, email, phone, social media | Available |
|                    | Business settings                 | Timezone, time/date format, currency                | Available |
|                    | Address - UK                      | Business venue address (getaddress.io)              | Targeted  |
|                    | Business hours                    | Opening hours per day, multiple periods             | Available |
|                    | Holidays                          | Override normal business hours                      | Targeted  |
| **Catalog**        | Inventory availability (toggle)   | Mark items available/unavailable                    | Available |
|                    | Inventory availability (quantity) | Track item quantities                               | Available |
| **Ordering**       | Shopping cart - simple            | Basic cart                                          | Available |
|                    | Shopping cart - with loyalty      | Cart with loyalty program                           | Available |
|                    | Shopping cart - with inventory    | Cart with inventory checks                          | Available |
|                    | Order list                        | List of orders                                      | Available |
|                    | Order detail                      | Single order view                                   | Available |
|                    | Local delivery areas/fees - list  | Delivery zone management                            | Available |
|                    | Local delivery area - detail      | Single zone config                                  | Available |
|                    | Local delivery area - delete      | Zone removal                                        | Available |
|                    | Catalog layout - simple           | Basic catalog display                               | Available |
|                    | Catalog layout - inventory        | Catalog with stock levels                           | Available |
|                    | Stripe setup                      | Payment configuration                               | Targeted  |
| **Customer**       | Customer list / detail            | Customer management                                 | Available |
|                    | Loyalty program (view/setup/edit) | 3 assets for loyalty config                         | Available |
|                    | Order list / detail               | Customer order history                              | Available |
|                    | Order analytics                   | Order reporting                                     | Available |
| **Misc**           | Printer list                      | POS printer management                              | Targeted  |
|                    | Print - customer                  | Customer receipt commands                           | Available |
|                    | Print - kitchen                   | Kitchen chit commands                               | Available |

### Cloud Agents

Cloud agents table exists but is mostly empty (placeholder for Customers category).

---

## Printer Integration

> Source: [Printer Integration](https://www.notion.so/e0e420eb2e254a2293cba1f4a337c138)
> Developer: padmanabha@remixlabs.com
> Release: v1.0 — 24 September 2024

Two components for POS printer integration:

### Print Customer Chit

Generates detailed customer receipts via API call. Includes:

- Order details (items, quantity, description, price, discounts)
- Customer info (name, contact)
- Payment details (method, total, tax, tips)
- Venue info (name, address, contact, logo)
- Additional notes (instructions, coupons, comments)

### Print Kitchen Chit

Generates kitchen order chits for meal preparation. Includes:

- Order breakdown (items, quantities, modifications, customizations)
- Customer info (order number for prioritization)
- Payment details (optional — prepaid vs pay later)
- Special instructions (allergies, dietary preferences)

Both components: no dependencies, generate print commands for POS printers.

---

## Ordering and Order Management

> Source: [Ordering and order management](https://www.notion.so/10b1d464528f807e9d03efef673da2cb)

Container page. Ordering channels supported: Web, Phone, WhatsApp, Mobile app, POS, Kiosk, QR code.

Sub-areas (from sidebar navigation):

- Shopping cart
- Address entry (getaddress.io)
- Payments (Stripe)
- Order management (prep time, order mgmt app, printer integration)
