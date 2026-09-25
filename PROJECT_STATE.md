# Karthikeya Burugula — Portfolio Site: Project State

Resume file for continuing work in a new chat. Attach this file and say **"continue from here."**

Last updated: 2026-09-25 · HEAD `8f1a45f`

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
- **Ambient motion, redesigned smooth**: `spawnSparkle()` (index.html, near `setMode`) drifts one (sometimes two — 35% chance) small flat-color pixel upward and fades (`ease-out`, ~2.1s) every ~0.65–1.4s while idle in gamer mode — replaces the old stepped glitch-bar flicker. **User liked this one specifically and asked for it more often** ("increase the output... more frequency"); the interval was originally 1.4–3.2s, roughly halved after that feedback. `modePowerFlash()` replaces the old 5-bar glitch burst with one calm full-viewport opacity pulse (`ease-out`, ~400ms) on every PM↔Gamer toggle.
- **CTA text is white, not dark.** `.btn-primary`/`.mini-btn.primary` briefly had dark text (`#1a0006`) for contrast against the bright red fill; the user pointed out the Recent Work tag/CTA convention is white text on the accent color (`.p-card .tag{color:#fff}`, PM's own base rule), so gamer mode's CTAs now match that (`color:#fff`) instead of the one-off dark override.
- **The hero photo is straight, not tilted.** `body.gamer-mode .hero-photo-wrap{transform:translateY(-26px);}` — the `rotate(-3deg)` this pass first shipped with was explicitly rejected ("the photo should not be slanted or tilted"); the upward reposition stays (that part landed fine, confirmed on mobile too), only the rotation was removed. **The alternating tilt on Recent Work/Skills/Background cards is untouched** — that's the "remaining tilting part I liked" the user was explicit about keeping.
- **Cursor trail recolored** — `TRAIL_STOPS` (index.html, search `TRAIL_STOPS`) sweeps red→yellow→light-blue→blue. Only the color stops changed; the trail's hard-won mechanics (Catmull-Rom, one-fill ribbon, mobile momentum handling, `IDLE_MS`/`DRAIN_PER_FRAME`) are untouched, per its standing "don't regress" constraints below.
- **Stack's tower blocks recolored** to the same red/yellow/blue discrete cycle (`BLOCK_COLORS`, 3 colors now instead of 4) — still supersedes the older "don't touch Stack" instruction from a couple of rounds back, since the user has separately asked for the blocks to change. Only fill color changed; physics/scoring/perfect-detection/crumble untouched, and the "perfect stack" burst is still cyan-accented (`#8df3ff`/`rgba(0,229,255,…)`, untouched).
- Moderate rounded corners (`14px` cards, `8px` buttons/pills) replace the previous pass's razor-sharp `0px` — a third, distinct shape language (PM is very round at `22px`, "Combat Terminal"/"Signal Glitch" were `0px`, this is in between) chosen to read as "chunky game-console panel," not either extreme.

**Cursors are gamer-only** — PM mode stays `auto` (explicit user requirement). Crosshair 31×31 hotspot 15 15; sniper scope 57×57 hotspot 28 28 scoped to `#shootBoard`. Recolored red/blue rings + yellow center dot.

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
Three small, targeted refinements to "Arcade Cabinet" (no new redesign this round — first time in a while the feedback was "keep this, adjust that" instead of "start over"):
1. **Ambient sparkle frequency increased** — the user liked `spawnSparkle()` specifically and asked for it more often; interval roughly halved (1.4–3.2s → ~0.65–1.4s) plus an occasional double-spawn. See the Gamer theme section above for exact numbers.
2. **CTA text reverted to white** — was briefly dark (`#1a0006`) for contrast on the bright red fill; user pointed out the Recent Work tag already uses white on the accent color, so `.btn-primary`/`.mini-btn.primary` now match that convention.
3. **Hero photo un-tilted** — the `-3deg` rotation from the Arcade Cabinet redesign was explicitly rejected ("should not be slanted or tilted"); removed, keeping the `translateY(-26px)` upward reposition, which the user confirmed they liked *and* checked on mobile before asking for this. The alternating tilt on Recent Work/Skills/Background cards was explicitly asked to stay ("the remaining tilting part I liked") — untouched.

Verified: zero JS errors; `getComputedStyle` confirmed the photo wrap's transform is now a pure `matrix(1,0,0,1,0,-26)` (translation only, no rotation) and the CTA's computed color is `rgb(255,255,255)`; a 5-second mutation-observer count on a fresh gamer-mode session showed 5 sparkles (~1/s), consistent with the new interval and roughly 2–2.5× the old rate.

## Second most recent completed request
*"You are just changing colors in the theme... I asked you to come up with a new theme itself... don't just change colors and try something like that. Try something extremely new."* Plus: motion felt "stucky" (stuttery), and the photo's frame should come back (adaptive to the theme, just not an overlay on the image itself). Fourth full gamer-mode reskin — "Arcade Cabinet," full design in the Gamer theme section above. This was the first pass to touch **shape/structure**, not just CSS custom-property values:

1. **Replaced the visual vocabulary**: hard unblurred offset drop-shadows instead of glow, thick solid borders instead of thin translucent ones, a physical button-press interaction instead of hover-glow, corner-bracket HUD accents retired outright (survived 3 straight redesigns unchanged — that persistence was itself the "just changing colors" tell).
2. **Actually restructured, not just recolored**: alternating card tilt (`--tilt`, scattered-trading-card feel, straightens on hover) across Recent Work/Skills/Background; the hero photo rotated and shifted up out of PM's centered stack. This is the literal "reposition my images... restructure, that's fine" ask.
3. **Fixed the smoothness complaint at its source**: every `steps()` timing function (used for "digital glitch" jitter in the previous pass) is gone, replaced with `ease`/`cubic-bezier` throughout. The reveal animation went from two motion systems layered on one element (a `transition` plus a separate `animation`) down to one — just a bouncier easing curve on the existing transition. Ambient motion and the mode-switch transition were redesigned as smooth single-property fades (`spawnSparkle()`, `modePowerFlash()`) instead of the old stepped multi-bar bursts.
4. **Photo frame restored, adaptively**: reverted last pass's removal of `.hero-photo-card`'s padding+`var(--grad)` border/background — it was already theme-adaptive (that's what `var(--grad)` means), it just needed to exist again. Never became an overlay on the image itself either version.
5. **Trail and Stack blocks recolored** to the new red/yellow/blue palette, mechanics/physics untouched in both, consistent with prior passes.

Verified: zero JS errors across repeated PM↔gamer↔PM toggling with Ship It/Stack interactions in between; tags/braces balanced; confirmed via `getComputedStyle` that the alternating card tilt actually renders (differing `matrix3d` per card, not just written and unused). Screenshots: hero showing the photo's frame restored *and* visibly rotated/repositioned, comic-shadow cards with visible tilt. **Same standing caveat:** the *feel* of the smoothed motion — whether it actually reads as fluid now — can't be judged from a static screenshot or a JS check; only a live look settles that, and it's the one thing most worth confirming given this round's specific complaint.

## Previous completed request
*The "RGB battle-station" theme was rejected wholesale — "looks more like Canva now... I don't want any gradient kind of feel... no more gradient contrast on the theme side... the background is not moving anymore... don't add anything like what you added" — plus a real bug report: "you again fucked up the animation on Ship It... when I am clicking on it, I don't get any effects."* Did actual web research (see Sources) before rebuilding, rather than iterating blind a third time. Two things landed in the same pass:

1. **Full gamer-mode re-theme #2 — "Combat Terminal.".** Single flat toxic-green accent on black, `--grad` changed from a `linear-gradient()` to a flat hex so every consumer becomes a solid fill for free, all RGB hue-cycling deleted, the distracting scan-beam deleted (reverted `::after` to plain static scanlines), ambient grid restored to a single-hue orthogonal drift (faster/more visible), sharp `border-radius:0` everywhere for a real shape-level "opposite of PM" signal, cards flattened to a solid color (no top-strip gradient). Full rationale and diff of what changed vs. the rejected version is in the Gamer theme section above.
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
2. Possible feedback on the trail's *visual feel* — it has only been verified numerically.
3. Offered but never requested (do not build unprompted): mode persistence via `localStorage`, deeper gamification (XP bars, career-as-level-path, achievements).
