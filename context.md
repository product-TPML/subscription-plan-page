# Context: Subscription plan page prototype

A static HTML prototype of a redesigned **subscription / paywall page** for two newspaper brands from the same publisher:

- **DH**: Deccan Herald (English), "DH Premium"
- **PV**: Prajavani (Kannada), "Prajavani Premium" plus E-Magazines (Sudha, Mayura)

The main goal is to make the **plan picker work better on mobile**: all plans, their prices and the shared benefits should fit on the first screen. The rest of the page copies the live subscribe page (deccanherald.com/subscribe) so the new picker can be judged in context.

There is no build step, framework or package manager. Open `index.html` in a browser.

## Files

| Path | What it is |
|---|---|
| `index.html` | The prototype: one file holding all the CSS, markup and JS. The page is rendered from data objects in the `<script>`. |
| `wireframes.html` | Greyscale mobile wireframes (360px phone frames with a ~640px fold line) comparing the plan-section layouts: Current, Option 1 (swipe full cards), Option 2 (short stacked cards plus a common list, marked recommended), Option 3 (short swipeable cards), and **2B (select a plan, then one button)**. 2B is the layout `index.html` builds. |
| `Assets/` | Benefit icons (`premium articles.svg`, `no ads.png`, `epaper.png`, `all editions.svg`, `magazine.png`, `mobile and desktop.png`, `offline.png`), the badge (`premium tag.svg`), magazine logos (`Sudha.svg`, `Mayura.svg`) and `magazine and web view.png`. Several filenames contain spaces, so URL-encode them (the JS uses `encodeURI`). |
| `Assets/fonts/` | Prajavani brand fonts (woff2), used for the PV testimonials. |
| `Assets/testimonials/` | Reader photos, named `dh-*.jpg` / `pv-*.jpg`. |

## Page structure (`index.html`, top to bottom)

1. **Brand switch** (for the prototype only): DH/PV toggle fixed to the right edge. On mobile it sits level with the premium badge, at `top: 67px`, just below the offer banner. It sets the URL hash to `#dh` / `#pv`, and the page loads whichever hash is in the URL.
2. **Offer banner** (`.offer-banner`): yellow→gold gradient strip at the very top, copied from the live page (gradient, padding, and sizes, with a separate mobile gradient). The copy comes from `brand.offer`. The PV banner uses the Prajavani fonts.
3. **Hero**: content column with `max-width: 1018px`, so the back button lines up with the plan cards' left edge. It holds the **back button**, which is visual only (48px on desktop, 32px on mobile, white box with a gold arrow, like the live page), plus the premium badge, title and subtitle. Offering tabs appear only when a brand has more than one offering (PV: Premium / E-Magazines). The tabs have a sliding `#942A2A` pill that JS moves into place.
4. **Plan picker** (`.picker`):
   - "Every plan includes" box listing the benefits shared by the offering
   - a radio-card `fieldset` of plans (3 columns on desktop; compact grid rows on ≤1024px)
5. **Why choose**: carousel of 5 cards with inline SVG icons.
6. **Stats** band (subscriber count).
7. **Testimonials**: carousel, text clamped to 3 lines. "… See more" appears only on cards whose text is actually cut off, and opens a modal with a fixed header and a scrolling body. PV uses the Prajavani fonts through `[data-tfont]`.
8. **FAQ**: categories with an accordion. Questions use the tokens `{name}` / `{premium}`; the answers are lorem-ipsum placeholders.
9. **Contact**: phone and email pills, plus working hours.
10. **Subscribe dock** (`#cta-dock`): fixed to the bottom of the viewport for the whole page, kept in sync with the selected plan by `syncCta()`.
    - **Desktop**: kept deliberately simple. On the left, a small "Selected:" label above the plan name (19px). On the right, the 220px "Subscribe for ₹X" CTA. Price, tag, per-day price and benefits are left out on purpose; an earlier version listed the benefits and was too cluttered.
    - **Mobile**: a full-width CTA. An IntersectionObserver on `#plan-options` adds `.away` to the dock once the plans scroll out of view. A one-line summary then slides in above the button: plan name, per-day price, and a "Change plan" link that scrolls to the top and focuses the selected radio.
    - JS keeps `--dock-h` equal to the dock's height, and `body` uses it as bottom padding so the dock never covers the end of the page.

## Data model (edit content here; the page renders from it)

Everything lives near the top of the `<script>` in `index.html`:

