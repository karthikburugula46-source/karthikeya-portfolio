# Karthikeya Burugula — Portfolio Site: Project State

Resume file for continuing work in a new chat. Attach this file and say **"continue from here."**

Last updated: 2026-09-21 · HEAD `11b1626`

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

**Orbit reveal — Recent Work only** (replicates the revolving-card entrance from a reference recording of ultrahuman.com). This must NOT be applied to any other section.
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

Gamer theme (dark, minimal, cyan→blue→violet):
```css
body.gamer-mode{--bg:#05070b;--bg-2:#0a0d14;--bg-3:#10141d;
  --border:rgba(0,229,255,.12);--border-hi:rgba(0,229,255,.34);
  --text:#e6f7ff;--text-dim:#78909c;--a1:#00e5ff;--a2:#3b9dff;--a3:#8b5cf6;
  --grad:linear-gradient(120deg,#00e5ff 0%,#3b9dff 52%,#8b5cf6 100%);}
```
Plus: animated HUD grid + vignette (`body.gamer-mode::before`, `gridDrift 14s`), CRT scanlines (`::after`, z-index 9998), `> ` prefix on section headings, 8px card radius with expanding corner brackets on hover, `powerOn .42s steps(1,end)` on reveal, click bursts.

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
- **Ship It** (`#shootBoard`) — targets speed up as you shoot: `sizeFor(s)=Math.max(24, 46-s*1.4)`, `lifeFor(s)=Math.max(420, 1400-s*70)`. Sniper-scope cursor over the board. `popAt()` fires a **loud** hit: `.hit-flash` (white-hot radial core) + `.hit-ring` + `.hit-ring-2` (second, faster shockwave, `.08s` delay) + 14 `.hit-spark` (8px, 46–94px travel, own `hitSparkFly` keyframe so the shared `sparkFly` click burst is untouched) + a floating `.hit-plus` "+1", plus a one-shot `board-hit` inset pulse on the board. The `board-hit` class is removed → reflow → re-added so rapid hits re-trigger it.
- **Stack** (`#stackCanvas`) — time-based motion: `dt = Math.min(50, ts-lastTs)/16.667` (per-frame movement made it speed-dependent across 60/120Hz). Missed overhang becomes `falling[]` debris `{x,w,level,dy,vy,vx,rot,vr}`, drawn **last** so it renders above the game-over veil.
  - **Perfect stack**: landing within `PERFECT_PX = 2.5` keeps the block's full width, slices nothing off, and increments `combo`. `addPop()` pushes to `pops[]` (`{x,w,level,t,combo,parts[]}`), aged in frame units to `POP_LIFE = 48`. Drawn on canvas: white blow-out of the block, an expanding glowing outline, beams out of both edges, 12 shrapnel dots, and a rising `PERFECT`/`PERFECT xN` label. Anchored by `level` so the camera pan carries it, exactly like `falling[]`. `combo` resets on any non-perfect drop and on game over.
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
*Contact CTAs → dialer; louder Ship It hit; perfect-stack animation.* Verified headless: zero JS errors; tags balanced; Ship It hit spawns `flash=1, rings=2 (incl. ring2), sparks=14, plus=1, board-hit=1`; a forced pixel-perfect Stack drop paints 159 near-white px at frame 1 → 32 at frame 11 → 0 at frame 56, against a **control non-perfect drop that paints 0 at every frame**, so the burst is provably the perfect path. Visual screenshot of the Stack burst confirmed.

## Previous completed request
*"It should work on the normal scrolling also… irrespective of the time pool and everything."* → Mobile trail now survives flick + momentum scroll. Three mobile-only changes (momentum following, longer idle grace, half drain rate) plus the push/drain cancellation fix. Verified: `flick=3027, mom3=1845, mom8=1066, idle600=0, idle1500=0`; desktop unchanged at `mousePainted=14046, opaqueRatio=0.52, afterIdle=0`. Zero JS errors, tags balanced. Commit `07af968`, pushed, Pages build `built`.

## Pending / open
1. **Recent Work card content** — the user is drafting a PRD with the real copy for the 4 cards: *"I'll draft those sections for my PRD, and I'll give you later what to do and what should not be there."* Current copy is placeholder. **This is the main thing to wait for.**
2. Possible feedback on the trail's *visual feel* — it has only been verified numerically.
3. Offered but never requested (do not build unprompted): mode persistence via `localStorage`, deeper gamification (XP bars, career-as-level-path, achievements).
