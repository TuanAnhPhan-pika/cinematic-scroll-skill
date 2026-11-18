# E-Commerce & Product Showcase Adaptation Guide

How to repurpose the continuous-scroll storytelling engine into a high-converting product launch, luxury timepiece, sneaker, cosmetics, or hardware sales page.

## 1. Translating Manga Chapters into a 6-Stage Sales Funnel

Map the `700vh` continuous scroll (`6` chapters at `0` through `500vh`) directly to buyer psychology:

| Chapter (`position`) | Manga Equivalent | E-Commerce Funnel Role | Visual Action in `#world` |
| :--- | :--- | :--- | :--- |
| **01 (`0.00–0.16`)** | Silent Pole | **Hero Hook & First Impression** | Dramatic silhouette/rim-lit product reveal against ambient glow |
| **02 (`0.16–0.36`)** | Awakening Eye | **Core Tech / Signature Detail** | Outer casing/dial awakens; rings/gears spin with scroll |
| **03 (`0.36–0.56`)** | Night of Action | **Exploded Engineering / Ingredients** | Layers separate (`translate3d`), revealing internal build & materials |
| **04 (`0.56–0.76`)** | Mangekyō Zoom | **Macro Craftsmanship Close-Up** | Camera zooms (`scale(1.65)`) onto texture, lens, or precision movement |
| **05 (`0.76–0.90`)** | Emotional Peak | **Social Proof & Colorway Variants** | Product settles clean alongside variant switcher & trust metrics |
| **06 (`0.90–1.00`)** | Curtain & Finale | **Closing Offer & Conversion** | Ambient curtain frames final CTA, warranty badge, and quick checkout |

## 2. Interactive Controls Inside a Non-Blocking Stage

In storytelling mode, `#rail` and `#world` use `pointer-events: none`. In e-commerce mode, keep background visuals `pointer-events: none` while making UI controls interactive:

- **Sticky Bottom Conversion Dock (`#buyDock`)**:
  - `position: fixed; z-index: 40; pointer-events: auto;`
  - Shows active variant dot, product name, price, and primary `"ADD TO BAG"` / `"MUA NGAY"` button.
  - Opens a lightweight checkout/cart drawer (`#cartDrawer`) or triggers an inline toast confirmation without resetting scroll position.
- **Colorway / Edition Switcher**:
  - Bind variant buttons via CSS custom properties on `:root` or `#viewport` (`--accent`, `--accent-soft`, `--metal`, `--dial`)!
  - Changing `--accent` immediately recolors SVG gradients, progress bars, glow shadows, and exploded callouts **without interrupting `requestAnimationFrame` scroll math**.
- **Chapter Quick-Jump Navigation (Optional)**:
  - Clicking a stage pill scrolls smoothly to `track.offsetTop + stageIndex * (maxScroll / 5)` so buyers can jump straight to Specs or Buy.

## 3. Exploded-View Layering Recipe

Structure the product SVG with separate `<g>` layers (`#layerBack`, `#layerCore`, `#layerRing`, `#layerGlass`) centered around the same `viewBox`:

```js
const explode = smooth(0.32, 0.52, position) * (1 - smooth(0.58, 0.72, position));
layerBack.setAttribute('transform', `translate(0 ${(explode * 36).toFixed(1)})`);
layerRing.setAttribute('transform', `translate(0 ${(-explode * 28).toFixed(1)})`);
layerGlass.setAttribute('transform', `translate(0 ${(-explode * 54).toFixed(1)})`);
```

Notice how multiplying `smooth(0.32, 0.52, p) * (1 - smooth(0.58, 0.72, p))` automatically **explodes** the product open during Chapter 3 and **reassembles** it tightly before the Macro Zoom in Chapter 4!

## 4. Floating Spec Callouts Without Clutter

- Keep `.caption` cards under `260px` wide with **1 headline spec + 2 concise lines** (e.g., `"SAPPHIRE DOME · 0.8MM"` / `"Dual-coated anti-reflective crystal tested to 10 ATM."`).
- Stagger `.caption.left` and `.caption.right` vertically so the central product (`40%–60%` horizontal center) stays unobstructed.
- On mobile (`max-width: 650px`), dock `.caption` cards neatly above the sticky `#buyDock` (`bottom: 92px`) so text never collides with the Buy button.

## 5. Localization & Currency Formatting

When adapting for Vietnamese (`VNĐ`) or international (`USD`) storefronts:
- Format price cleanly (`4.890.000 ₫` or `$249`) in `#buyDock` and `#cartDrawer`.
- Pair short punchy benefit titles with concrete trust signals (e.g., `"BẢO HÀNH 5 NĂM"`, `"ĐỔI TRẢ 30 NGÀY"`, `"MIỄN PHÍ GIAO HÀNG"`).
