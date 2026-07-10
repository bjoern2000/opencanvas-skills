---
name: opencanvas-prototype
description: Build and iterate mobile & desktop prototypes for human review over the opencanvas MCP. Use when a PM or builder wants a shareable, comment-able prototype that an agent can revise from feedback — "prototype this flow", "make a clickable mockup", "let stakeholders comment on this screen", or any request that pairs UI generation with a review loop.
---

# opencanvas — authoring & iterating mobile & desktop prototypes

opencanvas is an agent-first, MCP-native prototype-review surface. You author a multi-page mobile or desktop prototype (one HTML/CSS/JS document per screen); humans open a share link, review each page in a device frame, and pin comments to specific elements. You read those comments, regenerate the page, and the comment anchors survive the regeneration. The loop — **comment → you iterate → anchors persist → resolve** — is the product.

## When to trigger
- The user wants a mobile or desktop prototype/mockup others will review and comment on.
- The user wants an agent to act on prototype feedback (not a one-off image).
- The user references opencanvas, a "concept", a share link, or the review loop.

Not for: static screenshots or slide decks (that's Deckpipe). Both mobile and desktop form factors are supported — pick with `device` at create time.

## Before you build — ask, then calibrate

If the brief is thin, ask up to a few sharp questions before committing:
1. **Audience** — who reviews this (PM, execs, users)? Sets fidelity.
2. **Fidelity** — rough grey-box wireframe, or polished hi-fi with real content and color?
3. **Flow scope** — which screens/steps? (e.g. Menu → Detail → Cart → Confirm.)
4. **Reference** — an app whose feel to match (Linear, Things, Cash App, Airbnb…)?

Then **calibrate one page first**: author ONE representative content-carrying screen (not the title/splash), `preview_page` it, and READ the screenshot — this is the one screenshot you must look at. Get that screen the way you want it before building the rest at the same bar; it stops you propagating a rendering mistake across every screen.

**Then draft fast and hand off — don't over-verify.** After creating all screens, do a *cheap* health sweep: check `report.ok` per page (it's a boolean — you do NOT need to open every screenshot). Look at a screenshot only when `report.ok` is false or a screen seems off; fix real `off_canvas`/JS-error/broken-image issues, then **share the link.** opencanvas is a human-review tool: the reviewer is the better judge of "is this good," and the comment→iterate loop is the product. Getting a not-broken prototype in front of them quickly beats perfecting every pixel first. Save the deep work — reading full screenshots, `{expect}` interaction assertions — for **iteration**, where you verify the specific thing a comment is about (not the whole app upfront).

## Building for the device frame

