# VRP Construction — "From Vision to Reality"

A full-screen, scroll-driven construction-journey section. A single hand-built
SVG house scene stays pinned on the left while ten timeline steps scroll past
on the right; each step scrubs the scene forward as the person scrolls,
exactly like an Apple/Tesla product page.

Open `index.html` directly in a browser — no build step, no server, no
dependencies beyond the two local GSAP files already included in `libs/`.

---

## Folder structure

```
vision-section/
│
├── index.html          → markup: header, pinned stage, SVG scene, timeline list
├── style.css            → dark/gold theme, glassmorphism timeline, responsive rules
├── script.js             → GSAP timeline, procedural SVG generation, scroll logic
│
├── assets/               → standalone reference SVGs (see "Why is the scene
│     house.svg              inline?" below) — swap these in your own mockups
│     blueprint.svg           or use them as a starting point for a redesign
│     birds.svg
│     tree.svg
│     clouds.svg
│     sun.svg
│
├── libs/
│     gsap.min.js         → GSAP core (vendored, no CDN required)
│     ScrollTrigger.min.js → GSAP's scroll-linked animation plugin
│
└── README.md
```

## Why is the scene inline in `index.html` instead of loaded from `assets/`?

GSAP animates individual elements *inside* the SVG (a single column, a single
brick, a single window pane). That level of control requires the SVG to live
in the DOM, not be referenced through an `<img>` tag. Loading it via
`fetch()` was considered, but `fetch()` of a local file is blocked by the
browser when `index.html` is opened directly from disk (no server) — which
would break the "must work immediately after opening index.html" requirement.

So the working scene is inlined directly in `index.html`, and the `assets/`
folder contains clean, standalone copies of the core shapes (house, blueprint,
birds, tree, clouds, sun) for you to reuse elsewhere, hand off to a designer,
or use as a visual reference when customizing the inline version.

## How the animation works

1. **One master GSAP timeline.** `script.js` builds a single `gsap.timeline()`
   divided into ten equal "units," one per construction step (see
   `populateTimeline()`). Every tween — grass growing, bricks stacking, the
   roof sliding down, the paint sweeping across the walls — is placed at an
   absolute position on that timeline (e.g. step 5's bricks start at `4`,
   step 6's roof starts at `5`).

2. **ScrollTrigger scrubs it.** That timeline is handed a `scrollTrigger`
   config with `scrub: 1`, so its playhead position is tied directly to
   scroll position rather than playing on a clock. On desktop the whole
   `.vision-stage` (SVG + timeline list) is pinned for the duration of a
   1000vh tall wrapper (`#pinWrap`, 100vh × 10 steps), so the visual stays
   fixed on screen while the story plays out.

3. **Mobile gets the same timeline, no pin.** Under 981px, `ScrollTrigger.matchMedia()`
   swaps in a second config: no `pin`, scrubbed instead against the natural
   height of the timeline list, while the SVG panel uses ordinary CSS
   `position: sticky` to stay in view. Same animations, no jank on small
   screens.

4. **The right-hand list reacts to the same playhead.** `setStep()` reads the
   master timeline's `progress` on every scroll tick, works out which of the
   10 steps that corresponds to, and toggles `.is-complete` / `.is-active`
   classes plus fills the vertical progress line — so the SVG and the list
   are always perfectly in sync because they're driven by the same number.

5. **Ambient loops run independently.** Clouds drifting, grass swaying, the
   sun's glow breathing — these are separate infinite GSAP tweens
   (`startAmbientLoops()`) that run from page load, layered underneath the
   scroll-scrubbed story animation without conflicting with it (they animate
   `rotation`/`x` on parent groups, never the same properties the step
   timeline controls). Birds, chimney smoke and sparkles only start looping
   once step 10 is reached (`startFinaleLoops()`), so the "completed home"
   feels alive without cluttering earlier steps.

6. **Bricks, grass and the blueprint grid are generated in JS**, not
   hand-typed in the SVG — see the top of `script.js`. This keeps the markup
   short and makes density/spacing a one-line change instead of a manual
   redraw.

## Respecting reduced motion & accessibility

- `prefers-reduced-motion: reduce` gets its own `ScrollTrigger.matchMedia`
  branch: the scene jumps straight to the finished house, and the timeline
  cards fade in individually via `IntersectionObserver`-style triggers
  instead of scrubbing.
- The timeline is a semantic `<ol>`; each `<li>` is `tabindex="0"` and
  responds to `Enter`/`Space` by scrolling to that step (mirrors the click
  behavior), with a visible focus ring (`:focus-visible`).
- A visually-hidden `aria-live="polite"` region announces the current step
  name as it changes, for screen-reader users following the scroll story.
- All decorative SVG groups are `aria-hidden="true"`; the actual content
  (title, subtitle, step names, descriptions) is ordinary readable text.

## Customizing

**Colors** — everything is driven by CSS custom properties at the top of
`style.css` (`--bg`, `--gold`, `--white`, etc.) plus a matching set of SVG
`<linearGradient>`/`<radialGradient>` definitions inside `index.html`'s
`<defs>`. Change both together to keep the scene and UI in sync.

**Copy** — step titles/descriptions live directly in the `<li class="timeline-item">`
blocks in `index.html`. The caption under the SVG panel and the `aria-live`
announcer text pull step names from the `STEP_NAMES` array at the top of
`script.js` — keep that array in the same order as the list in the HTML.

**Timing** — every step's animations are grouped and commented inside
`populateTimeline()` in `script.js` (`/* STEP 1 — VISION */`, etc.). Move a
tween's start time earlier/later by changing its position argument (the
number after the `duration` object), or change how long it takes via
`duration`.

**Pin length / scroll speed** — desktop scroll distance is controlled by
`#pinWrap`'s height in `style.css` (`height: 1000vh` = 100vh per step).
Increase it for a slower, more deliberate scroll; decrease it for a snappier
one.

**Replacing the SVG art** — the scene lives entirely inside
`<svg id="sceneSvg">` in `index.html`, organized into one `<g>` group per
building step (`#blueprintGroup`, `#foundationGroup`, `#columnsGroup`,
`#wallsGroup`, `#roofGroup`, `#windowsGroup`, `#treesGroup`, etc.). To swap
an element's artwork, edit the shapes inside its group — the `id`s are what
`script.js` targets, so keep those stable, or update the corresponding
selector in `script.js` if you rename something.

## Performance notes

- All animated properties are transform/opacity based (GPU-accelerated) —
  the only exceptions are the blueprint/steel-rod stroke-dash draws and the
  paint-sweep clip-path width, both of which are cheap, one-shot, and only
  run while their step is active.
- `will-change` is intentionally *not* applied broadly; GSAP manages
  transform compositing per element only while it's tweening.
- Grass blades and bricks are generated once on load, not per scroll frame.
- `ScrollTrigger.matchMedia()` fully tears down and rebuilds the pin/scrub
  setup on breakpoint changes, so resizing the window (e.g. rotating a
  tablet) doesn't leave orphaned pins.
