# Tech Stack — Shankar Custom T-Shirt Design Studio

Every choice below is paired with the constraint that forced it. Approved stack only: React/Vite, canvas library, Airtable, Razorpay, Make.com, Bolt/Lovable. No model/AI provider is used anywhere in v1 — the entire ordering flow is deterministic, per the PRD's non-goals.

| Layer | Choice | Constraint that forced it |
|---|---|---|
| App | React + Vite + TypeScript | Fast local iteration, no backend server required to run the editor itself |
| Design canvas | **Fabric.js** | Purpose-built for exactly this: drag/resize/rotate objects (text + images) on a canvas, built-in object controls, serializes to JSON for the print-ready export. Konva is the alternative (see trade-off below) |
| Garment/colour rendering | Flat garment mockup images (front view, one per colour) with the print-area bounds defined per garment as fixed coordinates | No 3D/fabric-drape rendering needed for v1 — CustomInk's own flow is 2D flat-preview, not 3D |
| Product/order database | Airtable | No backend to stand up; Shankar can edit catalog and see orders without touching code; matches the approved stack |
| Payment (B2C) | Razorpay | Approved stack; standard India payment gateway, handles cards/UPI |
| Payment (B2B) | Razorpay payment link generated after quote approval (not instant checkout) | Matches the submit-for-quote assumption in the PRD; avoids building a negotiated-pricing engine in v1 |
| Automation / order handoff | Make.com | New Airtable order record → email notification to Shankar with order summary + exported design image. No custom backend needed for this glue step |
| Hosting / scaffold | Bolt or Lovable | Approved stack; fastest path to a deployed storefront shell |
| Local state (design-in-progress) | Browser state only (React state + Fabric.js canvas state), submitted to Airtable only on order placement | No draft-saving backend needed for v1; if "save and resume" becomes a requirement later, add localStorage first before adding a backend |

## Canvas library: Fabric.js vs Konva (trade-off)

| | Fabric.js | Konva |
|---|---|---|
| Object model | Rich built-in objects (text, image, shapes) with resize/rotate handles out of the box | Lower-level; more manual wiring for handles, but more control |
| Serialization | `toJSON()` gives element positions/sizes/rotation directly — maps cleanly to the print-ready export the PRD requires | Also serializable, but requires more custom shape of the output |
| React integration | Works fine imperatively; less "React-native" | Has `react-konva` bindings, more idiomatic in a React tree |
| Recommendation | **Fabric.js** — the built-in transform controls and direct-to-JSON export are exactly the two things this build needs (customer-facing move/resize/rotate, and a machine-usable spec for Shankar), and outweigh the marginally cleaner React bindings Konva offers. | |

## Explicit exclusions

- **No Drop Studio / AI design generation, no AI mockup photography.** Different product shape; not part of this stack.
- **No WhatsApp Business API integration in v1** — requires Meta approval (up to 7 days), same constraint noted in prior builds. Order notifications go by email via Make.com until that's set up.
- **No Voiceflow / Retell AI / voice** — this is a visual design tool, not a conversational agent. Not applicable here.
- **No Claude API / any LLM call anywhere in the ordering flow** — canvas placement, pricing, and catalog lookups are all rule-based. If an "AI logo cleanup" feature is added later (phase 2, only if customer uploads prove messy), that would be the first and only place a model enters this system.

## What breaks first, and the fallback

The single highest-risk technical decision is **print-area accuracy**: the coordinates that define where a design can legally sit on each garment mockup image. If these are wrong, either the live preview lies to the customer or the exported design doesn't match what Shankar can actually print. This must be measured against Shankar's real garments (photograph the actual printable zone, don't guess it from a stock mockup), the same "measure it, don't reason about it" principle as the print-area/colour-contrast checks in prior builds.
