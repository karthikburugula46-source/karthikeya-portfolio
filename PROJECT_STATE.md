# Karthikeya Burugula — Portfolio Site: Project State

Resume file for continuing work in a new chat. Attach this file and say **"continue from here."**

Last updated: 2026-09-22 · HEAD `552145f`

## What this is
Single-file static portfolio website for Karthikeya Burugula (Product Manager). No build tooling, no frameworks — HTML, inline `<style>`, inline `<script>`, all in one file (~1,730 lines). Dark/gradient brand aesthetic inspired by ultrahuman.com/in. Scroll-triggered reveal animations throughout. Now has a **PM mode / Gamer mode** toggle that re-themes the whole site.

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
   Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
   ```
7. **Vercel is abandoned** — "Skip Vercel, GitHub Pages is enough. Do not revisit unless asked."

## Critical incident (why rule #3 exists)
User reported: "You screwed up the entire portfolio page... You removed the skill section... top highlights..." Root cause: GSAP/ScrollTrigger (CDN-loaded, plus complex 3D transforms and custom `toggleActions`) silently failed on the user's device, leaving `.reveal` elements stuck at `opacity:0` — looked exactly like deleted content, but the HTML was intact. Fix: removed GSAP, replaced with vanilla `IntersectionObserver` toggling a `.visible` class driving plain CSS transitions. Permanent architecture.

---

## Page structure (current, in DOM order)
Hero → `#highlights` (Recent Work) → `#arcade` (games) → `#skills` → `#career` → `#background`

