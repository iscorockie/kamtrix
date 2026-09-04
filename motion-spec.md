# Kamtrix Technologies — Interaction & Motion Script
**Senior Motion Design Spec · v1.0 · Template: `index.html` (Ink/Copper/Signal theme)**
**Date: 2026-09-04**

This document scripts every animation on the Kamtrix Technologies gadget store in plain language. Each entry gives the trigger, the visual change, and technical parameters (duration in ms + easing). Developers can implement directly from this with CSS transitions / WAAPI / GSAP ScrollTrigger.

> How to read an entry: **"On [trigger]: [what moves/changes] using [easing] over [duration]."** Followed by exact tokens.

---

## 0. Motion Language & Global Tokens

Kamtrix motion feels like a circuit powering up: fast, precise, copper-warm, with one teal `signal` glow moment per screen. Nothing bouncy or playful. Everything settles flat and confident.

### 0.1 Duration scale (use only these)

| Token | Value | Use for |
|---|---|---|
| `instant` | 100ms | Chip-dot blinks, link underlines |
| `micro` | 150ms | Button hovers, card border-color, `add-btn` fill |
| `quick` | 200ms | Overlay fade, chip pill select, toast in/out |
| `base` | 250ms | Cart drawer slide, toast slide, thumb zoom start |
| `calm` | 300ms | Nav compress, headline line slide |
| `reveal` | 450–600ms | Scroll reveals, card entrances |
| `hero` | 700–900ms | Page-load hero sequence |
| `ambient` | 26000ms | Marquee loop (linear only) |

### 0.2 Easing tokens (use exactly)

| Token | Value | Personality |
|---|---|---|
| `ease-out-expo` | `cubic-bezier(0.16, 1, 0.3, 1)` | Default for entrances, reveals, drawer. Fast start, long soft landing. |
| `ease-out` | `cubic-bezier(0.33, 1, 0.68, 1)` | Hovers, fades. |
| `ease-in-out` | `cubic-bezier(0.65, 0, 0.35, 1)` | Parallax / scrubbed scroll that must reverse cleanly. |
| `snap-back` | `cubic-bezier(0.34, 1.3, 0.64, 1)` | Badge pop, add-button press. Slight overshoot, never more than 4%. |
| `linear` | `linear` | Marquee, circuit dash flow only. |
| `spring-soft` (JS spring) | `mass: 1, stiffness: 320, damping: 28` | Cart count pop, toast entry on capable devices (WAAPI/GASP). Fallback = `snap-back` 250ms. |

### 0.3 Performance rules (applies to all sections)

- Animate **only `transform` and `opacity`** on scroll and hover. Never animate `width`, `height`, `top`, `left`, `padding`, or `box-shadow` per-frame. The nav "compress" in SC-01 is done with `transform: scaleY + translate` on inner content plus a single height class-toggle at the threshold — not a per-pixel height tween.
- `border-color` / `background-color` fades under 200ms are allowed — they do not trigger layout.
- Use `will-change: transform, opacity` **only while animating**: add the class on `animationstart` / IntersectionObserver entry, remove on `animationend`. Never leave `will-change` on the 12 product cards permanently.
- Images: always `transform: scale()` on the inner `img`, with `overflow: hidden` on `.thumb`. Add `contain: layout paint` to `.card` and `content-visibility: auto` to below-fold sections.
- Respect `prefers-reduced-motion: reduce` — the template already kills durations. In that mode: skip parallax, pinning, marquee animation (render static row), and replace all slides with 1-frame opacity fades over 1ms.
- Marquee uses `transform: translateX` on a GPU-composited track (`translateZ(0)`). Duplicate content 2× and loop `-50%` so there is no layout thrash.

---

## 1. Page Load Choreography

Total sequence: **0ms → ~1450ms**. One calm cascade, top to bottom, left to right. No loader screen — the page itself is the loader.

- **PL-01 — On page load: header bar slides down from 12px above and fades from 0 to 1 opacity using ease-out-expo over 400ms, starting at 0ms.**
  - Detail: `header` goes `opacity: 0, translateY(-12px)` → `opacity: 1, translateY(0)`. Backdrop blur (`blur(10px)`) is static — do not animate blur.

