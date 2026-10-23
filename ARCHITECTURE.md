# ARCHITECTURE // How this profile is built

The banner at the top of my profile is not a template, a GIF, or a screenshot. It is a
single hand-authored SVG file — **1,110 lines and 271 animations**, all declarative, with
no JavaScript and no external assets. This document explains how it works and why it is
built the way it is.

---

## The constraint that shapes everything

GitHub serves README images through a caching image proxy (`camo`). That proxy imposes
hard limits on what an animated profile banner can be:

| Constraint | Consequence |
|:---|:---|
| **Scripts are stripped** | No JavaScript. Every animation must be declarative SMIL. |
| **No external requests** | No web fonts, no CDN, no remote images. The file must be self-contained. |
| **No interactivity** | `:hover` and pointer events never fire. Motion has to be autonomous. |
| **Cached aggressively** | The file must look right on first paint, with no loading state. |

So the whole banner is SMIL — `<animate>`, `<animateTransform>`, and `<animateMotion>` —
using only system font stacks (`Consolas`, `Segoe UI`, `Georgia`) that resolve on every
platform.

---

## Animation architecture

Motion is split into **independent concurrent loops** rather than one master timeline.
Each system runs on its own period, so they drift against each other and the banner rarely
repeats the same combination twice.

| System | Period | What it does |
|:---|:---:|:---|
| **Boot sequence** | one-shot | JARVIS boot lines, name reveal wipe, armor suit-up |
| **Story loop** | 20s | Decode → scan → identity verified → target lock → power 0–100% → launch |
| **Typography modes** | 12s | Name cycles through futuristic, analysis, and armor typefaces |
| **Repair drone** | 15s | Drone patrols the plating and stops to weld |
| **Blueprint X-ray** | 24s | Reactor resolves into a technical schematic with dimensions |
| **Nanite self-heal** | 26s | Hull breach opens; nanites swarm in and seal it |
| **Aux power** | 32s | Brownout flicker, emergency rails, grid recovery |

Choosing co-prime-ish periods (12, 15, 20, 24, 26, 32) means the blueprint sweep and the
power-up rarely coincide, and when they do it reads as a busy diagnostic moment rather
than a scripted beat.

### One-shot vs looping

Boot animations use `fill="freeze"` so they play once and hold their final state:

```xml
<animateTransform attributeName="transform" type="translate"
                  from="-470 -130" to="0 0" dur="0.85s" begin="0.05s"
                  calcMode="spline" keySplines="0.16 1 0.3 1" fill="freeze"/>
```

That is one of the four armor plates flying into place. The `keySplines` value
`0.16 1 0.3 1` is a strong ease-out — the plate arrives fast and settles, the way a
physical object with mass would.

### Easing

SMIL defaults to **linear** interpolation, which is why most animated SVG banners feel
mechanical. Every breathing, pulsing, and sweeping animation here is spline-eased instead —
**118 of them**:

```xml
<animate attributeName="r" values="66;70;66" dur="2s"
         calcMode="spline"
         keySplines="0.42 0 0.58 1;0.42 0 0.58 1"
         repeatCount="indefinite"/>
```

`0.42 0 0.58 1` is the cubic-bezier behind CSS `ease-in-out`. Different curves do
different jobs:

| Curve | Feel | Used for |
|:---|:---|:---|
| `0.42 0 0.58 1` | Symmetric ease-in-out | Breathing, pulsing, hovering |
| `0.16 1 0.3 1` | Sharp ease-out | Plates locking, nanites arriving |
| `0.2 0.7 0.4 1` | Explosive decay | Shockwaves, repulsor bursts |
| `0 0 1 1` | Linear | Hold segments between eased moves |

A three-segment sweep that waits, glides, then waits combines them:
`keySplines="0 0 1 1;0.42 0 0.58 1;0 0 1 1"`.

---

## The silent-failure trap

In SMIL, `values`, `keyTimes`, and `keySplines` must agree in count:

- `keyTimes` must have **exactly as many entries** as `values`
- `keySplines` must have **exactly one fewer** (it describes the segments *between* values)

Get it wrong and the browser does not warn — **the animation is silently dropped**. The
element just sits there, and you have no idea which of 271 animations broke.

So changes are validated mechanically rather than by eye:

```python
nspl = len(keySplines.split(';'))
nseg = len(values.split(';')) - 1
assert nspl == nseg, tag        # catches the silent killer
assert len(keyTimes.split(';')) == len(values.split(';'))
```

---

## Verifying motion without watching it

Screenshotting an animated SVG gives you whatever frame the renderer happened to be on —
useless for checking a beat that happens at 12 seconds. Instead the SMIL clock is driven
directly, which makes any moment reproducible:

```js
const svg = document.getElementById('hud');
svg.pauseAnimations();
svg.setCurrentTime(12.0);   // jump to the blueprint reveal
```

Rendered headless in Chromium, this confirms exact states: plates mid-flight during boot,
the nanite seal ring closed at `t=10.4s`, blueprint callouts clear of the text panels at
`t=12s`, aux power readable at `t=30s`.

---

## Asset inventory

| File | Lines | Animations | Role |
|:---|:---:|:---:|:---|
| `assets/profile-banner.svg` | 1,110 | 271 | The main HUD banner |
| `assets/skill-radar.svg` | 78 | 8 | Capability radar with a polygon that locks in on load |
| `assets/jarvis-quotes.svg` | 73 | 11 | Rotating quote archive console |
| `assets/energy-divider.svg` | 45 | 7 | Animated section rule used between every section |

### Palette

| Token | Hex | Use |
|:---|:---|:---|
| Arc cyan | `#2BE7FF` / `#5FFFEA` | Primary energy, JARVIS systems |
| Armor gold | `#F6C453` | Trim, alerts, secondary energy |
| Armor red | `#A41D23` | Plating, structural mass |
| Deep space | `#06141D` | Backgrounds |

---

## Automation

Three GitHub Actions workflows keep the generated pieces current, so the profile updates
without manual work:

| Workflow | Schedule | Output |
|:---|:---|:---|
| `readme-cards.yml` | daily 03:17 UTC | `profile/stats.svg`, `profile/top-langs.svg` |
| `snake.yml` | every 12h | Contribution snake on the `output` branch |
| `3d-contrib.yml` | daily 18:00 UTC | `profile-3d-contrib/` skyline renders |

Self-hosting the stats cards means the profile still renders if an upstream service is
rate-limited or down.

---

## Reusing this

The SVGs are plain text — open one and read it. If you want the same approach for your own
profile, the transferable parts are: pick independent loop periods, ease everything that
repeats, keep one-shots on `fill="freeze"`, validate your `keySplines` counts, and verify
with `setCurrentTime` instead of hoping.