- `BRANDS.dh` / `BRANDS.pv`: `name`, `premium`, `title`, `offer` (`[bold lead, text]` for the top banner; live copy as of Nov 2026), `stats`, `phone`, `email`, `offerings[]`
- Each **offering**: `{ id, label, included: [[label, iconFile]], plans: [...] }`
- Each **plan**: `{ id, name, price, was?, period, days, extras[], tag?, selected?, logos? }`
  - `days` is used to work out the per-day price (rounded down to 2 decimals)
  - `was` is the struck-through price
  - `extras` holds only what is specific to that plan; anything every plan has goes in `included`
  - `period` is title case ("Month", "Year", "6 Months", "2 Years"). It shows as "₹999 / Year" on both desktop and mobile.
  - `logos` (`["sudha"]`, `["sudha","mayura"]`) shows magazine logos in place of the name. `prefix` (optional) is text shown before the logos, e.g. "1 Year Digital +". The logo pair wraps onto a new line as one unit.
  - `selected: true` marks the default plan
- `NEWS_INCLUDED`: the shared news benefits. PV rewrites "All city editions" as "All 32 city editions".
- **PV plans**:
  - Premium tab: Monthly ₹149, 1 Year ₹999 (default), 2 Years ₹1899.
  - E-Magazines tab: Sudha ₹599, Mayura ₹299, Sudha + Mayura ₹749 (struck-through ₹999, default).
  - `PV_BUNDLE` ("1 Year Digital + Sudha + Mayura", ₹1499, struck-through ₹1999, tag "Best value") is one object placed last in **both** PV tabs. It has the same extras text in each tab.
- When a tab has exactly 4 plans, the desktop grid switches to 2×2 (`.options[data-count="4"]`) so no card is left alone on a row.
- `TESTIMONIALS.dh` / `.pv`: `{ name, img, text }`. An empty `img` falls back to initials.
- `FAQS`: `[category, [questions]]`

## Behaviour notes

- **Carousels** (Why choose, Testimonials) copy the live page: they autoplay every 4s and wrap to the start, pause on hover, pause on touch/drag/wheel and resume 4s later, and the active dot is the card nearest the scroll position. When a brand switch re-renders the cards, `carousel._refresh()` rebuilds the dots.
- **Plan tag chip** (`plan.tag`, e.g. "Most popular"; the text is configurable): copied from the live page's approach. It's absolutely positioned and straddles the card's top border at the top-right (`top:-12px`, or `-9px` on mobile, where the chip is smaller). Since it's out of the text flow, long plan names or tags can't push or wrap it. `max-width` plus an ellipsis stops a very long tag from running past the card. On mobile the gap between plan rows is 12px so the chip clears the row above. The desktop plan name has right padding so it doesn't run under the radio.
- On mobile (≤1024px) the carousel arrows are hidden and the cards are sized from `100vw`.
- The breakpoint is `max-width: 1024px` (the same as the live page). The testimonial modal gets bigger layout at `min-width: 769px`.
- Accessibility: radio inputs inside the cards, `aria-selected` tabs, `aria-expanded` FAQ toggles, and a modal with focus return, Esc to close and focus kept inside it. Motion respects `prefers-reduced-motion`.

## Design tokens

These come from deccanherald.com/subscribe and are set on `:root`:
gold `#c78d3c` (accent/selection), yellow `#ffd34d` (CTA), light beige `#f5ebde`, beige `#eeddc5`, navy text `#1c2742`, tab red `#942A2A`.
Fonts: Playfair Display (headings), Roboto Slab (body/cards), Roboto (sans), all from Google Fonts.

## Temporary / placeholder content (remove or replace before handoff)

- **DH mock plans** `m1`, `m2`, `m3` ("Mock 1–3"): added to test the sticky CTA with a long list of plans. The dock replaces that CTA, so these are no longer needed.
- The back button has no behaviour.
- The offer banner copy is a seasonal campaign (Diwali, Nov 2026).
- **"Dummy Long Reader"** testimonial in both DH and PV: there only to test scrolling in the modal.
- PV values that are still placeholders: `stats: "XX,XXX+"`, phone and email.
- Every FAQ answer is the same lorem-ipsum placeholder.
- CTA and contact links are `href="#"`.

## History (summary)

1. First prototype for DH and PV.
2. Added benefit icons and Sudha/Mayura logos, and moved assets into `Assets/`.
3. Made the magazine logos 20px tall on mobile.
4. Restyled the offering tabs with the sliding `#942A2A` pill and toned down the benefits box.
5. Added the sticky Subscribe button and the mock plans.
6. Added testimonial truncation and the popup, with the long dummy testimonial.
7. Added the top offer banner and the back button, and replaced the in-picker sticky CTA with a page-wide bottom dock (a full bar on desktop, a button plus a "Change plan" summary on mobile).
8. Moved the plan-tag chip out of the text flow, as on the live page. Set the real PV magazine prices, added the digital + magazines bundle to both PV tabs, added "/ duration" to mobile prices, and switched 4-plan tabs to a 2×2 grid on desktop.
