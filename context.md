# Widforss Shorts — Context

Shoppable vertical-video feed prototype (TikTok-style) for Widforss (Nordic outdoor retailer). Single-file HTML prototypes, green brand palette. `PLAN.md` = production Sanity/Centra/Mux plan.

## Files
- `handoff.html` — **spec/handoff page** (started 2026-07-02), styled to match `../Shop-in-Shop/handoff.html` (`.wrap`/`.head h1` 60px/`.frame-block`/`.label` with olive `.n`/`.canvas`/`.cap`, Greed Standard from local TRIAL .otf). Block 01 = the **"Klipp fra dere" entry module** (Figma frame 1427×744): title "Klipp fra dere" + play triangle, prev/next arrows top-right, 4 cards 318×399 r12 gap 24, each a `<video>` (SnapInsta mp4 as placeholder) with bottom gradient + white name (Greed Medium 28), links to `index-v2.html`, **plays on hover** (mouseenter play / mouseleave pause+reset). Real specs/redlines still TODO (Stefan sending mobile version next).
- `index.html` — original prototype (in-site, white header, right thumbnail rail, white shop drawer).
- `index-v2.html` — **current active variant.** Rebuilt 2026-06-21 to match the new Figma (`new[1896x1179].png` + annotation). See below.
- `index-v2-backup-*.html` — timestamped backups of prior v2 states (the `20260621-150319` one is the pre-Figma-redesign v2 with the right-side expanding panel, size chips, action rail, timestamped tagging).

## index-v2.html — current design (Figma "fullscreen player")
Fullscreen dark `#1F1E1D` shorts player (no site header). Font: **Bagoss Extended as a stand-in for "Greed Standard TRIAL"** (Greed not in /fonts — drop the .otf there and swap the @font-face family + `.si-title` etc.). Rule: never uppercase unless specified.

Layout per Figma:
- Close circle top-right: 40px `#292926`, white X.
- Video: centered, portrait aspect 574/1021, radius 12. Width derived from `--video-h`/`--video-w` CSS vars on `.content-area`; rail & left column positioned relative to it.
- **Site header (2026-07-02): refined Widforss nav ported from `../Shop-in-Shop/handoff.html`.** Main nav row only — dept menu (Herr active olive `#3D4B19` pill / Dame Barn Jakt Friluftsliv Hund), centered Widforss wordmark (inline `#ic-logo` SVG, 140×36, olive), right: search pill (405×38, r6, `--search-bg`, icon-left + input) + user icon + cart icon w/ green `#134B2B` badge. Deliberately **excludes** the olive promo/free-shipping banner, the mega-menu, and the sub-nav row (per Stefan). Icon symbol defs (`ic-logo/search/user/cart`) injected as a hidden `<svg>` right after `<body>`. Hidden on mobile.
- **Mobile nav (2026-07-02):** compact Widforss header (`.mobile-nav`) sticky at top on mobile, ported from handoff — burger left, logo (104×26) centered, cart+badge right, full-width search pill (46px, `#F3F3F3`) below. No promo banner. `padding: 26px 20px 16px` (top padding gives the logo breathing room). Sits above the player; video fills the content-area beneath it.
- **DESKTOP LEFT COLUMN IS UNCHANGED — do not touch.** `.shorts-info` stays left-anchored at `clamp(40px,4.3vw,81px)` / `top:33%`, aligned with the "Herr" top-nav item. `.vp-card-wrap` stays bottom-left. (A 2026-07-02 attempt to right-anchor these to the video was WRONG and reverted — the "void" complaint was about **mobile**, not desktop.)
- **Mobile text position (2026-07-02):** `.shorts-info` sits at `bottom:40px` (was 150px → floated mid-air). Single-product: `.shorts-feed.has-product` (toggled in `onSlideChange`) lifts it to `bottom:218px` = 40 + card 150 + 28px gap, so text sits **28px** above the card. Card-dismissed drops it back to 40px. The "Produkt fra video:" label (`.vp-card-label`) is **hidden on mobile** (`display:none`); it still shows + animates on desktop.
- **Shop panel mobile fix (2026-07-02):** the desktop `4.3vw` side padding was clipping cards on the 390px frame. Mobile now uses fixed 20px padding, full-bleed scroll row (`margin:0 -20px` + `padding:0 20px`), `scroll-snap-type:x`, and cards at `flex:0 0 74%` so card 1 aligns with the "Shop the products" heading and card 2 peeks for swipe affordance.
- Left column (`.shorts-info`, top:33% when no product): title 36px/115%, time 18px `#919191`, tag chips (64×24, r6, 1px white border, 14px). For **multi-product** videos a white "Shop all products" button (170×36, r12, 16px) sits below the chips.
- **Single-product** videos: bottom-left "Produkt fra video:" + white card 330×150 r12 (thumb 89×115 `#B8B8B8`, brand+name, "Gratis frakt!", price, green `#42541F` "Legg til", × to dismiss).
- Right rail: two `#3C3C39` arrow buttons (41×44 r12) + a `#292926` pill holding a **three-dots menu** (opens Lagre/Del — save+share hidden here, TikTok-style) and mute.
- **Shop-panel scrim (2026-07-02):** `.shop-scrim` (z-index 56, between chrome/video at ≤55 and panel at 60) fades to 0.55 black on `.panel-open`, so the **black overlay now covers the video too**; click scrim closes the panel. Close-X (70) stays above.
- **Shop the products panel: horizontal, slides up from the bottom** (`.shop-panel`, translateY). Header "Shop the products" + chevron-down to collapse. Horizontal scroll row of `.sp-card` (thumb 64, name, gray price, green "Legg til"). Opening dims the left column + rail and nudges the video up.
- Toast feedback ("Lagt i handlekurv", "Lagret", "Lenke kopiert").
- Timestamped product tagging from the prior v2 was **removed** (kept lightweight per Stefan; production Mux timing TBD).
- Data: each video has `tags[]` and `products[]` (with `img`, `freight`). Products use new images `skoo.png` (boot), `caps.png` (cap), `harkilaa.png` (jacket); `mat.webp` for the waffle iron.
- Mobile mode (device toggle): basic adaptation only — desktop is the focus.

## Local server
`python3 -m http.server 8000` → http://localhost:8000/index-v2.html
Verify states with Chrome headless: append `#panel`, `#single`, `#menu` to a temp debug hook if needed (removed from shipped file). Use a separate `--user-data-dir`, `--virtual-time-budget≈2500`; long budgets hang.