- **PL-02 — On page load: logo chip draws attention with a copper pulse using ease-out over 600ms, starting at 100ms.**
  - Detail: The 22px `.logo .chip` border flashes from `--line` to `--copper`, and the two 6px side ticks fade in from 0 opacity with a 2px outward nudge. One pulse only, then rests. This is the brand "power-on."

- **PL-03 — On page load: nav links stagger in from 8px below, 300ms duration with 60ms delay between each link, using ease-out-expo, starting at 150ms.**
  - Detail: Order: Shop → Categories → Why Us → Contact. Each link goes `opacity: 0, translateY(8px)` → visible. Cart button and WhatsApp button follow as a 5th step at +60ms.

- **PL-04 — On page load: hero eyebrow ("KAMPALA · GENUINE GADGETS") fades in and its teal dot glows on using ease-out over 400ms, starting at 200ms.**
  - Detail: Text slides up from 10px below. The 7px signal dot scales from 0 to 1 with `snap-back`, then its `box-shadow: 0 0 8px signal` blooms from 0 to full over the same 400ms. Then the dot holds a slow ambient pulse (see AM-01).

- **PL-05 — On page load: hero headline reveals line-by-line sliding up from 24px below, 700ms duration per line with 110ms delay between lines, using ease-out-expo.**
  - Detail: Line 1 "Real tech," starts at 300ms; line 2 "real fast," (copper `em`) at 410ms; line 3 "straight from Kampala." at 520ms. Each line is masked (overflow hidden wrapper) so text rises into place with no layout shift. Copper word finishes 80ms after its line lands with a color fade from paper to copper.

- **PL-06 — On page load: hero lede paragraph slides up from 20px below and fades in using ease-out-expo over 600ms, starting at 620ms.**
  - Detail: Max-width 46ch paragraph; keep line-height fixed so the rise does not reflow the CTAs.

- **PL-07 — On page load: the two CTA buttons rise from 16px below with stagger, 450ms duration and 90ms delay between them, using ease-out-expo, starting at 720ms.**
  - Detail: "Browse the shop" first, "Order on WhatsApp" second. Buttons land with a 1px border shimmer — border-color flashes copper at 30% then settles.

- **PL-08 — On page load: hero product panel slides in from 28px right and 16px below with a fade, 800ms duration using ease-out-expo, starting at 550ms (overlaps the headline for depth).**
  - Detail: `.hero-panel` goes `opacity: 0, translate(28px, 16px) scale(0.98)` → `opacity: 1, translate(0,0) scale(1)`. The two 7px copper corner dots pop in with `snap-back` over 250ms at 1050ms and 1120ms. The panel image inside holds scale 1.08 and settles to 1.0 over 1000ms (slow Ken Burns landing).

- **PL-09 — On page load: hero spec row (256GB / 4.8★ / order line) staggers up, 400ms each with 80ms delay between the three stats, using ease-out over 400ms, starting at 950ms.**
  - Detail: Each stat's bold value (`signal` color) counts up from 0 to its final value during its 400ms window; labels fade in without movement.

- **PL-10 — On page load: circuit SVG traces draw themselves left-to-right using linear flow over 1200ms, starting at 250ms.**
  - Detail: Each `path` in `.hero-circuit` animates `stroke-dashoffset` from full-length to 0 at 0.5 opacity. The five node circles (copper + signal) pop from scale 0 with `snap-back` 250ms, staggered 150ms apart starting at 800ms. Traces never re-animate on scroll — they are a load-only moment.

- **PL-11 — On page load: marquee strip fades in from 0 opacity with no movement, 500ms ease-out, starting at 1150ms, then begins its infinite scroll.**
  - Detail: Track is already positioned; only opacity animates to avoid a sideways jump. Loop runs `translateX(0 → -50%)` linear over 26000ms infinite (see SC-06).

- **PL-12 — On page load: below-fold sections sit parked 24px low at 0 opacity; they do not animate until scrolled to (see Section 2).**
  - Detail: Prevents a "flash of unstyled entrance" on fast connections.

---

## 2. Scroll-Triggered Effects

