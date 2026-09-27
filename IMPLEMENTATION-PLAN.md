# Implementation Plan — Shankar Custom T-Shirt Design Studio

Build order below is sequenced so every stage is demonstrable on its own — each one produces something Shankar can look at and react to, rather than a long stretch of invisible progress.

## Stage 0 — Catalog & pricing data (prerequisite, before any UI)
- Load real product catalog into Airtable: garment styles, colours, sizes, base prices, per-size upcharges.
- Define print-area coordinates per garment/colour mockup image (photograph or measure the real printable zone).
- **Demonstrable output:** an Airtable base Shankar can open and confirm "yes, this is my real catalog and my real prices."
- **Blocked on:** Shankar's answers to PRD §8 open questions (catalog, print method, B2B flow, payment terms).

## Stage 1 — Product picker
- Customer selects garment style → colour → size from the Stage 0 catalog data.
- **Demonstrable output:** a working picker screen, no design yet, no payment yet.

## Stage 2 — Design canvas: text
- Fabric.js canvas over the garment mockup, bounded to the print area.
- Add text, choose font/colour/size, move/resize/rotate.
- **Demonstrable output:** a customer can put their name on a shirt and see it, live, in the selected colour.

## Stage 3 — Design canvas: logo upload
- Upload PNG/JPG/SVG, place on canvas alongside or instead of text, move/resize/rotate.
- Rights checkbox on upload.
- **Demonstrable output:** a customer can upload a real logo and position it on the shirt.

## Stage 4 — Live pricing
- Price recalculates from garment + colour + size + number of design elements, using Stage 0's pricing rules.
- **Demonstrable output:** price on screen updates in real time as the customer changes anything.

## Stage 5 — B2C checkout
- Razorpay payment for a single shirt/single design order.
- On success: order record written to Airtable, design exported (flattened image + Fabric.js JSON position data).
- **Demonstrable output:** a full B2C order, start to finish, with a real payment and a real record in Airtable.

## Stage 6 — B2B bulk flow
- Same canvas, but order intake becomes a size/quantity grid against one design.
- Submit-for-quote flow: buyer submits, Shankar reviews/confirms price, buyer receives a Razorpay payment link (per PRD's default assumption — confirm with Shankar before building this stage).
- **Demonstrable output:** a bulk order with a size breakdown lands in Airtable, distinguishable from a B2C order.

## Stage 7 — Order handoff automation
- Make.com scenario: new Airtable order → email to Shankar with order summary, design export, and customer contact.
- **Demonstrable output:** Shankar receives an email the moment an order is placed, with everything he needs to print — no WhatsApp thread required.

## The decision most expensive to reverse

**Print-area coordinates per garment (Stage 0).** Every later stage — canvas bounds, live preview accuracy, and the exported print file — depends on these being correct. Getting this wrong doesn't surface as a bug until Shankar tries to actually print an early order and finds the design position doesn't match what customers saw on screen. This is measured against his real garments before Stage 1 begins, not assumed from a generic mockup template.

## Verification checklist before calling any stage done

- Live preview matches the exported design (same position, same size, same rotation) — check this explicitly, don't assume Fabric.js's canvas and its JSON export agree by construction.
- Price shown to the customer matches the price recorded in Airtable for the same order.
- B2C and B2B orders are both traceable end-to-end in Airtable without opening a code editor.
- Works with zero external services reachable except Razorpay at the checkout step — confirms the "no model dependency" claim in the PRD actually holds in the running app, not just in the plan.
- Test at phone width (390px) and desktop width — the canvas and picker must both be usable on a phone, since most of Shankar's customers will land here from a WhatsApp-shared link.

## Suggested week-by-week cadence

| Week | Stages | Outcome Shankar sees |
|---|---|---|
| 1 | Stage 0 + 1 | Real catalog loaded, product picker working |
| 2 | Stage 2 + 3 | Full design canvas — text and logo — working on real garments |
| 3 | Stage 4 + 5 | Live pricing + a real paid B2C order end-to-end |
| 4 | Stage 6 + 7 | B2B bulk flow + automated order handoff to his inbox |
