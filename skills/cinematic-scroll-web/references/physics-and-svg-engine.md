# Physics & SVG Engine Reference

Detailed mathematical primitives and SVG animation patterns for continuous scroll-coupled pages.

## 1. Spring-Lerp Scroll & Velocity Impulse Loop

Decouple raw `scrollTop` jumps from visual rendering using exponential smoothing so mouse-wheel ticks, trackpads, and touch drags all feel fluid:

```js
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const smooth = (a, b, x) => {
  const t = clamp((x - a) / (b - a), 0, 1);
  return t * t * (3 - 2 * t);
};

let target = 0, position = 0, speed = 0, wind = 0, phase = 0;

function draw() {
  const max = Math.max(1, track.offsetHeight - innerHeight);
  target = clamp(-track.getBoundingClientRect().top / max, 0, 1);

  const prev = position;
  position += (target - position) * 0.115; // position lag / smoothness
  const delta = position - prev;

  speed = speed * 0.82 + delta * 0.18;
  const impulse = clamp(speed * 4800, -34, 34);
  wind += (impulse - wind) * 0.11;
  phase += 0.016 + Math.min(Math.abs(speed) * 10, 0.03);

  rail.style.transform = `translate3d(0, ${-position * 5 * innerHeight}px, 0)`;
  requestAnimationFrame(draw);
}
```

### Tuning Constants
- **Track height**: `700vh` gives `600vh` of scroll travel (`max = 6 * innerHeight`), while `6` chapters placed at `top: 0, 100vh, ..., 500vh` translate by `-position * 5 * innerHeight` so `position = 0` centers Chapter 1 and `position = 1` centers Chapter 6.
- **Damping (`0.115`)**: Higher (`0.18`) feels snappy for technical product UI; lower (`0.08`) feels weighty and cinematic.
- **`prefers-reduced-motion`**: When `(prefers-reduced-motion: reduce)` matches, snap `position = target` directly and zero out `wind` / oscillation amplitudes.

## 2. Choreography with `smooth(a, b, position)`

Use overlapping smoothstep windows `[a, b]` to orchestrate multi-stage reveals without abrupt jumps:

| Stage Window | Purpose | Example Transform |
| :--- | :--- | :--- |
| `0.00 – 0.18` | Establishing shot | Base `scale(1)`, cold ambient lighting |
| `0.16 – 0.36` | Primary transformation | Crossfade core orb / product reveal (`opacity: smooth(.16, .34, p)`) |
| `0.34 – 0.60` | Dynamic action / burst | Particle scatter, exploded parts (`translate`), kinetic lines |
| `0.55 – 0.82` | Macro / focal zoom | Center zoom (`scale(1 + 0.75 * smooth(.52, .82, p))`) |
| `0.84 – 0.96` | Transition curtain | Vignette / curtain sweep (`opacity: smooth(.84, .94, p)`) |
| `0.93 – 0.99` | Final statement / CTA | Final tagline or conversion card (`opacity: smooth(.93, .985, p)`) |

## 3. Reactive SVG Displacement Filters

Drive SVG `<feTurbulence>` and `<feDisplacementMap>` attributes gently from `wind` and `phase` to make clouds, mist, heat shimmer, or energy auras react directly to scroll speed:

```xml
<filter id="fogwarp" x="-40%" y="-60%" width="180%" height="220%">
  <feTurbulence id="fogNoise" type="fractalNoise" baseFrequency="0.008 0.02" numOctaves="2" seed="7" result="noise"/>
  <feDisplacementMap id="fogDisplace" in="SourceGraphic" in2="noise" scale="26" xChannelSelector="R" yChannelSelector="G"/>
</filter>
```

```js
$('#fogNoise').setAttribute(
  'baseFrequency',
  `${(0.007 + Math.abs(wind) * 0.00022).toFixed(4)} ${(0.019 + Math.abs(wind) * 0.00014).toFixed(4)}`
);
$('#fogDisplace').setAttribute(
  'scale',
  String((24 + Math.abs(wind) * 1.6 + Math.sin(phase) * 6).toFixed(1))
);
```

Keep `numOctaves="2"` (never `4+` in full-viewport layers) to stay smooth at 60fps on laptops and mobile devices.

## 4. Concentric SVG Rotation & Morphing

For rotating rings, Sharingan tomoe, clockwork gears, or camera lens blades inside a `400x400` `viewBox`:

```js
const rot = position * 720 + spinAngle * 0.4;
const scale = 1 - 0.35 * morphOut;
ringGroup.setAttribute('opacity', (1 - morphOut).toFixed(3));
ringGroup.setAttribute(
  'transform',
  `translate(200 200) rotate(${rot.toFixed(2)}) scale(${scale.toFixed(3)}) translate(-200 -200)`
);
```

Placing radial child elements via `<g transform="rotate(angle 200 200)"><g transform="translate(200 112)">...</g></g>` guarantees exact angular symmetry.

## 5. Traveling-Wave Cubic Bézier Paths

Instead of rotating or skewing hair, ribbons, or fabric with CSS `skewX`, update cubic Bézier `C` control points with a phase lag between root, midpoint, and tip:

```js
const breeze = clamp(wind * 0.35, -10, 10);
const w1 = Math.sin(phase) * 3.5 + breeze * 0.45;       // mid-strand wave
const w2 = Math.sin(phase - 1.1) * 4.5 + breeze * 0.65; // tip lag wave

strand.setAttribute(
  'd',
  `M284 208 C264 ${(205 + w1 * 0.4).toFixed(1)}, 240 ${(218 + w2 * 0.7).toFixed(1)}, ${(214 - breeze * 0.4).toFixed(1)} ${(238 + w2).toFixed(1)} ` +
  `C${(206 - breeze * 0.4).toFixed(1)} ${(244 + w2).toFixed(1)}, ${(208 - breeze * 0.4).toFixed(1)} ${(246 + w2).toFixed(1)}, ${(217 - breeze * 0.3).toFixed(1)} ${(241 + w2 * 0.9).toFixed(1)} ` +
  `C242 ${(228 + w2 * 0.5).toFixed(1)}, 264 ${(218 + w1 * 0.3).toFixed(1)}, 285 220 Z`
);
```

Root coordinates (`M284 208` and ending `285 220`) stay fixed so the strand never detaches from the anchor.

## 6. Particle / Flock Scatter Math

Define particle items once with base positions `(bx, by)`, scatter vectors `(vx, vy)`, and base scale `s`:

```js
const scatter = smooth(0.3, 0.6, position);
nodes.forEach(({ g, c, i }) => {
  const x = c.bx + c.vx * scatter + Math.sin(phase * 0.9 + i) * 24 + wind * (i % 2 ? 1.2 : -1.2);
  const y = c.by + c.vy * scatter + Math.cos(phase * 1.1 + i) * 18;
  g.setAttribute('transform', `translate(${x.toFixed(1)} ${y.toFixed(1)}) scale(${(c.s * (1 + scatter * 0.3)).toFixed(2)})`);
});
```