Trigger engine: IntersectionObserver at 15% visibility, `rootMargin: 0px 0px -8% 0px`, fire once (no re-hide). Parallax via `requestAnimationFrame` scroll listener, lerped (0.12) to avoid jank.

- **SC-01 — On scroll past 40px: navigation bar compresses from 80px to 60px visual height using ease-out over 300ms.**
  - Detail: Do not tween `height` per frame. Toggle a `.compressed` class at 40px: `.nav` padding goes `16px → 10px`, logo font `19px → 17px`, cart/WhatsApp buttons shrink one step. Inner content uses `transform: translateY(-2px) scale(0.99)` during the 300ms to sell the squeeze. Reverses symmetrically when scrolling back above 40px. Add `box-shadow: 0 4px 20px rgba(0,0,0,0.35)` only in compressed state (toggled, not animated).

- **SC-02 — On scroll: hero background glows drift with parallax at 0.15× scroll speed using ease-in-out, continuous while hero is in view.**
  - Detail: The two radial gradients (copper at 85%/20%, signal at 10%/80%) translate at 15% of scroll distance in the opposite direction (`transform: translateY(scrollY * 0.15)` on a dedicated bg layer). GPU-only transform. Disabled under `prefers-reduced-motion` and on screens < 760px wide to save battery.

- **SC-03 — On scroll: hero circuit SVG drifts slower than content at 0.08× speed and fades from 0.5 to 0.15 opacity across the hero exit, continuous.**
  - Detail: Creates depth: text scrolls normally, traces lag behind. Opacity fade is linear with scroll progress. No pinning — hero never locks; it scrolls away naturally.

- **SC-04 — On scroll into view: each section header (`.sec-head`) reveals as a unit — tag, then title, then description — sliding up from 24px below, 600ms duration with 100ms delay between the three pieces, using ease-out-expo.**
  - Detail: Applies to "Pick your gadgets," "How ordering works," "Built for how Kampala shops." Tag (copper mono) first, H2 second, paragraph third. Fire once.

- **SC-05 — On scroll into view: product cards (`.card`) rise from 28px below with stagger, 550ms duration and 70ms delay between each visible card, using ease-out-expo.**
  - Detail: First row of 4 staggers 0/70/140/210ms. Cards on later rows re-trigger per row as they enter. Card goes `opacity: 0, translateY(28px) scale(0.98)` → resting. Thumbnail image inside holds `scale(1.08)` and settles to 1.0 over 700ms for a soft focus-landing. Never animate more than 4 cards simultaneously — batch the rest.

- **SC-06 — On scroll: marquee track keeps its 26-second linear loop but eases to 2× speed while the user is actively scrolling, then relaxes back over 600ms using ease-out.**
  - Detail: Track is `translateX` only. Velocity boost is a playback-rate change, not a new animation. Pause the marquee when it is off-screen (IntersectionObserver toggles `animation-play-state`) to save GPU.

- **SC-07 — On scroll into view: "How it works" step numbers (01/02/03) count up from 00 and the connecting circuit line draws left-to-right, 600ms using ease-out-expo with 120ms stagger per step.**
  - Detail: Number goes `opacity: 0, translateY(12px)` → visible; a 1px `--line` connector between steps draws via `scaleX(0 → 1)` with `transform-origin: left`. On mobile (stacked layout) the connector draws top-to-bottom via `scaleY` instead. No pinning — steps scroll normally.

- **SC-08 — On scroll into view: why-cards (Genuine / Same-day / Warranty / Human) tilt up from 20px below with 500ms duration and 90ms stagger, using ease-out-expo; the icon ring draws on landing.**
  - Detail: Card rises; then the 34px `.ico` circle border draws (rotate a dashed ring from -90° to 0°) and the ✓/⚡/🛡/💬 glyph fades in over 200ms. Stagger keeps all four landings inside ~770ms total.

- **SC-09 — On scroll into view: contact panel scales from 0.97 to 1.0 and fades in over 600ms using ease-out-expo, with its CTA button arriving 150ms later from 12px below.**
  - Detail: Panel border glows copper at 40% opacity for 300ms on landing, then settles to `--line`. One-time emphasis so the conversion block feels "lit."

