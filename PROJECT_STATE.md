# Karthikeya Burugula — Portfolio Site: Project State

Resume file for continuing work in a new chat. Attach this file and say **"continue from here."**

Last updated: 2026-10-07 · HEAD `c74191e` (live on GitHub Pages)

**Read "Latest session" (below) first — it supersedes older sections wherever they disagree.**

## Latest session (2026-10-05 → 2026-10-07) — commits `7fa34fc` … `c74191e`
Many small, tightly scoped rounds. Mid-session user verdict: *"the animations and everything look perfect now."* Everything below is **live** and user-approved unless marked.

### 1. Cards had no entrance animation — fixed (`7fa34fc`)
Cards now replay the Skills/Recent Work reveal, both scroll directions. **Root cause was a CSS specificity trap**, confirmed by measuring computed styles: `body.gamer-mode .p-card, .skill-card, .edu-card, .arcade-card` set `transform` for the tilt and its own `transition`, overriding `.reveal`'s. Fix: the tilt moved to the individual **`rotate:var(--tilt,0deg)`** property (independent of `transform`), plus an explicit transition list so opacity/transform/filter use `var(--delay)` while `rotate`/shadow/border don't:
```css
body.gamer-mode .p-card, … .arcade-card{
  rotate:var(--tilt,0deg);
  transition: opacity .6s cubic-bezier(0.34,1.56,0.64,1), transform .6s …, filter .6s …,
              rotate .25s …, box-shadow .25s ease, border-color .25s ease;
  transition-delay:var(--delay,0s), var(--delay,0s), var(--delay,0s), 0s, 0s, 0s;
}
body.gamer-mode .p-card.visible:hover, .skill-card.visible:hover, .edu-card.visible:hover{
  rotate:0deg; transform:translate(-3px,-3px); box-shadow:11px 11px 0 #000; border-color:var(--a1); …
}
```
(each selector is prefixed `body.gamer-mode`.) **Never put the tilt back on `transform`.** Arcade also got `.reveal`: `<div class="section-head reveal">`, `.arcade-grid.stagger`, both `.arcade-card.reveal` (now **14** `.reveal` elements, 4 `.orbit-card`, 4 `.p-slot`).

### 2. Landing screen (web, ≥821px) — what the fold shows
On load the user sees only: name, intro, **Get in Touch** + **LinkedIn** CTAs, and the "Recent Work" heading + subtitle. The first card stays hidden until the first small scroll so its entrance plays (animations work both ways). Implemented as `.hero{min-height:calc(90vh - 283px)}` + `#highlights{padding-top:62px}` inside `@media (min-width:821px)`. The 283 was found by **measuring with `.visible` forced and transitions off** — a hidden `.reveal` section-head's transform skews `getBoundingClientRect` (constant went 250 → 318 → 255 → 283). Mobile unchanged (`#highlights{padding-top:40px}`, `.hero{padding:44px 0 18px}` ≤640).
- This **supersedes** the third-pass note that `min-height` was removed — it is back, but only ≥821px and only to position the fold; the real entrance fix is still the two-rAF observer attach.

### 3. Hero / copy / colour changes
- Name shadow softened: `body.gamer-mode .hero h1{text-shadow:3px 3px 0 #3d2a99}`. "Burugula" and the violet theme kept.
- `.hero .sub{color:#fff}` (intro white). `.hero-role{color:#a994ff}` ("Product Manager at AssetPlus", lightened).
- Intro copy: "PM by day" → **"Product Manager by day"**. (Still says "4 years" vs Career "4.5 years" — flagged, unresolved.)
- **Glow cut fixed**: `.hero{overflow:hidden; overflow-x:clip; overflow-y:visible}` (the `hidden` is a fallback for old browsers) — the left/right purple hues no longer end in a straight vertical line. `body` keeps `overflow-x:hidden`.
- Section-title triangle (▸) gap: `body.gamer-mode .section-head h2::before{margin-right:0.55em}` — user-tuned (0.9 too big → 0.72 → **0.55em final**). Applies to every title.

### 4. CTAs
- **"See Recent Work" removed.** Hero CTA row is now **Get in Touch** (`btn btn-primary`, violet, tactile press, `tel:+918296114865`) + **LinkedIn** (`btn btn-ghost btn-li`).
- **LinkedIn CTA**: mixed-case label (`.btn-li{text-transform:none}`), inline LinkedIn glyph SVG (`.li-icon`, `fill=currentColor`) inside the button, links to https://www.linkedin.com/in/karthikeyaburugula/ (`target=_blank rel="noopener noreferrer"`). Style: `body.gamer-mode .btn-ghost.btn-li{border-color:var(--a1);background:#3a3845}` — violet border, **solid** gray fill (an 8% white tint still read as transparent and was rejected). The user's "use this format" had no attachment; mixed-case "LinkedIn" was assumed and disclosed.

