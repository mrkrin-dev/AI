# Implementation Plan — Shankar Custom T-Shirt Design Studio

Build order is sequenced so every stage is demonstrable on its own. Both B2C and B2B run through the same studio throughout — there is no separate "B2B track" to build later; bulk is the quantity/size-breakdown step applied to the same picker, canvas, and preview.

## Stage 0 — Catalog, print areas, and data model (prerequisite)
- Load product catalog into Postgres: T-shirts (crew, oversized, polo) first, retail/single-order use case leading per PRD §8.1; hoodies and caps deferred unless time allows.
- Each product: colours, sizes, unit price, bulk tiers (per PRD §8.2: 1–9 / 10–49 / 50–199 / 200+ flagged for review).
- Print-area coordinates per garment, **per side (front/back), per print method** (DTF/embroidery/vinyl each get their own max area and colour/gradient rule, per PRD §8.3–4) — built as a print-method dimension from day one, not retrofitted later.
- **Demonstrable output:** a catalog with front/back print-area bounds marked per print method, on real or representative garment photos.

## Stage 1 — Product picker
- Browse T-shirts (crew, oversized, polo) with colours and sizes.
- Show unit price and bulk price for B2B quantities directly on the picker, per the brief.
- **Demonstrable output:** a working picker, price visible before any design work starts.

## Stage 2 — Design canvas: text, front side
- Fabric.js canvas over the front garment mockup, bounded to the front print area.
- Add text: font, colour, size; move/scale/rotate.
- **Demonstrable output:** text placed and styled on the shirt front, visibly bounded to the print area.

## Stage 3 — Design canvas: artwork upload, front side
- Upload PNG/JPG, place alongside or instead of text, move/scale/rotate.
- **Demonstrable output:** a real logo placed on the shirt front.

## Stage 4 — Back side + colour/size switching without data loss
- Same canvas capability applied to the back print area, as a second design surface on the same order.
- Switch garment colour and size and confirm the design (front and back) survives unchanged — this is the invariant from PRD §6, tested explicitly here, not assumed.
- **Demonstrable output:** a two-sided design that survives a colour and size change in front of Shankar.

## Stage 5 — Live preview realism
- Design renders on the actual selected shirt colour with shading/blend appropriate to that colour, not a flat pasted layer — this needs a visual check against a real printed sample, not just code review.
- **Demonstrable output:** side-by-side of the on-screen preview against Shankar's own printed reference for the same design, judged close enough to ship.

## Stage 6 — Live pricing + order (size/quantity breakdown)
- Price recalculates from product, colour, and quantity, with the bulk tier applying automatically once quantity crosses the threshold.
- Order step: size/quantity grid (S:10, M:25, …) for B2B; defaults to quantity 1 for B2C, same UI.
- **Demonstrable output:** price updates live as quantities change across sizes, bulk tier kicking in correctly.

## Stage 7 — Cart, checkout, and the printable-order guarantee
- Cart holds one or more configured products; Razorpay checkout for the total.
- Hard rule enforced at checkout: an order cannot be placed without an attached design (PRD §6) — reject or block, don't allow an empty/blank submission.
- On success: order + full structured design record (elements, positions, sizes, rotation, colour, per side) written to Postgres.
- **Demonstrable output:** a complete order — B2C and, separately, a B2B bulk order — each paid and stored with its design attached.

## Stage 8 — Admin side
- Order list, each row showing its attached design (thumbnail + link to full detail).
- Download the flattened artwork, or the structured print-ready data, per the order's print method.
- Status flag per order: in production → printed → shipped.
- **Demonstrable output:** Shankar opens the admin panel, finds a real order, downloads a print-ready file, and marks it in production — with zero WhatsApp/email back-and-forth.

## Stage 9 — Optional, only if there is time
- Text-prompt design generation and background removal on uploaded artwork (the Drop Studio shape).
- Saved designs for returning customers.
- **Not built unless Stages 0–8 are done and confirmed working** — this is the one place in the whole system a model would be introduced, and only after the fully deterministic core is proven end to end.

## The decision most expensive to reverse

**Print-area coordinates per garment, per side, per print method (Stage 0).** Every later stage depends on these being right — the canvas bounds, the live preview's honesty, and the printable file all trace back to this. PRD §8.3–4 already decides that print method changes the allowed area and colour rules, so Stage 0 builds that dimension in from the start rather than risking a rework after the canvas assumes one universal print area.

## Verification checklist before calling any stage done

- Colour or size change never alters or drops the design (front or back) — test this explicitly, don't assume it from the code path.
- The live preview and the exported/stored design record agree on position, size, rotation, and colour for the same design.
- Bulk pricing tier applies at the correct quantity threshold, and the same checkout path handles both a B2C single order and a B2B bulk order (per PRD §6: both go through the same studio).
- An order cannot reach the admin list without an attached design.
- The admin-downloaded print file actually matches what the printer needs for that order's print method (confirm with Ginger Prints' actual production process, not just visually).
- Test at phone width (390px) and desktop — most customers land here from a shared link, not a desktop session.

## Suggested week-by-week cadence

| Week | Stages | Outcome Shankar sees |
|---|---|---|
| 1 | Stage 0 + 1 | Real catalog + print-area data confirmed, product picker working |
| 2 | Stage 2 + 3 + 4 | Full two-sided design canvas, colour/size switching provably non-destructive |
| 3 | Stage 5 + 6 | Preview realism signed off against a real printed sample; live bulk pricing working |
| 4 | Stage 7 + 8 | Real paid orders (B2C and B2B) landing in an admin panel he can act on |
