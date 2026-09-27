# PRD — Shankar Custom T-Shirt Design Studio

## 1. Business context

Shankar Hemrajani is founder and CEO of **Sweet Ginger Fashions**, a Jaipur apparel company grown from a single 150 sq ft T-shirt outlet over 17+ years. It runs three verticals:

- **Sweet Ginger Basics** — B2B/wholesale blanks: plain T-shirts, polos, hoodies, sweatshirts for resellers, corporate gifting, events, printers.
- **The T-Shirt Shop** — retail/D2C: physical stores plus an online store.
- **Ginger Prints** — printing and customization: DTF, embroidery, vinyl, for tees and caps.

Today, every custom order is negotiated over WhatsApp and email, one message at a time. Shankar wants a **customizable T-shirt design system a customer can use on their own**, for both a single B2C order and a B2B bulk order.

## 2. The problem, stated precisely

The bottleneck is the **design convergence loop**, not "no website": a customer describes an idea in words, someone mocks it up, sends a photo, the customer asks for changes, repeat — across two channels, with no record of what was actually approved, and no structured order until a human converts a conversation into a print job.

This costs: production time spent on back-and-forth; ambiguity at handoff (a photo of a mockup, not a precise spec); no reusable order record, no size/quantity breakdown for bulk.

## 3. Users

- **B2C customer (The T-Shirt Shop)** — one shirt, self-serve, pays immediately.
- **B2B buyer (Sweet Ginger Basics)** — orders for resale/events/corporate gifting, needs a quantity broken down per size against one design, bulk pricing.
- **Shankar / production (Ginger Prints)** — needs, per order: garment, exact design placement, and a file usable for the actual print method (DTF, embroidery, or vinyl) — not a photo, not a thread.

**Both single and bulk orders go through the same studio.** This is not two separate flows with a shared canvas bolted on — one product picker, one design canvas, one live preview, with quantity/size breakdown as the variable that differs between a B2C and a B2B order.

## 4. Product shape (reference: CustomInk Design Lab)

Pick a shirt, add a design, see it on the shirt, order it — CustomInk's flow: product → text/art → placement → colour → price → cart. Drop Studio (AI design generation, background removal, realistic mockups) is a different shape entirely and is **optional, only if there's time** — not the core of this build (see §7).

## 5. Functional requirements

### 5.1 Product picker
- Browse blanks: **T-shirts first** (crew, oversized, polo), then hoodies and caps if time allows.
- Each product has its own colours and sizes.
- Show a **price per product**, and a **bulk price for B2B quantities**, both visible at the picker stage — not only revealed at checkout.

### 5.2 Design canvas
- Pick a product and colour, then design on it: add **text** and upload **artwork** (PNG or JPG).
- Move, scale, rotate each element.
- Support **front and back** of the shirt as two design surfaces on the same order.
- A **print area the design cannot leave** — enforced, and visually indicated, not just clipped silently.
- Text controls: font choice, text colour, size.

### 5.3 Live preview
- The design sits on the **actual selected shirt colour**, not a generic mockup — and updates as the customer edits. It must read as close to printed, not pasted flat on top (shading/blend appropriate to fabric colour, not just a flat image layer).
- Switching colour or size **must not lose the design** — this is a hard invariant, not a nice-to-have (see §6).

### 5.4 Order
- Quantity, size breakdown, and price, all updated live.
- B2B: buyer sets a **quantity per size** against the one design. B2C: quantity is usually one.
- Cart and simple checkout.
- The design is **saved with the order** so it can be printed — an order without artwork is not a valid, printable order.

### 5.5 Admin side (Shankar's side — required for v1, not a stretch goal)
- A list of orders, each with its design attached.
- Download the artwork, or a **print-ready file** matching the order's print method (DTF, embroidery, or vinyl).
- Mark an order's status: in production → printed → shipped.

### 5.6 Optional, only if there is time
- Generate a design from a text prompt; remove the background from an uploaded image (the Drop Studio shape — fits the printing business, but is not core).
- Save a design so a returning customer can reuse it.

**Per the decision already made for this MVP: none of §5.6 ships in v1.** It is not deferred as "phase 2 by default" either — it's built only if explicitly requested later, exactly like the prior de-risking call to keep the ordering flow's entire critical path free of any external model dependency. See §7.

## 6. Rules that must hold (invariants, not preferences)