### 5. Nav pair (Open-to-roles pill + Contact) — final state
Applies to mobile **and** web.
```css
nav .nav-pill, nav .cta{height:30px;box-sizing:border-box;display:inline-flex;align-items:center;justify-content:center;}
nav .cta{height:28px; text-transform:uppercase; …}      /* CONTACT, all caps, slightly shorter */
nav .nav-pill{margin-bottom:0;flex-shrink:0;font-size:12px;font-weight:700;letter-spacing:0.3px;padding:0 18px;gap:8px;}
@media (max-width:400px){ nav .cta{font-size:11px;padding:0 14px;height:25px;} nav .nav-pill{font-size:11px;padding:0 14px;height:27px;} }
body.gamer-mode nav .nav-pill{border:0;background:rgba(255,255,255,0.12);color:#fff;}
body.gamer-mode .eyebrow-pill .dot{background:#39ff14;box-shadow:0 0 6px #39ff14, 0 0 14px rgba(57,255,20,0.75);} /* fluorescent green */
```
**Root cause of "pill still not the same size"** (equal height/font weren't enough): the gamer `.eyebrow-pill{border:2px solid #000}` is invisible on the dark nav but still eats 4px, so the visible fill looked smaller. Fix = remove the border in the nav and fill the background. Don't "fix" this with equal widths — that was tried (grid) and **rejected**; the Contact CTA's own width stays normal. This supersedes the 34px/30px note in the 2nd-pass section.

### 6. Cursor "lagging" — fixed (`c74191e`)
Not a perf problem: page holds ~60fps with every suspect (grid, nav blur, glow, sparkles) disabled one at a time; trail JS costs ~0.12ms/frame. Real cause: the ribbon head eased toward the pointer at 0.22/frame, leaving the glowing tip **~46px behind the real cursor** (up to ~75px) on a normal swipe. Fix: `const HEAD_EASE = coarsePointer ? 0.22 : 0.75;` — **desktop only**; touch is unchanged. Measured gap on a synthetic swipe: avg 4.5px, p95 6.9px, max 9px; ribbon turn angle ~12° (unchanged). If it feels too tight/loose, `HEAD_EASE` is the single knob (higher = tighter). Whether it *feels* right is only judgeable live.

### 7. Sizing rule that bit us
`getBoundingClientRect` returns **transformed** boxes → use `offsetWidth/offsetHeight` for canvas/board sizing. `fitStack` now uses `stack.offsetWidth/offsetHeight`; Ship It `spawn()` uses `shootBoard.offsetWidth/offsetHeight`, so a not-yet-revealed (scaled/rotated) Arcade card no longer corrupts the games.

### Verification done each round (and still the checklist)
Tag + brace balance; zero JS errors; counts 14 reveal / 4 orbit-card / 4 p-slot / 1 arc-path; trail canvas `display:block`; no horizontal overflow at 390 and 360 (fixed-width iframe wrapper); desktop + mobile screenshots read; then push, poll Pages `built`, `curl` the live URL for a changed string. **A Node CDP (Chrome DevTools Protocol) driver is the tool for real-time checks** (fps, cursor gap) — `--virtual-time-budget` can't do those. Headless can't scroll for real; feel of animation/cursor needs a live look.

### Other lessons from this session
- `sed -i` on macOS needs `-i ''` — use Python for edits.
- A `rm -f …/*.html` cleanup in the scratchpad was blocked by the safety check; it blocks the *whole* compound command, so keep checks and cleanup in separate calls.
- Scratchpad (`/private/tmp/claude-501/-Users-karthikeya/…/scratchpad/`) still holds temp test files; harmless.

---

## Reimagined site ("The Factsheet") — 2026-10-07, lives at `/v2/` (NOT promoted to root)
User asked for a complete reimagining using prashanthnimmagadda.vercel.app only as a reference for technique/UX/positioning. Built as `v2/index.html` (single file) so the approved root site is untouched. Promote by copying `v2/index.html` over `index.html` and changing `../assets/` to `assets/` (assets already live in root `assets/`).
- **Concept:** mutual-fund factsheet / market terminal (fits AssetPlus MF/PMS/SIF work). Dark default + light toggle (`data-theme`, `kb-theme` in localStorage). Fonts: Bricolage Grotesque / Figtree / JetBrains Mono via Google Fonts.
- **Sections:** hero (live NAV chart behind, count-up stats) → ticker → 01 Overview (objective, fund details, riskometer) → 02 Performance (scrubbable career chart, role tabs + panels) → 03 Holdings (accordion w/ before/after bars) → 04 Toolkit (heatmap tiles; tile sizes are Claude's guess) → 05 Credentials → 06 Arcade (Ship It + Stack ported unchanged apart from colours) → 07 Contact + CV card.
- **Cursor:** candlestick trail (green up / red hollow down) on a fixed canvas + ring follower (mouse only). Touch uses passive touch events + scroll-momentum candles (same lesson as before: no pointer events for touch).
- **Rules kept:** no animation libs; reveal = IntersectionObserver attached after two rAFs, bidirectional, using the `translate` property (not `transform`); photo is straight with nothing over it.
- **Resume:** `assets/Karthikeya_Burugula_CV.pdf` is a copy of `~/Downloads/Karthikeya_CV_AB.pdf` (user said ONLY use the AB file; V0 was used briefly by mistake and replaced). `assets/cv-preview.jpg` is its page-1 render. All numbers on v2 come from that CV or the previously approved site copy. The chart's y-axis is an *illustrative* "scope index", labelled as such.
- **Verified:** CDP driver at 1440 and 390 (touch emulation): zero JS errors, no horizontal overflow, scrub/tabs/accordion/theme/menu/games all exercised.

- **Round 2 (same day):** ambient ticks ~2x more frequent; section padding cut (`.sec` 136→104px max); Ship It replaced by **Flappy Bull** (canvas, time-based; constants researched from Flappy Bird clones: gravity ~1000px/s², flap -335px/s, scroll 130→215px/s, bear-candle pipes, price-line tail, best in `flappyBest`); **Stack** rewritten so tower is theme green only, a clean landing prints a green rising market line + "▲ +x% xN", a slice/miss prints a red falling line + "▼ -x%" (old chunk-shatter removed); ring cursor removed → **trading-arrow pointer** (native cursor hidden on fine pointers via `html.cx`) with hover tags (BUY/OPEN/SCRUB/FLAP…), turns red while pressed, and **3 selectable trails** in the footer picker (`kb-cursor`): Bull candles (default), Ticker zigzag price line, Terminal crosshair with ₹ price + time tags. Hero CTAs = Get in touch + LinkedIn only; nav brand = "Karthikeya Burugula"; kicker/status = "Open subscription"; photo top-aligned with name; Overview is now a 2x2 grid (objective|facts, quote|riskometer, equal heights); contact title on one line, CV preview card removed (`assets/cv-preview.jpg` deleted), Download CV button kept.

## What this is
Single-file static portfolio website for Karthikeya Burugula (Product Manager). No build tooling, no frameworks — HTML, inline `<style>`, inline `<script>`, all in one file (~2,080 lines). Dark/gradient brand aesthetic inspired by ultrahuman.com/in. Scroll-triggered reveal animations throughout. The whole site runs in a single **Gamer** theme — the PM/Gamer toggle was removed on 2026-09-29 and `<body class="gamer-mode">` is now hardcoded.

- **File:** `/Users/karthikeya/projects/portfolio/index.html` (the entire site)
- **Live URL:** https://karthikburugula46-source.github.io/karthikeya-portfolio/
- **GitHub:** https://github.com/karthikburugula46-source/karthikeya-portfolio
- **GitHub Pages:** legacy `build_type` (not Actions-based)

---

## Standing rules (do not violate)
1. **Never break or alter animations/transitions unless explicitly asked.** Violated once before (see incident) — the user is strict. Hard guardrail on every edit, even padding tweaks.
2. **Scope discipline** — only touch what's explicitly requested. "Only this section" means only that section's animation, spacing, and structure.
3. **No external animation libraries.** GSAP/ScrollTrigger silently failed on the user's real device. Removed entirely; replaced with vanilla `IntersectionObserver` + CSS transitions. Never reintroduce GSAP or any CDN-dependent animation library.
4. **Verify before every push** (see Verification workflow).
5. **Don't blame caching for a bug.** Measure first. This was a real mistake once: a mobile misalignment was attributed to cache when in fact dot markup had only been reverted on 3 of 4 career items.
6. Git commit trailer:
   ```
   Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
   ```
7. **Vercel is abandoned** — "Skip Vercel, GitHub Pages is enough. Do not revisit unless asked." (On 2026-10-07 the user pasted `prashanthnimmagadda.vercel.app` — someone else's live portfolio, HTTP 200 — apparently as a reference; it is not this project and nothing was changed for it.)
8. **Attach IntersectionObservers after two `requestAnimationFrame`s, never synchronously.** See Animation architecture.
9. **Don't ship the first plausible diagnosis.** Two animation bugs in a row were "fixed" from a story that was never A/B-tested against an alternative, costing the user extra rounds. Isolate one variable and measure both states of it before editing.
10. **Deploy without being asked.** After a verified change: commit, push, poll the Pages build to `built`, `curl` the live site for a changed string. The user once replied "nothing has changed on the output. Did you even deploy it?" — never just offer to push.
11. **"Don't touch anything else" is literal.** The user repeats "make sure everything already fixed stays the same" every round. Change only the named thing; re-verify the reveal/orbit counts after.

## Critical incident (why rule #3 exists)
User reported: "You screwed up the entire portfolio page... You removed the skill section... top highlights..." Root cause: GSAP/ScrollTrigger (CDN-loaded, plus complex 3D transforms and custom `toggleActions`) silently failed on the user's device, leaving `.reveal` elements stuck at `opacity:0` — looked exactly like deleted content, but the HTML was intact. Fix: removed GSAP, replaced with vanilla `IntersectionObserver` toggling a `.visible` class driving plain CSS transitions. Permanent architecture.

---

## Page structure (current, in DOM order)
Hero → `#highlights` (Recent Work) → `#arcade` (games) → `#skills` → `#career` → `#background`

Section-by-section:
- **Nav "Contact" CTA and hero "Get in Touch" both `href="tel:+918296114865"`** — they dial directly instead of scrolling to the footer (works in PM and gamer mode, mobile and desktop). The footer `#contact` section still exists with the mail + phone row.
- **Hero** — two columns (photo left; name, "Product Manager at AssetPlus", intro, CTA row right; stacks at 820px). CTA row = **Get in Touch** + **LinkedIn**. The availability pill lives in the **nav** (`.nav-right`, next to CONTACT), not the hero; the PM/Gamer toggle no longer exists.
- **`#highlights`** — titled **"Recent Work"** (the "Impact" eyebrow was removed). 4 cards in a `.bento` grid. Cards are **purely informational**: no CTA, no click target, no hover pop-up. Content is placeholder — **the user will supply a PRD with the real copy.**
- **`#arcade`** — two games: **Ship It** (`#shootBoard`) and **Stack** (`#stackCanvas`). Stack replaced an earlier doodle-pad idea.
- **`#skills`** — 3 cards only: Product Management, Analytics & Data, Prototyping & Tools. New AI tool pills go inside "Prototyping & Tools" only, formatted "Tool (Purpose)". Skills sits **above** Career (deliberate reorder).
- **`#career`** — arc timeline, 4 entries. The two internships (Upliance.ai, Accredian) are merged into one **"Product Internships"** entry, Mar 2023 – Oct 2023, with the timeline under the title and then Upliance first, Accredian second.
- **`#background`** — 2 `.edu-card` items.
- Weekend Overhaul / `#work` section was **removed entirely**.
- Gradient pills use **white** text (not black).
- Section padding: desktop `70px 0`, mobile (≤640px) `42px 0`. Do not change unless asked.

---

## Animation architecture

**Global reveal:** `.reveal` starts hidden; `revealObserver` (index.html:~1183) toggles `.visible` bidirectionally on `entry.isIntersecting`, so scrolling back up reverses it.

**Observer attach timing is load-bearing — never make `observe()` synchronous again.** Both `revealObserver` and `orbitObserver` are attached inside **two nested `requestAnimationFrame` calls** at the end of the script:
```js
requestAnimationFrame(()=>requestAnimationFrame(()=>{
  document.querySelectorAll('.reveal').forEach(el=>revealObserver.observe(el));
  document.querySelectorAll('#highlights .p-slot').forEach(el=>orbitObserver.observe(el));
}));
```
Attached inline during script parse, anything already on screen at load goes from *never rendered* straight to `.visible` inside a single style pass — there is no painted start frame to interpolate from, so the transition never runs and the entrance just pops. Two frames guarantee the hidden state is painted first. **This, not hero height or fold position, is why entrances looked "completely gone."** A/B on the same file: sync attach → `raf2(paintedHidden):vis=4`; deferred attach → `raf2(paintedHidden):vis=0`, then `+80ms:vis=4`.

**Orbit reveal — Recent Work only** (replicates the revolving-card entrance from a reference recording of ultrahuman.com). This must NOT be applied to any other section. **Gamer mode must never animate `opacity`/`transform`/`filter` on `.orbit-card` via a separate `animation`** — an animation always wins over a transition on the same property while it plays, so when a shorter animation ended before the orbit card's own .55s transition did, opacity snapped to whatever the transition had reached mid-swing — a visible break the user caught early on, and the root cause of "stucky" motion a couple of rounds later too (`steps()`-based animations layered on top of transitions). Current gamer mode (see PM/Gamer mode section below) sidesteps the whole hazard class: no separate `animation` on `.reveal`/`.orbit-card` at all any more, just a bouncier `transition-timing-function` on the same transition PM already runs.
```css
.orbit-card{
  opacity:0; transform-origin:center center;
  transform:rotate(var(--orbit-deg,-45deg)) translateX(var(--orbit-dist,200px))
            rotate(calc(var(--orbit-deg,-45deg) * -1)) scale(0.5);
  filter:blur(10px);
  transition:opacity .9s cubic-bezier(0.16,1,0.3,1), transform .9s …, filter .9s …;
  transition-delay:var(--delay,0s);
}
.orbit-card.visible{opacity:1;filter:blur(0);transform:rotate(0) translateX(0) rotate(0) scale(1);}
/* per-card: 1 → -38deg/240px, 2 → 64deg/170px, 3 → -64deg/170px, 4 → 38deg/240px */
```
**Why this works:** CSS interpolates *matching transform function lists component-wise*, so `rotate → translateX → rotate⁻¹ → scale` produces genuine arc motion, not a straight slide.

**The `.p-slot` wrapper is load-bearing — do not remove it.** The first card used to vibrate/stick mid-transition because `IntersectionObserver` uses the **transformed** bounding box; observing the animating element created a self-feedback loop. Fix: observe a stable `.p-slot` wrapper instead, move `grid-column` onto it, and give `.p-card{height:100%}` so grid stretch survives.
```js
const orbitObserver = new IntersectionObserver((entries)=>{
  entries.forEach(entry=>{
    const card = entry.target.querySelector('.orbit-card');
    if(card) card.classList.toggle('visible', entry.isIntersecting);
  });
}, {threshold:0.15, rootMargin:'0px 0px -10% 0px'});
document.querySelectorAll('#highlights .p-slot').forEach(el=>orbitObserver.observe(el));
```

**Career arc timeline** — SVG path + `.arc-node` circles, animated via `getTotalLength()` + `stroke-dashoffset`, driven by a dedicated `arcObserver` (index.html:~1244). Geometry was recomputed in Python for 4 unequal-height items: circle center (292.47, 287.292), R = 272.47. Item tops: 2.77%, 24.14%, 45.52%, 79.32%. Wrapper `height:723px`.
```html
<path class="arc-path" d="M242.9,19.37 A272.47,272.47 0 0,0 20,287.29 A272.47,272.47 0 0,0 242.9,555.22"/>
```
Items are absolutely positioned and **anchored to the job title line** (`translateY(-13.5px)`), per an explicit user reversal — an earlier version aligned dots to the company name instead. Each item is `<div class="c-title-row"><span class="c-dot"></span><div class="c-role">…</div></div>` then `.c-company`, `.c-date`, `.c-line`. Mobile dots are plain `border-radius:50%` divs, **not** SVG (SVG with `preserveAspectRatio="none"` squashed them into ovals).

---

## Gamer theme (the only mode)

`body.gamer-mode` is set in the markup and drives everything. There is no toggle any more; the block below is kept only because the geometry note explains a class of bug worth remembering.
```css
.mode-option{position:relative;z-index:1;min-width:84px;padding:7px 12px;text-align:center;}
.mode-slider{position:absolute;top:4px;left:4px;height:calc(100% - 8px);width:calc(50% - 4px);
             background:var(--grad);transition:transform .34s cubic-bezier(0.16,1,0.3,1);}
body.gamer-mode .mode-slider{transform:translateX(100%);}
```
**`min-width:84px`, not `flex:1`** — flex items have `min-width:auto`, so "PM" and "GAMER" have different content floors and the slider misaligns.

Gamer theme — **"Arcade Cabinet"**, fourth full reskin. History: cyan→blue→violet (original) → pink/violet/cyan "RGB battle-station" (rejected: *"looks more like Canva... no gradient"*) → flat toxic-green "Combat Terminal" (rejected: *"that green color... doesn't look really good"*) → amber/magenta/cyan "Signal Glitch" (rejected, bluntly: *"you are just changing colors in the theme... I asked you to come up with a new theme itself... don't just change colors and try something like that. Try something extremely new."* Plus a feel complaint — *"make things smoother... it feels stucky"* — and a correction on the photo: *"add back the borders and everything. I just don't want any foreground layer on top of it. The photo should have a background and everything which was there previously, but it will adapt according to the mode."*) → **current**.

The first three passes were all the same skeleton — dark bg, one or few neon/glow accents, thin translucent HUD borders, corner brackets, a `.reveal`-flicker animation — with different hex values. This pass changes the *vocabulary*, not just the palette:
```css
body.gamer-mode{--bg:#0b0b10;--bg-2:#141419;--bg-3:#1c1c22;
  --border:#000;--border-hi:#000;
  --text:#fff9e8;--text-dim:#9a93a6;--a1:#ff2f4f;--a2:#ffcf3f;--a3:#2f9bff;
  --grad:#ff2f4f;}
```
Red/yellow/blue (arcade-marquee primaries) instead of any neon hue tried before. `--grad` is still a flat hex — "no gradient except the cursor" was never retracted, so this pass's novelty is structural, not a loophole around that rule.

**What actually changed (not just which hex is in `--a1`):**
- **Hard, unblurred, offset drop-shadows** (`box-shadow:7px 7px 0 #000`, no blur radius) replace glow (`box-shadow:0 0 Npx rgba(...)`) everywhere — cards, buttons, the photo, the target ball. This is a comic-panel/trading-card depth cue, not a neon one, and it's the single biggest reason this pass doesn't look like the previous three with new colors.
- **Corner-bracket HUD accents are gone.** They survived unchanged through all three earlier redesigns; retired outright, no replacement — the hard shadow + thick `3px solid #000` border carries the "gamer panel" read now.
- **Tactile press interaction**: `.btn-primary`/`.btn-ghost` shift toward their shadow on hover (`translate(-2px,-2px)`, shadow grows) and slam flat on `:active` (`translate(3px,3px)`, shadow to 0) — a physical button-press language that didn't exist in any earlier pass (those were all hover-glow only).
- **Alternating card tilt** (`--tilt:-1.6deg` / `1.6deg` via `nth-child(odd/even)` on `.bento`, `.skills-grid`, `.edu-grid`) breaks PM's precisely aligned grid — cards lie like scattered trading cards, straightening to `rotate(0)` on hover. This is the literal "reposition/restructure, not just recolor" ask.
- **The hero photo**: **restored** the padding+`var(--grad)` border/background frame (removed last pass, wrongly — the user clarified they wanted it back, just adaptive to the theme instead of a fixed color, which it already was via the CSS variable). On top of that it's now **rotated (`-3deg`) and nudged up (`translateY(-26px)`)** out of PM's centered stack, with the same hard trading-card shadow as the cards — genuinely repositioned, not just recolored, and still nothing rendered on top of the image pixels (the frame is still the border-around-it padding trick, never an overlay).
- **Reveal animation simplified, not layered.** Earlier passes added a second `animation` (`powerOn`, then `glitchIn`) on top of `.reveal`'s own `transition` — safe once the animated property didn't collide, but still two motion systems stacked on one element. This pass uses **one mechanism**: just a bouncier `transition-timing-function` (`cubic-bezier(0.34,1.56,0.64,1)`) on the same opacity/transform/filter transition PM already has. Smooth by construction, no `steps()`, nothing to feel "stucky" — directly answers the smoothness complaint.
- **`steps()` timing removed everywhere.** The "Signal Glitch" pass used `steps(4–6,end)` (deliberately jumpy, "digital") on the reveal, the ambient bars, and the mode-switch burst — that stepped/discrete motion is almost certainly what read as stuttery. Every animation in this pass uses `ease`/`ease-out`/`cubic-bezier` — continuous interpolation, nothing snapping between fixed frames.
- **Ambient motion, redesigned smooth**: `spawnSparkle()` (index.html, near `setMode`) drifts 1-3 small flat-color pixels upward and fades (`ease-out`, ~2.1s) every ~0.3–0.7s while idle in gamer mode — replaces the old stepped glitch-bar flicker. **User liked this one and asked for it more twice** — original interval 1.4–3.2s (single spawn) → 0.65–1.4s (35% chance of a pair) → **current** 0.3–0.7s with a weighted 1/2/3 spawn count (15% triple, 40% double, 45% single). Measured via a 5s mutation-observer count each round: 5 → 14 sparkles, confirming each bump actually landed, not just changed numbers that didn't move the visible rate. `modePowerFlash()` replaces the old 5-bar glitch burst with one calm full-viewport opacity pulse (`ease-out`, ~400ms) on every PM↔Gamer toggle.
- **CTA text is white, not dark.** `.btn-primary`/`.mini-btn.primary` briefly had dark text (`#1a0006`) for contrast against the bright red fill; the user pointed out the Recent Work tag/CTA convention is white text on the accent color (`.p-card .tag{color:#fff}`, PM's own base rule), so gamer mode's CTAs now match that (`color:#fff`) instead of the one-off dark override.
- **The hero photo is straight, not tilted.** `body.gamer-mode .hero-photo-wrap{transform:translateY(-26px);}` — the `rotate(-3deg)` this pass first shipped with was explicitly rejected ("the photo should not be slanted or tilted"); the upward reposition stays (that part landed fine, confirmed on mobile too), only the rotation was removed. **The alternating tilt on Recent Work/Skills/Background cards is untouched** — that's the "remaining tilting part I liked" the user was explicit about keeping.
- **Cursor trail recolored** — `TRAIL_STOPS` (index.html, search `TRAIL_STOPS`) sweeps red→yellow→light-blue→blue. Only the color stops changed; the trail's hard-won mechanics (Catmull-Rom, one-fill ribbon, mobile momentum handling, `IDLE_MS`/`DRAIN_PER_FRAME`) are untouched, per its standing "don't regress" constraints below.
- **Stack's tower blocks recolored** to the same red/yellow/blue discrete cycle (`BLOCK_COLORS`, 3 colors now instead of 4) — still supersedes the older "don't touch Stack" instruction from a couple of rounds back, since the user has separately asked for the blocks to change. Only fill color changed; physics/scoring/perfect-detection/crumble untouched, and the "perfect stack" burst is still cyan-accented (`#8df3ff`/`rgba(0,229,255,…)`, untouched).
- Moderate rounded corners (`14px` cards, `8px` buttons/pills) replace the previous pass's razor-sharp `0px` — a third, distinct shape language (PM is very round at `22px`, "Combat Terminal"/"Signal Glitch" were `0px`, this is in between) chosen to read as "chunky game-console panel," not either extreme.

**Cursors are gamer-only** — PM mode stays `auto` (explicit user requirement). Crosshair 31×31 hotspot 15 15; sniper scope 57×57 hotspot 28 28 scoped to `#shootBoard`. Recolored red/blue rings + yellow center dot.

There is no mode toggle any more (gamer is the only mode), so nothing persists or needs to.

---

## Cursor trail (the most iterated feature)

Canvas ribbon that follows the pointer in gamer mode. Constants at index.html:~1343-1373 (values below updated to current, except `TRAIL_STOPS`, which is the old pink palette — the live stops are violet→mint, `rgba(124,92,255)` … `rgba(43,232,200)`):
```js
const coarsePointer = window.matchMedia('(pointer: coarse)').matches;
const TRAIL_LEN = 40, HEAD_W = 15;
const SUB = coarsePointer ? 12 : 20;         // spline samples per source point
const SPARK_CAP = coarsePointer ? 70 : 140;
const HEAD_EASE = coarsePointer ? 0.22 : 0.75; // desktop 0.75 since 2026-10-07 (was 0.22 everywhere → tip ~46px behind cursor)
const IDLE_MS = coarsePointer ? 190 : 70;
const DRAIN_PER_FRAME = coarsePointer ? 1 : 2;
const TRAIL_STOPS = [[0,'rgba(255,47,138,0)'],[0.10,'rgba(255,47,138,1)'],[0.32,'rgba(168,85,247,1)'],
                     [0.54,'rgba(99,102,241,1)'],[0.76,'rgba(59,157,255,1)'],[1.00,'rgba(120,240,255,1)']];
```
**Draw pipeline (order matters):** ease head toward pointer (`HEAD_EASE`) → push a point only if `performance.now() - lastMoveTs <= IDLE_MS` → 4 neighbor-averaging smoothing passes (head pinned) → Catmull-Rom resample at `SUB` → build left/right edges by perpendicular offset → **one `fill()`** with a linear gradient → solid tip circle at `HEAD_W*0.42` → depth-sorted sparks.

**Hard-won constraints — don't regress any of these:**
- **One filled ribbon, not per-segment strokes.** Per-segment round-capped strokes stack alpha and read as "multiple cursors" / translucent beads. Width taper replaces alpha fade; gradient stops are fully opaque.
- **Catmull-Rom passes *through* its inputs**, so raw corners survive as visible straight lines. The pre-smoothing passes (now 4) + `SUB` (now 20) are what fixed it (max turn 98.8° → 28.3°, avg 6.41° → 1.68°).
- **Mobile uses passive touch events, not pointer events.** Browsers fire `pointercancel` when claiming a gesture for scroll, which the handler read as a finger-lift, so the trail died the instant you moved. A/B proof: old 0px vs new 3862px mid-drag.
- **The trail follows momentum scrolling on mobile** (`onTrailScroll`, index.html:~1425) — a flick is a very short gesture, so without this it only ever flashed. It bails once the position would leave the viewport (±8px), otherwise the ribbon pins to the screen edge and collapses.
- **Push and drain can cancel out.** At the slower mobile drain rate, pushing a point every frame left a permanent stub. Hence the `IDLE_MS` gate on pushing.
- Desktop path is deliberately untouched: *"Web works absolutely fine. You don't have to touch the web on anything."*

---

## Games (`#arcade`)
- **Ship It** (`#shootBoard`) — targets speed up as you shoot: `sizeFor(s)=Math.max(24, 46-s*1.4)`, `lifeFor(s)=Math.max(420, 1400-s*70)`. Sniper-scope cursor over the board. Went through **three** hit-effect designs this project: loud flash+shockwave → water-droplet dispersal → **current: an FPS hit-marker** (researched real shooter/RPG UI conventions before building this one — see Sources below). The droplet version had a real bug the user caught ("when I am clicking on it, I don't get any effects"): `.hit-ring`/`.hit-droplet` were `position:absolute` children of `#shootBoard` (`z-index:auto`), while the site-wide click-burst (`.click-ring`/`.click-spark`, fired on *every* pointerdown including on the target) is `position:fixed;z-index:9997`. Since `#shootBoard` never establishes its own stacking context, the fixed click-ring rendered on top of and visually masked the specific hit effect at the same coordinates — a pure CSS/stacking bug, invisible to a JS-error check. **Fix, and the new design's whole point:** `popAt()` now renders `.hit-marker` (white X, COD-style), `.hit-ring`, and `.hit-number` ("+1") as `position:fixed` elements appended straight to `<body>` at the click's *viewport* coordinates (`shootBoard.getBoundingClientRect()` + `cx/cy`) with `z-index:9999` — same layer as the click burst, deliberately higher, so it can never be masked again. Damage number drifts up and fades over ~0.7s (instant spawn, drift-up-and-fade is the universal convention). Plus a `.board-shake` transform-based shake on `#shootBoard` itself (kept *off* the fx elements' ancestry specifically so the shake's `transform` — which creates a new stacking context — can't trap them again).
- **Stack** (`#stackCanvas`) — time-based motion: `dt = Math.min(50, ts-lastTs)/16.667` (per-frame movement made it speed-dependent across 60/120Hz). **A miss used to fall as one solid slab; now `addFalling()` crumbles it into 3-5 irregular chunks (each its own `falling[]` entry with independent `vx/vy/rot/vr`) plus 7 tiny white dust chips** (`dust:true` flag branches the renderer to draw a small square instead of the tower's gradient bar) kicked up at the fracture line. All entries still ride the one `falling[]` array/physics loop and draw **last**, above the game-over veil.
  - **Perfect stack**: landing within `PERFECT_PX = 2.5` keeps the block's full width, slices nothing off, and increments `combo`. `addPop()` pushes to `pops[]` (`{x,w,level,t,combo,parts[]}`), aged in frame units to `POP_LIFE = 48`. Drawn on canvas: white blow-out of the block, an expanding glowing outline, beams out of both edges, 12 shrapnel dots, and a rising `PERFECT`/`PERFECT xN` label — deliberately left cyan-accented (untouched by the RGB battle-station recolor) so "perfect" reads as a distinct bonus color against the pink/violet main palette. Anchored by `level` so the camera pan carries it, exactly like `falling[]`. `combo` resets on any non-perfect drop and on game over.
  - Tower blocks and falling chunks now render in the new pink→violet→cyan gradient (`#ff2fb0 → #7c3aff → #00e5ff`), was cyan→blue→violet.
- High scores in `localStorage`: `shipItBest`, `stackBest`.

---

## Verification workflow (run before every push)
Headless Chrome can't simulate real scrolling here. Instead:
1. **Structural check** — Python tag-balance counts (`<section>`, `<div>`, `<span>`, `<svg>`) to catch unclosed tags.
2. **JS error + element count** — inject an error listener that writes results into `document.title`, then:
   ```
   chrome --headless=new --window-size=1200,900 --virtual-time-budget=4000 --dump-dom <file>
   ```
   and grep the title. Confirms zero errors and no element-count regressions.
3. **Visual check** — `--screenshot` with a temporary debug `<style>` forcing `.visible` states so elements render settled.
4. **Game physics** — stub `requestAnimationFrame` with `setTimeout` + synthetic timestamps to drive the loop deterministically (rAF barely fires under `--virtual-time-budget`).
5. **A/B against the previous version** — `git show HEAD:index.html` into the scratchpad and measure both.
6. Clean up scratchpad/temp files when done.

**Known headless limitations:**
- Viewport height is floored at ~500px in `--dump-dom` mode (a "broken" 390px nav was just a cropped screenshot). `--screenshot` honors small sizes.
- CSS transitions do not advance under `--virtual-time-budget` — verify transform end-states by temporarily disabling the transition.
- **The fixed-position trail canvas is never composited into screenshots.** The trail has only ever been verified *numerically* (painted px, opaque ratio, color buckets, turn angles), never visually. Disclose this whenever reporting on it.
- A test harness that injects `transform:none !important` will strip legitimate layout transforms (e.g. `.c-item` `translateY`) and fabricate misalignment. Exclude those.

## Deployment (GitHub Pages, legacy build)
```bash
~/.local/bin/gh auth setup-git   # refresh credential helper before every push
git add -A
git commit -m "..."              # + Co-Authored-By trailer
git push
~/.local/bin/gh api -X POST repos/karthikburugula46-source/karthikeya-portfolio/pages/builds
~/.local/bin/gh api repos/karthikburugula46-source/karthikeya-portfolio/pages/builds/latest --jq .status
```
- `gh` is at `~/.local/bin/gh` (no Homebrew on this machine).
- Pages `build_type` set to `legacy` via `gh api -X PUT repos/.../pages`.
- The session environment sometimes reports this directory as **not** a git repo. It is one — run `git status` before believing otherwise.

---

## Most recent completed request (2026-10-04, third pass) — commit `3fbaafe`
Four complaints in one message: *"fix the padding issue between my profile and the recent work. Such a huge padding issue is not allowed. Remove that lined foreground layer on my image. My image should be clear. You didn't use the image I gave you. How are you making such mistakes, man? And the animation is not yet fixed again. It's completely gone. How? Fix it."*

1. **Animation — the real fix, after a wrong one.** See **Observer attach timing** under Animation architecture. Method that found it: diff `.reveal`/`.orbit-card` CSS against the last user-confirmed-good commit (`5a77701`) — byte-identical, so nothing was deleted; rule out `prefers-reduced-motion`; then isolate the one remaining variable (when `observe()` runs) and A/B it on the same file. **Lesson: the previous round's fold-based theory was never tested against an alternative — it was the first plausible story, and it cost the user an extra round.**
2. **Padding.** Because hero height no longer governs whether the entrance plays, `min-height` was removed outright. `.hero{padding:56px 0 24px}`, new `#highlights{padding-top:40px}`, mobile `.hero{padding:44px 0 18px}`. Measured heroH **1000 → 414**, gap CTA→"Recent Work" heading **204 → 142**.
3. **"Lined foreground layer" / "you didn't use my image."** The file on disk *is* the supplied image — verified by opening `assets/profile.png` directly. What read as a baked-in filter was `body.gamer-mode::after`: a full-viewport `repeating-linear-gradient` CRT scanline overlay at `z-index:9998`, sitting on top of the portrait. **Removed entirely.** The user had objected to a foreground layer on the photo once before — that earlier round only removed the frame, not this overlay, which is why it came back.

Verified: tags and CSS braces balanced (289/289); `ERR=[] reveal=11 orbit=4 slots=4 arc=1 cdot=4 citem=4 sparkles=true trail=block`; desktop and true-390px screenshots reviewed; pushed `4279171..3fbaafe`, Pages build polled to `built`, and a `curl` of the live site grepping `requestAnimationFrame(()=>requestAnimationFrame` returned 1 — fix confirmed live.

**Caveat disclosed to the user:** `window.scrollTo` does not scroll in this headless environment (every step reports `@y0`), so the fix was verified by frame-timing A/B, not by watching a real scroll.

## Second most recent completed request (2026-10-04, second pass)
A batch of requests that arrived mid-turn, so they are logged together.

1. **New portrait.** The old `assets/profile.png` had a filter baked into the *image file* — there was never a CSS filter on it, so this was an asset swap, not a style change. New shot resized to 800px (830KB, down from 1.28MB). Frame untouched, as asked.
2. **Name on one line.** Hard `<br>` removed; `.hero h1` → `clamp(26px,4.3vw,56px)`. It still wraps naturally on very narrow phones, and fits one line at 390px.
3. **Bigger photo:** `clamp(230px,26vw,340px)`, up from `clamp(186px,20vw,250px)`; mobile steps 250/210/180.
4. **"You lost the animations" — ~~diagnosed and fixed~~ MISDIAGNOSED.** Nothing was removed (`.reveal`/`.orbit-card` counts and the observers diffed clean against the session-start commit), but the cause was *not* the fold. I blamed the shorter hero pulling `#highlights` above the fold and added `.hero{min-height:min(calc(100vh - 64px), 1000px)}`. That fixed nothing, and the 1000px cap meant Recent Work still landed above the fold on a large monitor. It also created the padding complaint that followed. **Superseded by the third pass above — the real cause was observer attach timing.** The `min-height` is now gone entirely.
5. **Nav pill vs Contact sized differently** → both now share an explicit `height:34px` (30px ≤400px) with `box-sizing:border-box` and `display:inline-flex`. The pill's border plus smaller type made it visibly shorter before.
6. **Mobile crowding** → `.brand-name` now hides below **560px** (was 430px).
7. **Theme off red → "Violet & Mint"** (user picked from three offered options): `--a1:#7c5cff`, `--a2:#ffc23d`, `--a3:#2be8c8`, `--grad:#7c5cff`. Red was baked into **far more than the tokens** — card/button/photo glows, the HUD grid lines, the career arc gradient + 4 node colors + 4 `--dot-color` values, both cursor SVGs (URL-encoded `%23ff2f4f`), `SPARKLE_COLORS`, `SPARK_COLORS`, `TRAIL_STOPS`, the trail's `shadowColor`, `BLOCK_COLORS` and the Stack glow. `grep` for the old hexes now returns 0.
8. **Trail smoothness**: smoothing passes 2 → 4, `SUB` 14 → 20 (coarse 9 → 12). On a replayed synthetic flick the sharpest turn halved (4.35° → 2.05°), average turn −33%, 43% more samples along the curve.
9. **Copy**: intro now "PM by day, gamer and builder by night… 4 years into product" (wording confirmed by the user). Arcade subtitle dropped its stale "You switched to gamer mode" line.

**Still inconsistent, flagged to the user:** the intro says "4 years" (their explicit instruction) while the Career subtitle says "4.5 years".

**Headless gotcha hit again:** running several `--headless` Chrome invocations in a tight shell loop makes them reuse one window, so every width reports identical numbers. Give each its own `--user-data-dir` (slow) or just run them as separate commands.

## Earlier on 2026-10-04
Hero rebuilt around user-supplied intro copy; availability pill moved into the nav.

**Layout.** The hero was a centered stack (pill → name → one-line sub → CTAs → full-width 460px portrait underneath). It is now two columns: photo left, name + title + intro + CTAs right, inside `.hero-inner` (`display:flex; align-items:center`). The photo is sized by its column — `width:clamp(186px,20vw,250px)` — so it cannot dominate the fold again ("the image itself is taking a lot of space, which I don't want"). New `.hero-role` element carries **"Product Manager at AssetPlus."** `.hero h1` dropped from `clamp(48px,9vw,108px)` to `clamp(40px,6.1vw,82px)` to share the row.

**Gutters.** `.hero` lost its horizontal padding; `.hero-inner` took `max-width:1180px;padding:0 32px` (20px under 640) so it is the same box as `.wrap`. Without this the photo's left edge sat 32px outside the Recent Work cards below it.

**Responsive.** Columns stack at **820px**, not 640 — the copy column gets too narrow for the headline well before the phone break. Stacked = photo on top, everything centered. Photo 178px → 150px (≤640) → 132px (≤380). Verified at 1440/1280/1024/900/860/820/640/500 and at a true 390: no horizontal scroll anywhere.

**Nav pill.** "Open to Product roles" moved out of the hero into a new `.nav-right` beside Contact. Label shortens to "Open to work" at ≤860px; at ≤430px the brand drops to the avatar alone (`.brand-name` hidden) so the pill and Contact both keep full labels on a phone.

**Two bugs caught by measuring, not by looking:**
1. **CSS specificity.** `.eyebrow-pill` is defined *after* the nav block, so at equal specificity it beat every `.nav-pill` rule — the pill rendered at full hero size and its `margin-bottom:28px` pushed it exactly 14px above the nav's centre line (measured top `-1`, height `38`). Fixed by scoping to `nav .nav-pill`. **Nav CSS sits before hero CSS in this file; anything overriding `.eyebrow-pill` from the nav must be scoped under `nav`.**
2. **`assets/profile.png` was missing from the working tree** (`git status` showed ` D`) while still present in HEAD and on the live site. A `git add -A` would have deleted the user's photo from the deployed site. Restored with `git checkout --` before committing; added a `.gitignore` for `.DS_Store`. **Check `git status` for unexpected deletions before every `git add -A` in this repo.**

**Headless technique worth reusing:** `--dump-dom` floors the viewport at ~500px wide, so phone widths cannot be measured directly. Load the page in a **fixed-size iframe** inside a wrapper document and read `contentDocument` — needs `--allow-file-access-from-files` or `contentDocument` is null. That gives a genuine 390px viewport.

Verified: zero JS errors at every width; element counts unchanged (11 reveal, 4 orbit-card, 4 p-slot, 1 arc-path, 4 c-dot); trail canvas still `display:block`; tags and CSS braces balanced; desktop and 390px screenshots reviewed.

**Flagged to the user, unresolved:** their copy says "Four years into product" while the Career subtitle reads 4.5 years. Their wording was kept verbatim.

## 2026-09-29 completed request
Three things in one pass:
1. **PM mode and the toggle removed** — gamer is now the only mode. `<body class="gamer-mode">` is hardcoded in the markup, the `.mode-toggle`/`.mode-slider`/`.mode-option` markup + CSS + media-query overrides are gone, and `setMode`/`modeButtons`/`modePowerFlash` + the `.mode-flash` CSS went with them. `gamerOn` survives as `const gamerOn = true` because the click bursts, trail and sparkles all read it. **The base (non-`.gamer-mode`) CSS above the theme block must stay** — it is the foundation the theme overrides, not dead PM code. The boot calls (`fitStack/startStack/startTrail/startSparkle`) used to fire on the toggle click; they now run once at the **very end** of the script, because `fitStack`/`startStack` are `let`-assigned inside the Stack block, not hoisted declarations.
2. **BI Developer at Infometry: Feb 2022 – Aug 2022 → Feb 2022 – Mar 2023**, which closes the gap that used to sit between it and the Product Internships (Mar 2023).
3. **Career subtitle 4 → 4.5 years.** The old "4 years" equalled total months actually worked excluding that gap (6 + 43 = 49mo ≈ 4.08y). With the gap closed it is a continuous Feb 2022 → present span = 55mo ≈ 4.58y, hence 4.5. Same arithmetic if this needs redoing later.
4. **Ambient sparkle frequency up ~30%** (interval 300–700ms → 250–590ms, weights 15/40/45 → 20/42/38 for 3/2/1 per tick). Measured 14 → 16 in a matched 4s window.

Verified: zero JS errors; gamer mode active on load with `toggleEls=0`; element counts unchanged (11 reveal, 4 orbit-card, 4 p-slot, 1 arc-path, 4 c-dot, 4 c-item); tags and CSS braces balanced; A/B against the pre-change file with `.gamer-mode` force-applied showed identical computed cursor/text-shadow on `.p-card`/`.pill`/`nav a` and the hero row only 1px shorter (the toggle was the taller element in that flex row). Boot proof: rAF calls after load with zero interaction went 0 (old, PM default) → 1252 (new), with 9 sparkles already in the DOM.

## Sparkle frequency, round 2
Sparkle frequency turned up again — "reduce the timer more and increase the frequency and quantity more," a follow-up to the previous round's first bump, with everything else confirmed good. Interval cut from ~0.65–1.4s to ~0.3–0.7s, and the spawn count per tick went from a flat 1-or-2 to a weighted 1/2/3 (15% triple, 40% double, 45% single). Measured 14 sparkles in 5s post-change vs. 5 in 5s before — landed, not just relabeled. No other changes.

## Third most recent completed request
Three small, targeted refinements to "Arcade Cabinet" (no new redesign that round — first time in a while the feedback was "keep this, adjust that" instead of "start over"):
1. **Ambient sparkle frequency increased** (first bump — see above for the follow-up) — the user liked `spawnSparkle()` specifically and asked for it more often; interval roughly halved (1.4–3.2s → ~0.65–1.4s) plus an occasional double-spawn.
2. **CTA text reverted to white** — was briefly dark (`#1a0006`) for contrast on the bright red fill; user pointed out the Recent Work tag already uses white on the accent color, so `.btn-primary`/`.mini-btn.primary` now match that convention.
3. **Hero photo un-tilted** — the `-3deg` rotation from the Arcade Cabinet redesign was explicitly rejected ("should not be slanted or tilted"); removed, keeping the `translateY(-26px)` upward reposition, which the user confirmed they liked *and* checked on mobile before asking for this. The alternating tilt on Recent Work/Skills/Background cards was explicitly asked to stay ("the remaining tilting part I liked") — untouched.

Verified: zero JS errors; `getComputedStyle` confirmed the photo wrap's transform is now a pure `matrix(1,0,0,1,0,-26)` (translation only, no rotation) and the CTA's computed color is `rgb(255,255,255)`; a 5-second mutation-observer count on a fresh gamer-mode session showed 5 sparkles (~1/s), consistent with the new interval and roughly 2–2.5× the old rate.

## Previous completed request
*"You are just changing colors in the theme... I asked you to come up with a new theme itself... don't just change colors and try something like that. Try something extremely new."* Plus: motion felt "stucky" (stuttery), and the photo's frame should come back (adaptive to the theme, just not an overlay on the image itself). Fourth full gamer-mode reskin — "Arcade Cabinet," full design in the Gamer theme section above. This was the first pass to touch **shape/structure**, not just CSS custom-property values:

1. **Replaced the visual vocabulary**: hard unblurred offset drop-shadows instead of glow, thick solid borders instead of thin translucent ones, a physical button-press interaction instead of hover-glow, corner-bracket HUD accents retired outright (survived 3 straight redesigns unchanged — that persistence was itself the "just changing colors" tell).
2. **Actually restructured, not just recolored**: alternating card tilt (`--tilt`, scattered-trading-card feel, straightens on hover) across Recent Work/Skills/Background; the hero photo rotated and shifted up out of PM's centered stack. This is the literal "reposition my images... restructure, that's fine" ask.
3. **Fixed the smoothness complaint at its source**: every `steps()` timing function (used for "digital glitch" jitter in the previous pass) is gone, replaced with `ease`/`cubic-bezier` throughout. The reveal animation went from two motion systems layered on one element (a `transition` plus a separate `animation`) down to one — just a bouncier easing curve on the existing transition. Ambient motion and the mode-switch transition were redesigned as smooth single-property fades (`spawnSparkle()`, `modePowerFlash()`) instead of the old stepped multi-bar bursts.
4. **Photo frame restored, adaptively**: reverted last pass's removal of `.hero-photo-card`'s padding+`var(--grad)` border/background — it was already theme-adaptive (that's what `var(--grad)` means), it just needed to exist again. Never became an overlay on the image itself either version.
5. **Trail and Stack blocks recolored** to the new red/yellow/blue palette, mechanics/physics untouched in both, consistent with prior passes.

Verified: zero JS errors across repeated PM↔gamer↔PM toggling with Ship It/Stack interactions in between; tags/braces balanced; confirmed via `getComputedStyle` that the alternating card tilt actually renders (differing `matrix3d` per card, not just written and unused). Screenshots: hero showing the photo's frame restored *and* visibly rotated/repositioned, comic-shadow cards with visible tilt. **Same standing caveat:** the *feel* of the smoothed motion — whether it actually reads as fluid now — can't be judged from a static screenshot or a JS check; only a live look settles that, and it's the one thing most worth confirming given this round's specific complaint.

## Fourth most recent completed request
*The "RGB battle-station" theme was rejected wholesale — "looks more like Canva now... I don't want any gradient kind of feel... no more gradient contrast on the theme side... the background is not moving anymore... don't add anything like what you added" — plus a real bug report: "you again fucked up the animation on Ship It... when I am clicking on it, I don't get any effects."* Did actual web research (see Sources) before rebuilding, rather than iterating blind a third time. Two things landed in the same pass:

1. **Full gamer-mode re-theme #2 — "Combat Terminal.".** Single flat toxic-green accent on black, `--grad` changed from a `linear-gradient()` to a flat hex so every consumer becomes a solid fill for free, all RGB hue-cycling deleted, the distracting scan-beam deleted (reverted `::after` to plain static scanlines — that scanline `::after` was itself removed in the third pass, see above), ambient grid restored to a single-hue orthogonal drift (faster/more visible), sharp `border-radius:0` everywhere for a real shape-level "opposite of PM" signal, cards flattened to a solid color (no top-strip gradient). Full rationale and diff of what changed vs. the rejected version is in the Gamer theme section above.
2. **Ship It hit effect — found and fixed the real bug, rebuilt as a proper FPS hit-marker.** Root cause of "no effects": a stacking-context/z-index bug (site-wide click-burst at `z-index:9997` masked the hit-specific effect, which had no explicit z-index) — a pure CSS bug a JS-error check can't catch. Rebuilt per real hit-marker/damage-number conventions researched online: white X hit-marker + "+1" damage number (drifts up, fades ~0.7s) + a flat ring + a short `translate()`-based board shake, all rendered `position:fixed` straight on `<body>` at `z-index:9999` so they can never be masked again.
3. **Stack was explicitly left untouched at the time** — "Stack It is perfect. Don't change anything in that" — a since-superseded instruction (see Most recent completed request above).

### Sources (read before the Combat Terminal pass, informed that design)
- [Esports/HUD typography: Bebas Neue, angular condensed type](https://fontalternatives.com/blog/gaming-fonts-hud-esports-branding/)
- [2026 UI trends: grids as foreground elements, monospace-as-identity](https://tubikstudio.com/blog/ui-design-trends-2026/)
- [FPS damage indicator UX analysis](https://medium.com/@jasper.stephenson/a-ux-analysis-of-first-person-shooter-damage-indicators-59ac9d41caf8)
- [Damage numbers as "juice": instant spawn, drift-up-and-fade 0.6–1.2s convention](https://www.gamejuice.co.uk/articles/damage-numbers-satisfying-feedback)
- [Game feel on the web: screen shake via `transform`/`translate3d`, hitstop](https://valdemird.com/blog/game-feel-on-the-web/)
- [Gaming portfolio/dark-UI inspiration](https://www.sitebuilderreport.com/inspiration/game-developer-portfolios)

## Earlier completed request
*"It should work on the normal scrolling also… irrespective of the time pool and everything."* → Mobile trail now survives flick + momentum scroll. Three mobile-only changes (momentum following, longer idle grace, half drain rate) plus the push/drain cancellation fix. Verified: `flick=3027, mom3=1845, mom8=1066, idle600=0, idle1500=0`; desktop unchanged at `mousePainted=14046, opaqueRatio=0.52, afterIdle=0`. Zero JS errors, tags balanced. Commit `07af968`, pushed, Pages build `built`.

## Pending / open
1. **Recent Work card content** — the user is drafting a PRD with the real copy for the 4 cards: *"I'll draft those sections for my PRD, and I'll give you later what to do and what should not be there."* Current copy is placeholder. **This is the main thing to wait for.**
2. Feedback on the **cursor feel after `HEAD_EASE` 0.75** (just shipped, verified numerically only), and on the new landing fold / LinkedIn CTA on a real device.
3. Known inconsistency: intro says "4 years", Career subtitle says "4.5 years" — user's wording kept verbatim; ask before changing.
4. Offered but never requested (do not build unprompted): mode persistence via `localStorage`, deeper gamification (XP bars, career-as-level-path, achievements).