Section-by-section:
- **Nav "Contact" CTA and hero "Get in Touch" both `href="tel:+918296114865"`** — they dial directly instead of scrolling to the footer (works in PM and gamer mode, mobile and desktop). The footer `#contact` section still exists with the mail + phone row.
- **Hero** — `.hero-top-row` holds the "Open to Product roles" eyebrow pill **and** the PM/Gamer mode toggle side by side (toggle was moved here from the nav specifically so it's reachable on mobile).
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

**Global reveal:** `.reveal` starts hidden; `revealObserver` (index.html:1016) toggles `.visible` bidirectionally on `entry.isIntersecting`, so scrolling back up reverses it.

**Orbit reveal — Recent Work only** (replicates the revolving-card entrance from a reference recording of ultrahuman.com). This must NOT be applied to any other section. **Gamer mode must never apply the `powerOn` stepped-opacity flicker (below) to `.orbit-card`** — an animation always wins over a transition on the same property while it plays, so when the .42s `powerOn` animation ended before the orbit card's own .55s opacity transition did, opacity snapped to whatever the transition had reached mid-swing — a visible break the user caught. `.orbit-card.visible` gets no animation now, only its own smooth transition; `.reveal.visible` still gets `powerOn`.
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

**Career arc timeline** — SVG path + `.arc-node` circles, animated via `getTotalLength()` + `stroke-dashoffset`, driven by a dedicated `arcObserver` (index.html:1066). Geometry was recomputed in Python for 4 unequal-height items: circle center (292.47, 287.292), R = 272.47. Item tops: 2.77%, 24.14%, 45.52%, 79.32%. Wrapper `height:723px`.
```html
<path class="arc-path" d="M242.9,19.37 A272.47,272.47 0 0,0 20,287.29 A272.47,272.47 0 0,0 242.9,555.22"/>
```
Items are absolutely positioned and **anchored to the job title line** (`translateY(-13.5px)`), per an explicit user reversal — an earlier version aligned dots to the company name instead. Each item is `<div class="c-title-row"><span class="c-dot"></span><div class="c-role">…</div></div>` then `.c-company`, `.c-date`, `.c-line`. Mobile dots are plain `border-radius:50%` divs, **not** SVG (SVG with `preserveAspectRatio="none"` squashed them into ovals).

---

## PM / Gamer mode

Toggle markup lives in `.hero-top-row`; `body.gamer-mode` drives everything.
```css
.mode-option{position:relative;z-index:1;min-width:84px;padding:7px 12px;text-align:center;}
.mode-slider{position:absolute;top:4px;left:4px;height:calc(100% - 8px);width:calc(50% - 4px);
             background:var(--grad);transition:transform .34s cubic-bezier(0.16,1,0.3,1);}
body.gamer-mode .mode-slider{transform:translateX(100%);}
```
**`min-width:84px`, not `flex:1`** — flex items have `min-width:auto`, so "PM" and "GAMER" have different content floors and the slider misaligns.

Gamer theme — **"Combat Terminal"** (rebuilt from scratch a second time — the RGB battle-station attempt above was rejected outright: *"the entire theme is really bad... this looks more like Canva now... I don't want any gradient kind of feel on this... The only gradient that is there is the cursor"*). Current design is a single flat accent, no blended color anywhere in the UI chrome:
```css
body.gamer-mode{--bg:#05070b;--bg-2:#0a0d14;--bg-3:#10141d;
  --border:rgba(198,255,46,.18);--border-hi:rgba(198,255,46,.5);
  --text:#e7ffe2;--text-dim:#798c78;--a1:#c6ff2e;--a2:#c6ff2e;--a3:#c6ff2e;
  --grad:#c6ff2e;}
```
**`--grad` is a flat hex, not a `linear-gradient()`, on purpose.** Every consumer of `var(--grad)` (`.grad-text` hero name, `.mode-slider`, `.btn-primary`, `.mini-btn.primary`, `.game-target`, the `.hero-photo-card` padding-trick border) turns into a solid fill automatically just by changing this one variable — no other markup/JS touched. `--a1/--a2/--a3` are all the same green (no trio, no "gradient contrast" between accents) since a multi-hue scheme was itself part of what got rejected. Bright-green fills need dark text for contrast: `body.gamer-mode .btn-primary{color:#04140a}` and same for `.mini-btn.primary`.

Plus:
- **Ambient bg**: back to an orthogonal grid (single green hue, `gridDrift 9s`, faster/more opaque than either earlier attempt so the drift actually reads). `::after` is CRT scanlines only, **static, no animation** — the earlier version's sweeping scan beam was the specific thing the user called "distracting... in the foreground," so it's gone outright, not just recolored.
- **No RGB hue-cycle anywhere.** `rgbCycle`/`hue-rotate` was deleted entirely (was on `.mode-slider` and `.btn-primary:hover`) — cycling through hues is exactly the "gradient contrast going up and down" the user rejected, even though it's a `filter`, not a literal CSS gradient.
- **Cards**: flat `#070c07` background, flat border, no top strip, `border-radius:0`. Corner-bracket accents kept (bumped 13px→15px since nothing else decorates the card now), auto-inherit `var(--a1)`.
- **Sharp corners everywhere** (`border-radius:0` on cards, pills, buttons, mode-toggle, hero-photo-card) — a shape-language signal, not just color, for "polar opposite of PM's rounded cards."
- Neon flicker on the `> ` heading prefix kept (a flat-color opacity blink, not a moving/blended element — wasn't part of either complaint).
- Cursors (crosshair + sniper scope) recolored to flat green + a white center dot, dropped the old two/three-tone rings.
- `SPARK_COLORS` (site-wide click confetti) cut down to `['198,255,46','255,255,255']` — two flat hues, still not a gradient since each spark is one solid color.

**Cursors are gamer-only** — PM mode stays `auto` (explicit user requirement). Crosshair 31×31 hotspot 15 15; sniper scope 57×57 hotspot 28 28 scoped to `#shootBoard`.

Mode does **not** persist across reloads — always loads PM. Offered but never requested.

---

## Cursor trail (the most iterated feature)

Canvas ribbon that follows the pointer in gamer mode. Constants at index.html:1147-1173:
```js
const coarsePointer = window.matchMedia('(pointer: coarse)').matches;
const TRAIL_LEN = 40, HEAD_W = 15;
const SUB = coarsePointer ? 9 : 14;          // spline samples per source point
const SPARK_CAP = coarsePointer ? 70 : 140;
const IDLE_MS = coarsePointer ? 190 : 70;
const DRAIN_PER_FRAME = coarsePointer ? 1 : 2;
const TRAIL_STOPS = [[0,'rgba(255,47,138,0)'],[0.10,'rgba(255,47,138,1)'],[0.32,'rgba(168,85,247,1)'],
                     [0.54,'rgba(99,102,241,1)'],[0.76,'rgba(59,157,255,1)'],[1.00,'rgba(120,240,255,1)']];
```
**Draw pipeline (order matters):** ease head toward pointer (0.22) → push a point only if `performance.now() - lastMoveTs <= IDLE_MS` → 2 neighbor-averaging smoothing passes (head pinned) → Catmull-Rom resample at `SUB` → build left/right edges by perpendicular offset → **one `fill()`** with a linear gradient → solid tip circle at `HEAD_W*0.42` → depth-sorted sparks.

**Hard-won constraints — don't regress any of these:**
- **One filled ribbon, not per-segment strokes.** Per-segment round-capped strokes stack alpha and read as "multiple cursors" / translucent beads. Width taper replaces alpha fade; gradient stops are fully opaque.
- **Catmull-Rom passes *through* its inputs**, so raw corners survive as visible straight lines. The 2 pre-smoothing passes + `SUB` 14 are what fixed it (max turn 98.8° → 28.3°, avg 6.41° → 1.68°).
- **Mobile uses passive touch events, not pointer events.** Browsers fire `pointercancel` when claiming a gesture for scroll, which the handler read as a finger-lift, so the trail died the instant you moved. A/B proof: old 0px vs new 3862px mid-drag.
- **The trail follows momentum scrolling on mobile** (`onTrailScroll`, index.html:1225) — a flick is a very short gesture, so without this it only ever flashed. It bails once the position would leave the viewport (±8px), otherwise the ribbon pins to the screen edge and collapses.
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

## Most recent completed request
*The "RGB battle-station" theme was rejected wholesale — "looks more like Canva now... I don't want any gradient kind of feel... no more gradient contrast on the theme side... the background is not moving anymore... don't add anything like what you added" — plus a real bug report: "you again fucked up the animation on Ship It... when I am clicking on it, I don't get any effects."* Did actual web research (see Sources) before rebuilding, rather than iterating blind a third time. Two things landed in the same pass:

1. **Full gamer-mode re-theme #2 — "Combat Terminal.".** Single flat toxic-green accent on black, `--grad` changed from a `linear-gradient()` to a flat hex so every consumer becomes a solid fill for free, all RGB hue-cycling deleted, the distracting scan-beam deleted (reverted `::after` to plain static scanlines), ambient grid restored to a single-hue orthogonal drift (faster/more visible), sharp `border-radius:0` everywhere for a real shape-level "opposite of PM" signal, cards flattened to a solid color (no top-strip gradient). Full rationale and diff of what changed vs. the rejected version is in the Gamer theme section above.
2. **Ship It hit effect — found and fixed the real bug, rebuilt as a proper FPS hit-marker.** Root cause of "no effects": a stacking-context/z-index bug (site-wide click-burst at `z-index:9997` masked the hit-specific effect, which had no explicit z-index) — a pure CSS bug a JS-error check can't catch. Rebuilt per real hit-marker/damage-number conventions researched online: white X hit-marker + "+1" damage number (drifts up, fades ~0.7s) + a flat ring + a short `translate()`-based board shake, all rendered `position:fixed` straight on `<body>` at `z-index:9999` so they can never be masked again.
3. **Stack was explicitly left untouched** — "Stack It is perfect. Don't change anything in that" — including its own gradient fills, which is a deliberate, disclosed exception to the "no gradient" rule everywhere else.

Verified: zero JS errors toggling PM↔gamer↔PM repeatedly; tags balanced; a harness confirmed `.hit-marker`'s computed `z-index` (9999) beats `.click-ring`'s (9997) — `markerBeatsClick=true` — which is the actual mechanical fix, not just "we changed some CSS and hoped." Stack re-screenshotted pixel-identical to before (same pink/violet/cyan gradient tower, same crumble) to prove zero collateral changes. Old classes (`hit-flash`, `hit-plus`, `board-hit`, `hit-droplet`, `hit-ring-2`, `rgbCycle`, `scanSweep`) confirmed fully removed, not just superseded. Screenshots confirm the new black/green look and the button-contrast fix (dark text on bright green). **Caveat carried over from every past attempt:** motion (grid drift, neon flicker, hit-marker/shake timing) isn't visible under headless `--virtual-time-budget` and hasn't been eyeballed live by either of us — please look at it live before judging.

### Sources (read before this pass, informed the design)
- [Esports/HUD typography: Bebas Neue, angular condensed type](https://fontalternatives.com/blog/gaming-fonts-hud-esports-branding/)
- [2026 UI trends: grids as foreground elements, monospace-as-identity](https://tubikstudio.com/blog/ui-design-trends-2026/)
- [FPS damage indicator UX analysis](https://medium.com/@jasper.stephenson/a-ux-analysis-of-first-person-shooter-damage-indicators-59ac9d41caf8)
- [Damage numbers as "juice": instant spawn, drift-up-and-fade 0.6–1.2s convention](https://www.gamejuice.co.uk/articles/damage-numbers-satisfying-feedback)
- [Game feel on the web: screen shake via `transform`/`translate3d`, hitstop](https://valdemird.com/blog/game-feel-on-the-web/)
- [Gaming portfolio/dark-UI inspiration](https://www.sitebuilderreport.com/inspiration/game-developer-portfolios)

## Previous completed request
*"It should work on the normal scrolling also… irrespective of the time pool and everything."* → Mobile trail now survives flick + momentum scroll. Three mobile-only changes (momentum following, longer idle grace, half drain rate) plus the push/drain cancellation fix. Verified: `flick=3027, mom3=1845, mom8=1066, idle600=0, idle1500=0`; desktop unchanged at `mousePainted=14046, opaqueRatio=0.52, afterIdle=0`. Zero JS errors, tags balanced. Commit `07af968`, pushed, Pages build `built`.

## Pending / open
1. **Recent Work card content** — the user is drafting a PRD with the real copy for the 4 cards: *"I'll draft those sections for my PRD, and I'll give you later what to do and what should not be there."* Current copy is placeholder. **This is the main thing to wait for.**
2. Possible feedback on the trail's *visual feel* — it has only been verified numerically.
3. Offered but never requested (do not build unprompted): mode persistence via `localStorage`, deeper gamification (XP bars, career-as-level-path, achievements).