- **SC-10 — On scroll into view: footer columns fade up with 450ms duration and 80ms stagger (brand → Shop → Contact), using ease-out, plus the bottom bar rule draws via scaleX over 500ms.**
  - Detail: Quietest reveal on the page — small distance (16px), no scale. Footer never parallaxes.

- **SC-11 — No pinning anywhere on this template.**
  - Detail: Deliberate. This is a commerce page with a short viewport journey; pinning the hero, steps, or grid would trap mobile users and hurt conversion. Depth comes from parallax (SC-02/SC-03) and staggered reveals only.

---

## 3. Hover Micro-Interactions

All hovers respond within **1 frame (~16ms)** to pointer-enter, then complete over 150–400ms. On touch devices hovers are skipped (use `:hover` + `@media (hover: hover)` guards).

- **HV-01 — On hover over nav links: link color fades from dim to paper over 150ms using ease-out, with a 2px copper underline wiping in from left over 200ms using ease-out-expo.**
  - Detail: Underline is a `::after` with `scaleX(0 → 1)`, `transform-origin: left`. On leave it wipes out to the right over 150ms. No movement of surrounding links.

- **HV-02 — On hover over primary button ("Browse the shop"): background lightens from copper to #F0965C over 150ms using ease-out, and the button lifts 2px with a soft shadow over 200ms using ease-out-expo.**
  - Detail: `transform: translateY(-2px)`, `box-shadow: 0 6px 18px rgba(232,130,61,0.35)`. On press (active) it compresses to `scale(0.98)` in 100ms (see CT-01). Ghost and WhatsApp buttons do the same lift but only shift `border-color` (no bg wash).

- **HV-03 — On hover over product card: card lifts 3px and border warms to copper-dim over 150ms using ease-out.**
  - Detail: `transform: translateY(-3px)` — the template's existing behavior, keep it. Border-color goes `--line → --copper-dim`. The two 6px top-dot pseudo-elements flip from `--line` to `--copper` over the same 150ms. Shadow stays flat (no big drop shadow — keeps the technical/grid aesthetic).

- **HV-04 — On hover over product thumbnail: image zooms from scale 1.0 to 1.06 over 400ms using ease-out-expo.**
  - Detail: Transform on the `img` only, inside `overflow: hidden` `.thumb` (170px fixed). On leave, settles back over 350ms. Add `will-change: transform` on hover-enter, remove on hover-leave. Never scale past 1.06 — product photos must stay honest.

- **HV-05 — On hover over circular add button (+): button fills copper and glyph flips to ink over 150ms using ease-out, with a 2px lift.**
  - Detail: `background: transparent → copper`, `color: copper → ink`, `transform: translateY(-1px)`. Focus-visible gets a 2px signal outline (no motion). This is the highest-frequency hover on the page — keep it under 150ms so rapid shopping feels snappy.

- **HV-06 — On hover over category chip: pill border warms to copper and text brightens over 150ms using ease-out, with no movement.**
  - Detail: Deliberately static position — chips must not jump layout when hovered. Active chip holds copper border + paper text persistently (see CT-02 for the select motion).

- **HV-07 — On hover over why-card: icon ring rotates 12° and border warms to copper over 250ms using ease-out-expo; card background lifts one shade over 200ms.**
  - Detail: `.ico` does `transform: rotate(12deg) scale(1.05)`. Number in corner (`01–04`) brightens from copper-dim to copper. Subtle — this section sells trust, not excitement.

- **HV-08 — On hover over floating WhatsApp button (FAB): button scales to 1.08 and emits a soft green ring pulse over 250ms using snap-back.**
  - Detail: `transform: scale(1.08)`, ring is a `box-shadow: 0 0 0 8px rgba(37,211,102,0.25)` blooming from 0. Holds at rest after. Idle state has a slow 3s breathing pulse at 4% scale (see AM-02) — hover overrides it.

- **HV-09 — On hover over cart button in nav: border warms to copper over 150ms; cart count badge does a tiny 1.1× nudge over 150ms using snap-back.**
  - Detail: Teases the pop animation (CT-03) without firing it. No lift — nav must stay rock steady.

