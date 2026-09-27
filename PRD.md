# PRD — Shankar Custom T-Shirt Design Studio

## 1. Business context

Shankar runs a T-shirt business selling both B2B (bulk, brands/teams/events) and B2C (single custom shirt), and manufactures the shirts himself. Reference sites: his B2C store ("The T-Shirt Shop") and his B2B site (Sweet Ginger — sweetginger.in, custom printed team/brand shirts).

Today, every custom order — text, logo, placement, colour, size, quantity — is negotiated over WhatsApp and email, one message at a time. Nothing is captured in a structured way until Shankar personally converts a conversation into a print job.

## 2. The problem, stated precisely

The bottleneck is not "no website." It is the **design convergence loop**: a customer describes an idea in words, Shankar (or someone) mocks it up, sends a photo, the customer asks for changes, repeat — often across several days, across two channels (WhatsApp + email), with no record of which version was approved.

This has three costs:
- Shankar's time is spent on back-and-forth instead of production.
- Ambiguity at handoff: what reaches the print stage is a photo of a mockup, not a precise, positioned spec.
- No structured order record — no history, no reordering, no bulk-quantity breakdown by size.

## 3. Users

- **B2C customer** — wants one shirt, self-serve, pays immediately, wants to see exactly what they're getting before paying.
- **B2B buyer** — orders on behalf of a team/brand/event, needs quantity broken down by size, likely negotiates price or needs approval before paying, may need an invoice rather than instant card payment.
- **Shankar (producer/admin)** — needs, per order: garment (style/colour/size), the exact design (text/logo, position, size, rotation) in a producible form, and the total to print — not a photo, not a WhatsApp thread.

## 4. Product shape (reference: CustomInk Design Lab)

Take the shape from CustomInk, not the shape from Drop Studio. CustomInk's flow is a manual editor: choose a product → add text or upload artwork → move, resize, rotate it on the shirt → pick colour and size → price updates live → check out. Drop Studio is AI-prompt-to-design and AI-generated mockup photography — a different product entirely, and out of scope (see §7).

The customer converges on the design themselves. What reaches Shankar is already resolved: a garment spec, a design spec, and an order — not another ambiguous request.

## 5. Functional requirements

### 5.1 Product & catalog
- Fixed catalog: garment styles Shankar actually stocks (need his real list — round-neck, polo, hoodie, etc.).
- Each style has: available colours, available sizes, base price.
- Catalog data lives in Airtable, editable by Shankar without a code change.

### 5.2 Design canvas
- Add text: free text, font choice (limited set), text colour, font size.
- Upload artwork/logo: common image formats (PNG, JPG, SVG); customer moves, resizes, rotates it on the garment.
- Multiple elements per design (e.g. front text + logo) — stretch goal if time allows, not blocking v1.
- Live preview: design renders on the selected garment colour in real time as changes are made.
- Print-area constraint: element placement is bounded to a printable area per garment (not the whole image canvas) — this must be visually indicated, not just enforced silently.
- A rights checkbox at upload: "I own the rights to use this artwork" — standard liability hygiene, not a legal fix.

### 5.3 Colour & size selection
- Colour swatches update the garment preview under the design (design does not move; garment colour changes).
- Size selector (S–XXL or Shankar's real range) drives price where sizes are priced differently (e.g. XXL+ upcharge), and drives the B2B quantity grid.

### 5.4 Pricing
- Live price recalculates on every change: garment style, colour (if colour affects price), size, number of print locations, quantity (B2B).
- Pricing rules live in Airtable/config, not hardcoded — Shankar can change prices without a rebuild.

### 5.5 B2C checkout
- Single shirt, single design, pay now via Razorpay.
- Order confirmation shows the final design + garment + total.

### 5.6 B2B bulk order
- Same design canvas, but order intake is a size/quantity grid (e.g. S:10, M:25, L:15, XL:5) against one design.
- Needs a decision from Shankar (see open questions): instant checkout at bulk price, or submit-for-quote (he confirms price/turnaround before payment is collected). Default assumption for v1: **submit-for-quote**, because bulk pricing and production slotting are usually negotiated, not fixed — Shankar should override this in the first review if wrong.

### 5.7 Order handoff to Shankar
- Every completed order (B2C paid, or B2B submitted) produces:
  - A structured record (garment, colour, size(s)+qty, price, customer contact) in Airtable.
  - A print-ready export of the design: a flattened image of the design in position, plus the raw parameters (element type, x/y, width/height, rotation, colour) as data — so it is both human-checkable and machine-usable later.
- A notification reaches Shankar (email at minimum; WhatsApp is phase 2, gated on Meta Business API approval — same constraint that applies to any WhatsApp automation).

## 6. Success criteria for the MVP

- A customer can go from "pick a shirt" to "paid order placed" with zero messages exchanged with Shankar, for a single-item order.
- A B2B buyer can submit a bulk order with a full size/quantity breakdown and one design, without a phone call.
- Every order Shankar receives contains everything needed to print, with no photo-of-a-mockup step.

## 7. Explicit non-goals (v1)

- **No AI-generated designs or AI mockup photography** (the Drop Studio shape). Not deferred to phase 2 by default — only added later if explicitly requested.
- **No AI background removal / logo cleanup** on upload. Phase 2 candidate only if real customer uploads turn out to need it.
- **No WhatsApp order automation** — gated on Meta Business API approval (7-day process), same as prior builds. Web + email only in v1.
- **No voice ordering.**
- **No live production/inventory sync** — Shankar still manages physical stock and print scheduling himself; this system produces the order + design spec, it does not run his print floor.
- **Model dependency is zero for the entire ordering flow.** Every feature in §5 is deterministic logic (canvas math, pricing rules, catalog lookups). This is a deliberate de-risking decision: if any external service is down, the core flow still works.

## 8. Open questions (need Shankar's answers before build starts)

1. Real product catalog: which garment styles, colours, sizes does he actually stock/produce?
2. Print method (screen print vs DTG vs vinyl) — this determines whether colour choice is free-form or constrained, and whether the print area is one fixed zone or flexible.
3. B2B flow: instant bulk checkout, or submit-for-quote-then-pay? (v1 assumes submit-for-quote — confirm or override.)
4. What does he need per order to actually print — is a flattened image + position data enough, or does his production process need a specific file format?
5. Payment for B2B: full payment upfront, deposit, or invoice/pay-on-delivery?
