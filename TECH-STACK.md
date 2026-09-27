# Tech Stack — Shankar Custom T-Shirt Design Studio

Every choice below is paired with the constraint that forced it. No model/AI provider is used anywhere in v1 — the entire studio (picker, canvas, preview, pricing, cart, checkout, admin) is deterministic, per the PRD's non-goals. The brief itself suggests Next.js + Supabase/Neon; the table below evaluates that against the previously-approved stack and resolves the conflict rather than silently picking one.

## Core stack

| Layer | Choice | Constraint that forced it |
|---|---|---|
| App framework | **Next.js** (not plain Vite/React) | The brief's own suggestion, and it's the right call here specifically because the admin side (§5.5) needs server-side routes (order list, file downloads) alongside the customer-facing studio — a single Next.js app covers both without standing up a second service |
| Hosting | Vercel or Netlify | Zero-config deploy for Next.js; matches the brief |
| Database | **Postgres (Supabase or Neon)** — see trade-off below | The design record is structured and nested (an array of elements, each with type/x/y/width/height/rotation/colour, per front **and** back) — this is exactly what a relational/JSONB column handles cleanly and what a flat spreadsheet-shaped store does not |
| Auth | Included with Supabase (or a minimal auth layer on Neon) | Needed for the admin side (Shankar logging in to see orders) even if customers themselves don't need accounts in v1 |
| Design canvas | **Fabric.js** | Built-in move/resize/rotate handles for text and image objects, and `toJSON()` serializes an object's position/size/rotation directly into the structured design record the PRD requires — both the customer-facing interaction and the machine-usable export come from the same library |
| Garment/colour rendering | Flat garment mockup images (front + back, one set per colour) with print-area bounds as fixed coordinates per garment/side | No 3D rendering needed — CustomInk's own flow is 2D flat-preview; front/back are two separate mockup+bounds pairs, not a 3D-rotatable model |
| Payment | Razorpay | Standard India payment gateway; handles cards/UPI for both B2C checkout and B2B bulk checkout (same studio, same checkout, per PRD §6) |
| File storage (uploaded artwork) | Supabase Storage (or S3-compatible, if using Neon) | Uploaded PNG/JPG needs a stable URL referenced from the structured design record — not stored as a blob in the database row |
| Order handoff / notification | A database trigger or a lightweight scheduled job → email to Shankar on new order | Keeps notification logic next to the data instead of introducing a separate automation platform for one email |

## Database vs. Airtable (resolving the conflict with the prior approved-stack list)

| | Airtable | Postgres (Supabase/Neon) |
|---|---|---|
| Fits a flat catalog (products, colours, sizes, prices) | Yes, well | Yes, but more setup for something this simple |
| Fits a **design record** (nested array of canvas elements, per front/back, with x/y/rotation/colour) | Poorly — Airtable fields are flat; nesting requires JSON-stuffing a text field, which then can't be queried or validated | Natively — JSONB column, or a normalized elements table, both queryable |
| Fits admin order-status workflow (in production → printed → shipped) | Fine | Fine |
| **Recommendation** | Use for nothing structural in this build | **Use Postgres for the whole system.** The one place Airtable would have been convenient — Shankar editing his catalog without touching code — is better solved by a simple admin screen in the same Next.js app, since he also needs an admin screen for order status anyway. Building two admin surfaces (Airtable for catalog, custom UI for orders) is worse than one. |

This is a case where the previously-assumed default (Airtable, Make.com) doesn't fit the actual data shape this brief describes, and the brief's own suggested stack (Next.js + Supabase/Neon) is the better-reasoned choice — not a deviation to flag for permission, but one to record here so it's traceable.

## Canvas library: Fabric.js vs Konva (trade-off)

| | Fabric.js | Konva |
|---|---|---|
| Object model | Rich built-in objects (text, image) with resize/rotate handles out of the box | Lower-level; more manual wiring for handles |
| Serialization | `toJSON()` gives element positions/sizes/rotation directly — maps cleanly to the structured design record | Also serializable, but needs more custom shaping |
| Front/back as two design surfaces | Two canvas instances (or one canvas, two serialized states) — either way, straightforward with Fabric's JSON model | Same is possible, more boilerplate |
| **Recommendation** | **Fabric.js** — direct-to-JSON export is exactly what turns a customer's canvas edits into Shankar's printable spec | |

## Explicit exclusions

- **No AI design generation, no AI background removal** (§5.6 of the PRD) — not in v1, not a "phase 2 by default" either. Only built if explicitly requested later.
- **No WhatsApp Business API integration** — Meta approval gate; order handoff is the in-app admin panel plus email in v1.
- **No Claude API or any LLM call anywhere in the ordering flow** — product picker, canvas, pricing, and admin are all deterministic. The only place a model could ever enter this system is the explicitly-excluded §5.6 stretch goal.

## What breaks first, and the fallback

The highest-risk technical decision is **print-area accuracy per garment, per side, per print method** — the coordinates that bound where a design can sit on front and back mockups. Embroidery, DTF, and vinyl don't share the same maximum print area or the same colour/gradient limits (decided in PRD §8.3–4: DTF full-colour/largest area, vinyl solid-colour/smaller area, embroidery solid-colour/smallest area with a minimum legible text height). Getting this wrong doesn't surface as a bug until an early order reaches the print floor and the placement or colour doesn't match what the customer approved on screen — this is measured against real or representative garments per print method, not assumed from a single generic mockup template, before the canvas stage begins.