- **HV-10 — On hover over footer links: text slides 4px right and warms to copper over 180ms using ease-out.**
  - Detail: `transform: translateX(4px)`, `color: dim → copper`. Small, precise, circuit-like.

---

## 4. Click / Tap Transitions

Touch targets: minimum 34px (add-btn) to 56px (FAB). All presses acknowledge within 100ms.

- **CT-01 — On press of any button: button compresses to 98% scale over 100ms using ease-out, then springs back over 200ms using snap-back on release.**
  - Detail: The template already has `.btn:active { transform: scale(0.98) }` — extend it to `.add-btn`, `.chip-btn`, `.qty-btn`, `.fab`, `.wa-order-btn`. Scale from center. Disabled buttons (empty-cart order button at 40% opacity) have no press motion.

- **CT-02 — On tap of a category chip: tapped chip's border snaps to copper over 150ms using snap-back with a 1.04× pop, while outgoing cards fade and新 incoming cards stagger in.**
  - Detail: Full sequence: (1) chip pops `scale(1.04) → 1.0` over 200ms snap-back; (2) current grid fades to 0 opacity and drops 8px over 180ms ease-out; (3) new filtered grid rises from 12px below with 70ms stagger per card over 400ms ease-out-expo. Total filter swap lands inside ~600ms. Grid container height is locked during swap (measure → set `min-height`) so the page does not jump. If filter yields same set, skip step 2–3.

- **CT-03 — On tap of add-to-cart (+): button bursts with a 1.25× pop and copper flash over 200ms using snap-back, cart badge pops to 1.3× and ticks +1, and a toast slides up.**
  - Detail: Sequence: (1) `add-btn` fills copper, scales `1.0 → 1.25 → 1.0` over 200ms; (2) a 12px copper dot flies from button to cart icon along a curved path over 550ms ease-in-out then fades (desktop only; on mobile skip the fly, just pop the badge); (3) `.cart-count` badge pops `scale(1.3)` with spring-soft over 250ms and number rolls up; (4) toast (see CT-04) confirms. Cart persists to `localStorage` silently — no motion for storage.

- **CT-04 — On add-to-cart: toast slides up from 20px below and fades in over 250ms using ease-out-expo, holds 1800ms, then slides down 12px and fades over 200ms using ease-out.**
  - Detail: `.toast` goes `translateY(20px), opacity 0` → resting at `bottom: 96px, right: 24px`. Rapid adds reset the 1800ms timer (template already does this) and re-pop the toast with a 1.02× nudge so double-taps are visible. Never stack multiple toasts.

- **CT-05 — On tap of cart button: overlay fades from 0 to 1 opacity over 200ms ease-out while the drawer slides in from 100% right to 0 over 250ms using ease-out-expo.**
  - Detail: Overlay first (0ms), drawer follows at +50ms for a layered feel. Drawer uses `transform: translateX` only. Background body locks scroll (`overflow: hidden`) at drawer-open with no shift (preserve scrollbar gutter). Focus moves into drawer; Escape reverses the sequence.

- **CT-06 — On close of cart (✕, overlay tap, or Escape): drawer slides back to 100% right over 220ms using ease-out-expo while overlay fades out over 180ms, staggered so the drawer leads.**
  - Detail: Drawer starts at 0ms, overlay at +40ms. Cart badge gives a confirming 1.05× settle if quantities changed during the session. On mobile the drawer is a 92vw sheet — same timing, plus a 4px drag-handle hint (see TG-01).

- **CT-07 — On tap of quantity +/− in cart: number ticks with a 6px vertical roll and 1.1× pop over 150ms using snap-back; row subtotal cross-fades to new value over 200ms.**
  - Detail: Increment rolls up, decrement rolls down. At 0 via "−", the row collapses: fades to 0 and folds from 100% to 0 height over 250ms ease-out-expo, then sibling rows glide up 8px over 200ms to close the gap. "Remove" link does the same collapse with a red (`danger`) text flash over 150ms first.