- The design is never lost when colour, size, or quantity changes.
- The preview matches the chosen shirt colour and the chosen placement.
- The price updates from the product and the quantity, with a bulk tier for B2B.
- An order always carries its design; an order without artwork is not printable.
- Both single and bulk orders go through the same studio.

## 7. Explicit non-goals (v1)

- **No AI-generated designs, no AI background removal, no AI mockup photography** (the Drop Studio shape from §5.6). Zero model dependency anywhere in the ordering flow — product picker, canvas, live preview, pricing, cart, checkout, and admin are all deterministic logic. If any external service is down, the entire studio still works end to end.
- **No saved-design-for-reuse** in v1 (§5.6) — requires either an account system or a device-local store; add only once the core flow is proven.
- **No WhatsApp order automation** — same Meta Business API approval gate as any WhatsApp integration; order handoff to Shankar is in-app (admin order list) plus email, not WhatsApp, in v1.
- **No live inventory/production-floor sync** — the admin panel tracks order status (in production/printed/shipped) as a manual flag Shankar sets; it does not connect to an actual production or inventory system.

## 8. Decisions taken (no live client to interview — academy build, decided and documented, not left open)

The brief posed five open questions. Answered here as a domain call, with reasoning, so the build has no blocking unknowns. Each is a stated assumption, reversible if a real business review contradicts it — not a guess left unlabeled.

**1. Which vertical leads the first version?**
**Decision: The T-Shirt Shop (retail, single orders) leads.** Reasoning: both verticals share one studio (§6), so this only decides build order, not architecture. Retail/single-order is the simpler path through the same picker → canvas → preview → checkout — it proves the core mechanics (does the design survive a colour change, does the preview match, does the price update) before that same core is stressed with a size/quantity grid. CustomInk's own reference flow is fundamentally B2C-shaped; building toward that first, then extending the order step into a per-size quantity grid for Sweet Ginger Basics, is lower risk than the reverse.

**2. Bulk price breaks — per colour or per print method?**
**Decision: pricing varies by print method, not by garment colour.** Reasoning: blank garment cost is roughly colour-independent for a given style/size in standard cotton blanks; what actually drives cost is the print method (DTF > embroidery > vinyl, in typical per-unit cost for a given area) and quantity. Bulk tiers, applied on top of the print-method base price:
- 1–9 units: standard unit price
- 10–49: ~10% off
- 50–199: ~20% off
- 200+: custom tier — flagged for manual review before checkout completes, since true bulk (200+) production slotting is a real operational constraint even with an instant-checkout tool.

**3. Artwork rules the printers need — file type, resolution, max print area?**
**Decision:** accept PNG or JPG on upload (per brief); enforce a minimum effective resolution of 300 DPI at the design's placed print size (flag and warn, don't silently accept, if a customer's upload would print blurry); PNG preferred for logos with transparency. Maximum print area is defined **per garment, per side**, using standard industry dimensions as the starting point (adult tee: ~12"×16" front, ~10"×12" back) — pending confirmation against Shankar's actual press/frame sizes if this were a live engagement.

**4. Does print method change what the design tool allows?**
**Decision: yes**, and the canvas enforces this per selected print method:
- **DTF** — full-colour, photographic gradients allowed; largest print area of the three.
- **Vinyl** — solid/spot colours only (cap at a small fixed palette, no photographic gradients); print area capped smaller than DTF.
- **Embroidery** — solid colour blocks only, smallest print area, and a minimum text height enforced (below ~0.25" a stitched letterform stops being legible) — the tool blocks placement rather than allowing an unproducible design through.

**5. Stand-alone, or inside the existing store?**
**Decision: stand-alone for this build.** Reasoning: this is the de-risking call consistent with the rest of the plan — a stand-alone studio is fully demonstrable and deployable on its own, doesn't depend on credentials or integration access to Shankar's existing store platform, and can hand off a finished order to an existing store via a simple webhook/API later if that integration is ever pursued. Building the integration first would make the whole submission depend on infrastructure this project doesn't control.

## 9. Reference material

- **CustomInk Design Lab** — the flow to copy: product, text and art, placement, colour, price, cart.
- **Drop Studio** — AI design generation, background removal, realistic mockups (optional, §5.6 only).
- **The T-Shirt Shop** — his retail store and its existing Customize entry point.
- **Sweet Ginger Basics** — his B2B/wholesale site.
