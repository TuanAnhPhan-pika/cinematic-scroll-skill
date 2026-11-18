---
name: cinematic-scroll-web
description: >-
  Designs and builds continuous scroll-coupled interactive web experiences—including
  cinematic storytelling pages, interactive manga/editorial visuals, and scroll-driven
  product showcases or high-converting e-commerce landing pages—using a sticky SVG/CSS
  viewport, spring-lerp scroll physics, reactive SVG filters, and choreographed 6-stage rails.
  Use when creating scroll-triggered storytelling sites, luxury product reveals, exploded
  macro-zoom showcases, or continuous single-page sales experiences.
---

# Cinematic Scroll-Coupled Web Skill

Build zero-dependency, 60fps continuous-scroll interactive web experiences where every visual layer—camera zoom, SVG morphing, particle scatter, wind physics, and narrative/sales copy—shares a single normalized scroll coordinate (`position` in `[0, 1]`).

## Architecture Overview

Every page follows a four-layer sticky stage inside a tall scroll track:

```text
#track (height: 700vh, creates natural scroll distance)
└── #viewport (position: sticky; top: 0; height: 100vh; overflow: hidden)
    ├── #sky / ambient background layers (color & mood interpolation)
    ├── #world (isolated stage: hero centerpiece, SVG filters, particles, atmosphere)
    ├── #rail (position: absolute; N stacked 100vh chapters translated continuously)
    └── #texture / HUD (#bar, #meter, #guide, sticky CTA bar for e-commerce)
```

- **Never toggle discrete slides with classes alone.** Drive `#rail` via `translate3d(0, ${-position * (N - 1) * innerHeight}px, 0)` and interpolate all scene transitions with smoothstep (`smooth(a, b, position)`).
- **Keep `#world` and `#rail` non-blocking (`pointer-events: none`)** except on explicit interactive elements (`#overlay button`, variant swatches, Buy/Cart CTAs) which must set `pointer-events: auto` and higher `z-index`.

## Build Workflow

1. **Choose the Domain Mode**:
   - **Storytelling / Manga**: Follow [Physics & SVG Engine](./references/physics-and-svg-engine.md) and start from [starter-template.html](./examples/starter-template.html).
   - **Product Showcase / E-Commerce**: Follow [E-Commerce Adaptation Guide](./references/ecommerce-adaptation-guide.md) and study [ecommerce-product-showcase.html](./examples/ecommerce-product-showcase.html).
2. **Design the Layered Hero SVG**:
   - Build the hero centerpiece (character, emblem, or product hardware) inside a single normalized `viewBox` with named `<g>` layers so individual sub-parts can rotate, explode apart, zoom, or swap colorway independently.
3. **Wire the Spring-Lerp Loop (`requestAnimationFrame`)**:
   - Compute `target = clamp(-track.getBoundingClientRect().top / maxScroll, 0, 1)`.
   - Ease `position += (target - position) * 0.115` and derive smoothed `speed` and bounded `impulse` (`wind`) so fast scrolling naturally drives cloth, hair, particles, and SVG displacement filters.
4. **Choreograph the 6-Stage Arc**:
   - Map `0.00 -> 1.00` across 6 evenly spaced `100vh` `.chapter` sections (`top: 0` through `top: 500vh`).
   - Pair progressive zoom (`scale`) and layer crossfades (`smooth(a, b, position)`) with concise `.caption` cards and a top `#bar` + `#meter` HUD.
5. **Verify Before Shipping**:
   - Serve over local HTTP and test `00%`, `25%`, `50%`, `75%`, and `100%` scroll positions.
   - Confirm interactive overlays (`#enter`), variant selectors, and sticky CTAs remain clickable (`pointer-events: auto`).
   - Confirm zero console errors, readable text contrast across bright focal elements, and clean framing at both desktop (`1280x800`) and mobile (`390x844`) viewports.

## Critical SVG & CSS Pitfalls

Avoid these common failure modes when building SVG-heavy scroll pages:

1. **CSS `#id { opacity: 0 }` blocks `setAttribute('opacity', ...)`**:
   ID selectors in CSS override SVG presentation attributes. Either omit `opacity: 0` in CSS (use SVG attribute `opacity="0"` initially + `setAttribute('opacity', ...)`) or update `el.style.opacity` in JS.
2. **SVG `transform` attributes ignore CSS `transform-origin`**:
   `transform-box: fill-box; transform-origin: ...` applies only to CSS `el.style.transform`. For SVG `setAttribute('transform', ...)`, explicitly wrap rotations/scales around center `(cx, cy)`:
   `translate(cx cy) rotate(deg) scale(s) translate(-cx -cy)`.
3. **Use Cubic Bézier Curves (`C`) Instead of Spiky Wedges**:
   For organic hair, ribbons, cloth, smoke, or liquid trails, avoid triangular `L` spikes and large CSS `skewX` distortions. Animate cubic Bézier control points smoothly with sine-wave offsets along `phase` and clamped `wind`.
4. **Lock Hero Wrapper Aspect Ratio to `viewBox`**:
   Give the hero SVG container `aspect-ratio: <W>/<H>; height: auto;` (instead of mixing independent `vw` and `vh` dimensions) so `transform-origin` zoom coordinates stay locked on target across all screen sizes.
5. **Prevent Mid-Screen Atmospheric Seams**:
   Do not close mid-screen fog/mist `<path>` shapes along a flat horizontal line (`L1440 490 ... Z`) with a visible stroke. Use `stroke: none` and close cloud bands smoothly below the viewport or with undulating bottom curves.
6. **Guard Text Contrast Over Luminous Focal Areas**:
   Apply layered dark drop-shadows (`text-shadow`) on large display titles and dark translucent cards (`backdrop-filter: blur(4px)`) on captions so copy stays legible even when passing over bright moons, glows, or metallic product highlights.

## References & Examples

- [Physics & SVG Engine](./references/physics-and-svg-engine.md) — Spring-lerp math, `smooth()`, reactive SVG filters, concentric rotations, traveling Bézier waves, and particle scatter formulas.
- [E-Commerce Adaptation Guide](./references/ecommerce-adaptation-guide.md) — Adapting the 6-chapter arc to product launches, exploded views, colorway switching, spec callouts, and sticky buy bars.
- [Starter Template](./examples/starter-template.html) — Clean 6-stage continuous-scroll scaffold ready for any theme.
- [E-Commerce Product Showcase](./examples/ecommerce-product-showcase.html) — Full working luxury timepiece / hardware product launch demo built with this engine.