- **CT-08 — On tap of "Send order on WhatsApp": button shows a sending state — label cross-fades to "Opening WhatsApp…" with a 3-dot shimmer over 300ms — then hands off to wa.me; on return the drawer stays open with cart intact.**
  - Detail: No fake success checkmark (order completes in WhatsApp, not on site). Button compresses to 0.98 over 100ms on press, holds sending state minimum 600ms so the tap feels registered even on fast networks. Disabled empty state has zero motion.

- **CT-09 — On tap of anchor links (Browse the shop ↓, nav links, footer links): page smooth-scrolls to target over 700ms using ease-in-out, with the target section header arriving already mid-reveal.**
  - Detail: `scroll-behavior: smooth` (template default) with a `-72px` offset so the sticky header never covers the section title. The ↓ arrow in "Browse the shop" nudges down 4px over 150ms on tap as a directional cue. Active nav link underline follows scroll position (scrollspy) with a 200ms slide.

- **CT-10 — On tap of floating WhatsApp button: FAB compresses to 0.92 scale over 100ms then springs to 1.0 over 250ms snap-back while opening the wa.me chat in a new tab.**
  - Detail: Same press physics as CT-01 but deeper (0.92) since the FAB is large and round. No in-page modal — handoff is instant. Idle breathing pulse (AM-02) pauses for 2s after tap so the spring landing reads cleanly.

- **CT-11 — On tap of contact-panel CTA ("Message 0758 926 764"): button lifts and glows copper for 300ms (HV-02 pattern) then hands off to WhatsApp; panel border holds a 600ms copper afterglow fading back to line color.**
  - Detail: Afterglow is `border-color: line → copper (40%) → line` over 600ms ease-out. Signals "message sent" without faking delivery state.

---

## 5. Touch Gestures (Mobile-First, Kampala = Majority Mobile)

All gestures use touch-action hints so vertical page scroll is never hijacked.

- **TG-01 — On swipe-left-to-dismiss on the cart drawer (touch): drawer follows the finger 1:1 via translateX with no easing during the drag, then either snaps shut or springs back.**
  - Detail: Drag right-to-left past 96px or flick faster than 0.5px/ms → drawer completes exit over 220ms ease-out-expo and overlay fades 180ms (CT-06 timing). Below threshold → drawer springs back to 0 over 300ms using spring-soft (`stiffness 320, damping 28`). Only the drawer's top 24px handle zone + header initiate the drag; item rows scroll vertically undisturbed (`touch-action: pan-y` on `.drawer-items`, `pan-x` on handle). Overlay opacity tracks drag progress 1:1 (55% → 0%).

- **TG-02 — On horizontal swipe across the category chip row: chips scroll with momentum and soft edge bounce over 350ms using ease-out-expo.**
  - Detail: `#chipRow` is `overflow-x: auto` with `-webkit-overflow-scrolling: touch`, `scroll-snap-type: x proximity`, hidden scrollbar. Flick imparts native momentum; hitting either end overshoots 8px then settles over 250ms snap-back (iOS rubber-band preserved, Android emulated with a 1.02× scale guard). Chips never wrap on mobile — single scrollable rail.

- **TG-03 — On horizontal swipe across the product grid (mobile < 520px): grid behaves as a snap carousel — one card per view peeking 24px of the next — settling with a 300ms ease-out-expo snap.**
  - Detail: Desktop keeps the `auto-fill minmax(240px,1fr)` grid with no swipe. Under 520px, grid switches to `display: flex; overflow-x: auto; scroll-snap-type: x mandatory`; each `.card` is 78vw. Swipe velocity above 0.4px/ms advances an extra card. Dots or a thin copper progress bar under the rail fills `scaleX` with scroll progress (transform-only). Vertical page scroll always wins on diagonal swipes (10° slope test).

- **TG-04 — On pinch over a product thumbnail (mobile): image zooms from 1.0 toward 2.0 following finger spread 1:1 with no easing, then eases back to 1.0 over 250ms on release.**
  - Detail: Pinch is inspection-only (no modal, no separate zoom view — keeps the funnel short). Zoom is `transform: scale` on the `img`, clamped 1.0–2.0, anchored at pinch midpoint via `transform-origin` set from touch coordinates. Single-finger drag pans the zoomed image within the 170px thumb mask. Release always settles home over 250ms ease-out. Double-tap toggles 1.0 ↔ 1.75× over 250ms snap-back as an accessible alternative. Desktop hover-zoom (HV-04) is untouched.

