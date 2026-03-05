# Remix SMB Catalog

> Source: [SMB catalog](https://www.notion.so/1051d464528f80a09405d83aec33ffb9)
> Parent: Remix Documentation > Remix Catalogs and Toolkits

---

## Overview

Assets for assemblers to build SMB apps (restaurants, salons, grocery, auto repair, etc.) — goal: deploy in **one day**. Integrations: Stripe (payments), getaddress.io (UK address), HubSpot. All UI assets in `_rmx_drafts` library.

---

## Transaction Models

> Source: [Transaction models](https://www.notion.so/1291d464528f8077b390e89eec31115d)

1. **Ordering from a catalog** — e.g. restaurant menu
2. **Booking services by staff** — e.g. hair salon, dental ([details](https://www.notion.so/1291d464528f80429975c270abe1b80c))
3. **Booking resources** — e.g. hotel room, restaurant table

---

## Data Model

> Source: [Data model](https://www.notion.so/10d1d464528f8055b80bc699da804886)

### Core Entities

- **`business`** — main entity; has one or more `venues`
- **`venue`** — branch/location; contains: `business_profile` (name, email, phone, social), `address`, `settings` (timezone, currency, date/time format), `business_hours`, `holiday` schedule, `delivery_setup`, `payments`, `printer_list` (optional)

### Catalog Hierarchy

> Source: [Catalog data model](https://www.notion.so/88f26fb868cb4724ab86f0b0ccf502c6)

```
Catalog → Category (0..N) → Item (0..N, one Category only)
  Item → Item Variant (0..N, NOT reusable across Items)
  Item or Variant → Option Group (0..N, copied not shared) → Option (0..N, can appear in multiple groups)
```

### Booking Services Model

> Source: [Booking services](https://www.notion.so/1291d464528f80429975c270abe1b80c)

`business` → `venue` (with `business_hours`) → `staff` (with `staff_shift`) → offer `items` from `catalog`. A `booking` = `customer` picks `staff` + `items` fitting `business_calendar` availability.

---

## Order Data Model

> Source: [Order](https://www.notion.so/5cb5cb553449403cad142247d3bb9acf) | [JSON example](https://www.notion.so/11c1d464528f803ea04ff017e590c4c3)

### Dual Status Model

**Payment status:** `Paid` | `Authorized` (card auth'd by Stripe) | `Unpaid` (cash) | `Refunded`

**Order status:** `Pending` → `Processing` → `Completed` | `Rejected` | `Expired` | `Canceled`

### Valid Combinations

**Cash:** Unpaid-Pending → Unpaid-Processing → Paid-Processing → Paid-Completed. Rejection: Unpaid-Rejected. Cancellation: Unpaid-Canceled. Timeout: Unpaid-Expired.

**Credit card:** Unpaid-Expired (abandoned Stripe session) | Authorized-Pending → Paid-Processing (acceptance triggers capture) → Paid-Completed. Rejection: Refunded-Rejected. Cancellation: Refunded-Canceled. Timeout: Refunded-Expired.

### Order Attributes

Key fields: `entity` ("order"), `order_number`, `status`, `payment_status`, `payment_method` ("cash"/"card"), `filter_status` (combined, e.g. "processing-unpaid"), `currency_code`, `fulfilment_method` (e.g. "takeaway"), `fulfilment_time` (dateTime), `ordered_at` (dateTime), `customer` ({name, phone, email, address}), `line_items` (array), `sub_total`/`tax_total`/`delivery_fee`/`total` (minor units, e.g. 490 = GBP 4.90), `discounts` (array), `codes` (promo/voucher), `campaign_id`.

### Line Item Attributes

Key fields: `item_id` (_rmx_id ref), `name`, `category` ({entity, id, name}), `quantity`, `price`/`original_price` (minor units), `variant` ({entity, id, name, price} — empty if none), `options` (array of {option_group, selected_options}), `discounts`, `description` (auto-generated summary), `image`.

DateTime values use tagged type `{tag}case:calendar.dateTime`.

---

## Catalog Assets

> Source: [SMB catalog assets](https://www.notion.so/1521d464528f8074aa60c259b498851b)

All in `_rmx_drafts` library. Status: Available unless noted.

- **Business Setup:** Business profile, Business settings, Business hours | *Targeted:* Address (UK, getaddress.io), Holidays
- **Catalog:** Inventory availability (toggle), Inventory availability (quantity)
- **Ordering:** Shopping cart (simple / with loyalty / with inventory), Order list/detail, Local delivery areas+fees (list/detail/delete), Catalog layout (simple / inventory) | *Targeted:* Stripe setup
- **Customer:** Customer list/detail, Loyalty program (view/setup/edit), Order list/detail, Order analytics
- **Misc:** Print customer chit, Print kitchen chit | *Targeted:* Printer list

Cloud agents: placeholder (Customers category only).

### Printer Integration

> Source: [Printer integration](https://www.notion.so/e0e420eb2e254a2293cba1f4a337c138) | Dev: padmanabha@remixlabs.com | v1.0, Sep 2024

Two components (no dependencies): **Print Customer Chit** (receipt: order items, customer info, payment, venue branding, notes) and **Print Kitchen Chit** (prep: items with modifications, order number, special instructions/allergies). Both generate POS printer commands.

---

## Ordering Channels

> Source: [Ordering and order management](https://www.notion.so/10b1d464528f807e9d03efef673da2cb)

Web, Phone, WhatsApp, Mobile app, POS, Kiosk, QR code. Sub-areas: shopping cart, address entry (getaddress.io), payments (Stripe), order management (prep time, order mgmt app, printer integration).