Design taste (layout, hierarchy, what's primary, spacing) is yours and the user's call — this section is only the frame/tool mechanics you can't infer.

- Choose the form factor at `create_concept` with `device: "iphone" | "android" | "desktop"`. Design at the device width (iPhone 390×844, or Android 412×915). Vertical scroll is normal — screenshots capture the full height.
- **Desktop** (`device: "desktop"`): author at 1280×800. The viewer draws a minimal window bar (three traffic-light dots — no URL bar, tabs, or title) and auto-insets content ~34px below it via `--cc-safe-top`; there's no status bar / home indicator (`--cc-safe-bottom` is 0). Horizontal overflow past 1280 is still an `off_canvas` bug; `class="cc-bleed"` still pulls a hero up under the bar. `data-cc-navbar` / `data-cc-statusbar` are mobile-only and do nothing on desktop.
- **Don't draw the phone chrome — the viewer overlays it.** No status bar (time/signal/battery), no home indicator. Your content is auto-inset into the safe area (via `--cc-safe-top` / `--cc-safe-bottom`); the body background bleeds under the chrome. The viewer **auto-detects** the background behind the status bar and home indicator and picks glyph contrast for you (dark bg → light glyphs) — you usually don't need to set anything. To override, put `data-cc-statusbar="light"` (or `"dark"`) on any element. To set the screen background that bleeds under the chrome, style `body` (in the concept stylesheet or a `<style>` tag in the page).
- **Full-height screens with a bottom action bar**: wrap the screen in `<div class="cc-screen">` (a provided safe-height flex column) and give the action bar `margin-top:auto`. Don't use `position:sticky`/`fixed` for it — the screenshot captures the full scroll height, so a fixed bar floats mid-page. For a top hero that should run edge-to-edge under the status bar, add `class="cc-bleed"`.
- **Persistent bottom tab bar** (stays put while content scrolls): mark it `data-cc-navbar` (not `position:fixed`) — the viewer pins it in the frame above the home indicator, and content clears it automatically. It's lifted out of flow (margin/position on it are ignored) and pins even on a short page. Don't *also* sticky/fixed-pin an action bar on the same screen — they'd overlap; inline any action bar in the content.
- **Make it a flow**: add `data-cc-link="<page name or 1-based number>"` to tab-bar items and CTA buttons — tapping one navigates the viewer to that page, so reviewers walk the real flow instead of isolated screens. `create_concept`/`update_page`/`get_concept` return a `flow` graph — check `flow.broken_links` (typo'd/renamed refs) and `flow.unreachable_pages`.
- **Images**: prefer `search_images("query")` for real photos over guessing image IDs — returns ready `url` + `attribution_html` + a stable `id` + `width`/`height`, and takes a `crop` hint (e.g. `crop:"faces"`). External `https`/`data:` URLs also load; `report.images_unloaded` flags any that didn't paint.
- **Assets & decorative bleed**: external `https` image URLs and `data:` URIs both load (no image proxy — use full URLs); emoji and CSS gradients work. Mark purely decorative bleed (background leaves/glows inside `overflow:hidden`) `data-cc-decorative` so it (and the container it bleeds inside) doesn't trip the overflow linter. Check `report.images_unloaded` — the capture waits for images, and any that failed to paint show up there.
- **Typography as tokens**: define `--font-display` and `--font-body` as concept `tokens`; `--font-body` auto-applies to body text and you use `var(--font-display)` on headings. To restyle fonts concept-wide later, PATCH the token via `update_concept` (+ swap the head font link) — no need to resend the whole stylesheet.
- Horizontal overflow past the device width is a bug (`report.overflows`, reason `off_canvas`) — EXCEPT inside an intentional `overflow-x:auto/scroll` carousel, which is not flagged.

## Anchor discipline (this is what makes the product work)
Every meaningful element gets a **kebab-case `data-anchor-id`**: headings, buttons, inputs, cards, list items, prices, key sections.

```html
<button data-anchor-id="checkout-cta">Checkout</button>
<div data-anchor-id="cart-item-oat-latte"> … </div>
<span data-anchor-id="summary-total">$11.33</span>
```

Rules:
- **Preserve** existing anchor ids across every edit. Mint new ids only for genuinely new elements. Never rename or renumber an id that has comments.
- ids must be unique and kebab-case (`[a-z0-9-]`). The render report's `anchor_warnings` flags duplicates and malformed ids — fix them.
- Prefer `update_element` (replace one element by its anchor id) for small changes — it's cheap and inherently anchor-safe.
- Every `update_page`/`update_element` result has a `warnings` array. If it says an anchor with open comments disappeared, **re-add that `data-anchor-id` in the same turn** — otherwise the comment falls back to fuzzy re-resolution and may unanchor.

## Reading the render report
`preview_page` and `get_page_screenshot` return the screenshot AND a report. Read both — the screenshot is ground truth.
- `overflows` — `off_canvas` (horizontal, a bug) or `clipped` (content cut by an overflow box). Fix these.
- `page_height_px` — total scroll length (informational; vertical scroll is fine).
- `js_errors`, `console_errors`, `failed_requests`, `fonts_missing` — real problems to fix.
- `anchor_warnings` — duplicate/malformed anchor ids.
- `anchors` — the manifest (id, tag, text, region) for anchor-coverage checks.

## Verifying interactivity
The screenshot is static, but many comments are about behavior ("make the chips filter the list"). Drive it: `preview_page` with `interactions` runs actions before capture, so the screenshot + report show the resulting state. Verbs: `{click}`, `{scroll_to}`, `{type:{target,text}}`, `{key:"Enter"}`, `{wait_for:"anchor"|ms}`, and `{expect:{anchor|selector, visible/text_contains/has_class/count/css}}` to **assert** the outcome (it returns the actual value + element rect; steppers = repeated clicks on +/–). Then read `report.ok` (page **health**) and `report.assertions_passed` (did your `{expect}`s hold — **separate** from health, so a wrong expectation isn't a broken page). `report:"summary"` trims the payload; `viewport:"fold"` captures only above the fold.

## The review loop
1. `get_page` → read the human-written **brief** first (it's their channel to you). It also returns the effective concept `stylesheet`/`tokens`/`device`, so you can restyle without a separate `get_concept`. Read `list_comments` (defaults to open).
2. Address the feedback: `update_element` for targeted changes, `update_page` for a full rewrite. Preserve anchors.
3. `get_page_screenshot` → verify it actually looks right.
4. `reply_to_comment` — concise: say what you changed, don't repeat the feedback.
5. `resolve_comment` — ONLY after you've actually addressed it (and ideally the human confirms).

Never recreate a concept to edit it (that loses the URL and all comments) — use the update tools.

## Handoff
`get_annotations` returns the full machine-readable export — every thread with its anchor, the version it was raised against, and the version it was resolved in. This is the contract a downstream spec/ticket agent consumes; point that agent at it.