- **TG-05 — On pull-to-refresh at the very top of the page (touch): a 28px copper ring spinner stretches from the header, fills over 600ms, then releases to refresh product availability.**
  - Detail: Pull down past 64px at `scrollY === 0` reveals the spinner (`translateY(-28px → 8px)` following finger at 0.5× resistance, ease-out). Past threshold the ring spins linear 800ms/rotation and copy reads "Checking stock…". On release it re-renders prices/availability with the CT-02 grid stagger (400ms) and spinner collapses up over 250ms. If offline, spinner turns `danger` red and text reads "You're offline — showing saved list" with no reload. Never trigger pull-refresh while the cart drawer is open.

---

## 6. Ambient & Micro-Detail (Always-On Life)

- **AM-01 — Continuously: hero eyebrow signal dot breathes — glow pulses between 4px and 10px blur over 2400ms ease-in-out infinite alternate.**
  - Detail: `box-shadow` opacity 0.6 ↔ 1.0. Opacity-only glow; dot scale stays 1. Paused under reduced motion.

- **AM-02 — Continuously: WhatsApp FAB breathes at 4% scale over 3000ms ease-in-out infinite, with its green shadow swelling in sync.**
  - Detail: `transform: scale(1.0 ↔ 1.04)`. Hover (HV-08) and tap (CT-10) override and pause breathing for 2s afterward.

- **AM-03 — Continuously: marquee loops `translateX(0 → -50%)` linear over 26000ms infinite; pauses off-screen and on hover (desktop) with a 300ms ease-out wind-down, not a hard stop.**
  - Detail: Hover-pause lets shoppers read "Same-day delivery in Kampala." Resumes with 300ms wind-up. Matches the template's existing `@keyframes scroll` — keep the keyframes, add the pause behavior.

---

## 7. Accessibility & Fallbacks

- **Reduced motion:** all PL/SC/TG parallax, stagger, pinch-fly, and ambient loops collapse to instant opacity fades (≤1ms). Marquee renders as a static wrapped row. Drawer still slides (functional) but over 1ms. The template's `@media (prefers-reduced-motion: reduce)` block already enforces this — do not add per-element overrides that fight it.
- **Keyboard parity:** every CT hover has a `:focus-visible` twin with a 2px `signal` outline and zero positional motion (no lift on focus — avoids scroll jumping for keyboard users).
- **Contrast in motion:** copper-on-ink flashes never drop below 3:1 during transitions; toast border stays copper at full opacity throughout its slide so it reads mid-flight.
- **Stock/offline:** if product images fail, thumb holds `#0C1119` with a copper `chip` glyph pulsing once (400ms fade) — no broken-image icon, no layout shift (fixed 170px height preserved).

---

## 8. Implementation Checklist for Developers

1. Add `.reveal` (opacity 0, translateY 24px) initial states + `.in` (visible) via IntersectionObserver; stagger with `--i * 70ms` custom property delays.
2. Nav compress: scroll listener toggling `.compressed` at 40px; CSS transitions 300ms on padding/font-size (class toggle, not rAF tween).
3. Parallax layers (hero bg, circuit): rAF + lerp 0.12, `transform` only, disabled < 760px + reduced motion.
4. Drawer/overlay/toast timings already match spec (250/200/250ms) in the template `<style>` — keep them; add the badge spring + fly-dot + drag-to-dismiss JS.
5. Chips rail + mobile grid carousel + pinch-zoom + pull-refresh are new JS (~150 lines total); guard each with `(hover: none)` / width media queries.
6. Validate on a mid-range Android (Tecno/Infinix class): scroll reveals must hold 60fps, drawer drag must track finger with <50ms lag, no `will-change` left on cards after landing.

*End of spec — 11 load cues · 11 scroll effects (no pinning, by design) · 10 hovers · 11 tap transitions · 5 gestures · 3 ambient loops.*
