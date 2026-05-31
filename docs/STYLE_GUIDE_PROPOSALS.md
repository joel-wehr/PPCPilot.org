# PPC Pilot — Style Guide Proposals

Three aviation-themed visual directions, written as near-complete style guides so you can evaluate them as final products. Pick one (or call out the parts you want blended) and I'll produce the canonical `STYLE_GUIDE.md`, `tokens.json`, Tailwind theme, and native iOS (SwiftUI) / Android (Compose) theme files.

Status: **proposal — not yet adopted**.

---

## Shared Foundations

Theme-independent decisions that carry across all three directions.

### Spacing scale
4px base. `0`, `1` (4), `2` (8), `3` (12), `4` (16), `5` (20), `6` (24), `8` (32), `10` (40), `12` (48), `16` (64).

### Border radius
| Token | px | Use |
|---|---|---|
| `sm` | 4 | Chips, badges |
| `md` | 8 | Inputs, buttons |
| `lg` | 12 | Cards |
| `xl` | 16 | Modals, hero cards |
| `full` | 9999 | Pills, avatars |

### Elevation
Four levels — `surface` / `raised` / `overlay` / `modal`. Each direction tunes the actual shadow color or surface lightness.

### Motion
- **Fast** 150ms — hover, focus rings
- **Standard** 220ms — panels, tab switches
- **Slow** 320ms — modals, page transitions
- Easing: `cubic-bezier(0.4, 0, 0.2, 1)` standard / decelerate-in `cubic-bezier(0, 0, 0.2, 1)` / accelerate-out `cubic-bezier(0.4, 0, 1, 1)`

### Touch targets (mobile)
44×44pt minimum (iOS HIG). Checklist row 56pt. Map markers 48pt. Primary buttons 48pt min height.

### Type scale
| Role | Size | Line height |
|---|---|---|
| Display XL | 60 | 1.05 |
| Display L | 48 | 1.05 |
| Display M | 36 | 1.1 |
| Headline L | 30 | 1.15 |
| Headline M | 24 | 1.2 |
| Headline S | 20 | 1.25 |
| Body L | 18 | 1.5 |
| Body M | 16 | 1.5 |
| Body S | 14 | 1.45 |
| Caption | 12 | 1.4 |
| Eyebrow | 11 | 1.4 (tracking 0.1em uppercase) |

### Accessibility
- WCAG AA minimum (4.5:1 body, 3:1 large text).
- Semantic colors always paired with an icon — never color-alone.
- Focus ring: 2px solid + 2px offset, theme-tinted.
- All interactive elements have non-color hover/focus state changes.

### Iconography rules (all directions)
- Single stroke weight per surface.
- Sizes: 24px default, 20px inline w/ body, 32px feature.
- Custom aviation glyph set on top of any base library:
  wing, prop, METAR, sectional, AGL, hobbs, takeoff, landing,
  magnetic compass, fuel pressure, oil pressure, wind sock,
  temperature inversion, density altitude, runway, hangar,
  parachute canopy, throttle, magnetos, transponder.

### Charts
- Line: 2px stroke, markers on hover only, no minor gridlines.
- Bar: 2px gap, top corners radius `sm`.
- Always label axes; units in caption.
- Color order tied to semantic palette (primary → caution → info → success → muted).
- Mobile: thicker 2.5px strokes, 32% larger touch targets on data points.

---

## Direction 1: Modern Glass Cockpit *(recommended)*

The visual language pilots already parse instantly — Garmin G1000, Dynon SkyView, ForeFlight. Built for dense data and glanceability.

### Mood
Authoritative, technical, premium. Reads like a flight instrument, not a consumer app. The pilot trusts it because it looks like the panel they fly behind.

### Palette

**Surface (dark — primary mode)**
| Token | Hex | Use |
|---|---|---|
| `bg-base` | `#0B1220` | Screen root |
| `bg-panel` | `#0F1A2D` | Cards, elevated surfaces |
| `bg-raised` | `#16243F` | Hover/selected state |
| `bg-overlay` | `#0B1220E6` | Modal scrim (90%) |

**Surface (light — secondary mode)**
| Token | Hex | Use |
|---|---|---|
| `bg-light` | `#F8FAFC` | Light root |
| `bg-light-panel` | `#FFFFFF` | Cards |
| `bg-light-soft` | `#F1F5F9` | Sections |

**Brand (existing palette retained)**
| Token | Hex |
|---|---|
| `navy` | `#003366` |
| `navy-dark` | `#001F3F` |
| `navy-light` | `#0055A5` |

**Primary action (new)**
| Token | Hex | Note |
|---|---|---|
| `primary` | `#3B82F6` | Electric blue — primary CTAs, links, active states |
| `primary-hover` | `#2563EB` | |
| `primary-pressed` | `#1D4ED8` | |
| `primary-soft` | `#3B82F626` | Chip backgrounds (15% alpha) |

**Accent (existing orange — repositioned)**
| Token | Hex | Note |
|---|---|---|
| `accent` | `#FF9933` | Reserved for community, achievements, splash CTA |
| `accent-soft` | `#FF993326` | |

**Semantic**
| Token | Hex | Use |
|---|---|---|
| `success` | `#10B981` | OK, synced, complete |
| `warning` | `#F59E0B` | Caution, yellow band |
| `danger` | `#EF4444` | Abort, red band |
| `info` | `#06B6D4` | Informational, MVFR |
| `vfr` | `#10B981` | METAR VFR |
| `mvfr` | `#3B82F6` | METAR MVFR |
| `ifr` | `#EF4444` | METAR IFR |
| `lifr` | `#A855F7` | METAR LIFR |

**Text on dark**
| Token | Hex |
|---|---|
| `text-primary` | `#F8FAFC` |
| `text-secondary` | `#CBD5E1` |
| `text-muted` | `#64748B` |
| `text-disabled` | `#475569` |

**Borders**
| Token | Value |
|---|---|
| `border-subtle` | `rgba(255,255,255,0.08)` |
| `border-strong` | `rgba(255,255,255,0.16)` |
| `divider` | `rgba(255,255,255,0.06)` |

### Typography
- **UI / Body:** **Inter** (variable). 400 / 500 / 600 / 700.
- **Numerics:** **JetBrains Mono** 500 — altitude, airspeed, fuel, hobbs, registration numbers, ETAs, lat/long.
- **Display (marketing only):** Inter Display 700 — same family, no font swap penalty.
- **Eyebrow / labels:** Inter 600, tracking 0.1em, uppercase.

### Iconography
- **Base library:** **Lucide** (1500+ stroke icons, 1.5px standard).
- **Custom aviation glyph set:** ~20 SVGs on a 24×24 grid, 1.5px stroke, rounded caps.
- No filled icons — keep stroke language consistent with G1000 convention.

### Component patterns

**Cards**
- `bg-panel`, 1px `border-subtle`, radius `lg`.
- Elevation conveyed by surface lightness step, not drop shadow (Glass Cockpit convention).
- Hover state: surface → `bg-raised`.

**Buttons**
| Variant | Style |
|---|---|
| Primary | `bg-primary text-white`, hover `primary-hover` |
| Accent | `bg-accent text-navy-dark` (splash CTA, achievements) |
| Ghost | 1px `border-strong`, `text-primary` |
| Destructive | `bg-danger text-white` |

**Inputs**
- `bg-base`, 1px `border-strong`, focus 2px `primary` ring at 0 offset.
- Numeric inputs use JetBrains Mono.

**Checklist row**
- 56pt min height (64pt mobile).
- State icon replaces checkbox: `○` pending, `✓` complete (green), `⚠` flagged (amber).
- Long-press to flag — haptic feedback on mobile.
- Counter rows (in-flight practice) use mono numerics, +/- 44pt buttons.

**Weather panel**
- Mono numerics, color-banded by VFR/MVFR/IFR/LIFR.
- Wind: arrow rotation in degrees true, speed in mono.
- METAR raw shown in code-block style with semantic highlighting.

### Mobile-specific
- High contrast (≥7:1 for primary numerics) for sunlight readability.
- **Night mode** option: red-shifted palette (`#FF6B6B` headlines on `#0A0000` base). Preserves dark-adapted vision for dusk/dawn flights.
- Charts plotted with 2.5px strokes vs. 2px on web.
- Page padding 16 (mobile) → 24 (tablet) → 32 (desktop).

### Where it shines
Weather, flight assessment, currency, checklists, charts, mobile in-flight practice.

### Where it strains
Marketing surfaces feel utilitarian — pair with cinematic photo treatment for warmth (already shipped on splash).

---

## Direction 2: Heritage Cockpit

Brass-and-leather warmth — vintage navigator's logbook, sectional chart, compass rose.

### Mood
Warm, classical, hand-crafted. Reads like a well-worn pilot's logbook. Appeals to the community/tradition side of PPC flying.

### Palette

**Surface (light — primary mode)**
| Token | Hex | Use |
|---|---|---|
| `bg-paper` | `#FAF7F0` | Cream — primary surface |
| `bg-parchment` | `#F2EBDA` | Raised cards |
| `bg-vellum` | `#EDE3CC` | Section dividers |

**Surface (dark — secondary mode)**
| Token | Hex | Use |
|---|---|---|
| `bg-ink` | `#1A1A1A` | Dark root |
| `bg-leather` | `#2A1F1A` | Dark cards (warm not cool — preserves rod-adapted vision) |

**Brand**
| Token | Hex |
|---|---|
| `navy` | `#001F3F` (existing) |
| `brass` | `#B8860B` |
| `brass-light` | `#D4A85F` |
| `vintage-red` | `#A52A2A` |

**Semantic**
| Token | Hex | Note |
|---|---|---|
| `success` | `#5C8A3A` | Olive |
| `warning` | `#C28B1F` | Mustard |
| `danger` | `#A52A2A` | Vintage red |
| `info` | `#3D5A7C` | Slate blue |

**Text**
| Token | Hex |
|---|---|
| `text-primary` | `#1A1A1A` (on cream) / `#FAF7F0` (on ink) |
| `text-secondary` | `#5C5247` |
| `text-muted` | `#8A7E70` |

**Borders**
- `border-rule` `#1A1A1A` — 1px hairline (logbook column rules)
- Decorative double-rule for cards (top + bottom only)

### Typography
- **Display:** **Fraunces** (variable serif) — h1/h2 on marketing, training.
- **Body / UI:** **Inter** — UI legibility wins; serifs at small UI sizes fatigue.
- **Long-form (training modules):** **Source Serif 4**.
- **Numerics:** Inter tabular figures — no monospace, fits the navigator-log aesthetic.

### Iconography
- **Base:** **Phosphor Duotone** (1.5px stroke + filled accent layer — looks hand-drafted).
- **Aviation glyphs:** drawn in the same duotone style, drawing inspiration from sectional chart symbology.

### Component patterns
- **Cards:** parchment fill, hairline rule top + bottom only, **no corner radius** — like a logbook entry.
- **Buttons:** primary `bg-navy text-cream`, accent `bg-brass text-ink`. Small buttons square, large get radius `sm`.
- **Checklist:** rule-divider rows, brass checkmark, no fill change.
- **Charts:** dotted grid like a sectional, sepia-toned series.

### Mobile-specific
- Cream surfaces read well in sunlight without modification.
- Dark mode is leather brown not pure black (preserves dark adaptation).
- Serif fatigue at small sizes mitigated by keeping body Inter.

### Where it shines
Training content, community profiles, equipment "logbook," maintenance records.

### Where it strains
Glanceable mobile data — METAR tables and counters lose contrast against parchment. Charts feel decorative rather than authoritative.

---

## Direction 3: Sky & Field

Soft modern SaaS — sunrise palette, friendly geometry. Calm/Headspace-adjacent.

### Mood
Approachable, optimistic, contemporary. Lowers the barrier for new pilots and non-pilot family members.

### Palette

**Surface (light — primary)**
| Token | Hex | Use |
|---|---|---|
| `bg-base` | `#FBFAF7` | Off-white root |
| `bg-soft` | `#F1F5F9` | Sections |
| `bg-panel` | `#FFFFFF` | Cards |

**Surface (dark — secondary)**
| Token | Hex |
|---|---|
| `bg-dark` | `#1E293B` |
| `bg-dark-panel` | `#334155` |

**Brand**
| Token | Hex | Use |
|---|---|---|
| `sky` | `#0EA5E9` | Primary |
| `sky-deep` | `#0369A1` | Hover, gradient bottom |
| `sunrise` | `#F97316` | Accent |
| `sand` | `#FCD7AC` | Warm fill, illustrations |

**Semantic**
| Token | Hex |
|---|---|
| `success` | `#22C55E` |
| `warning` | `#F59E0B` |
| `danger` | `#EF4444` |
| `info` | `#0EA5E9` |

**Text**
| Token | Hex |
|---|---|
| `text-primary` | `#0F172A` |
| `text-secondary` | `#475569` |
| `text-muted` | `#94A3B8` |

### Typography
- **Display:** **Space Grotesk** 600 — friendly geometric, slight quirk.
- **Body / UI:** **Inter**.
- **Numerics:** Inter tabular figures.

### Iconography
- **Base:** **Phosphor Regular** (rounded terminals, 1.5px) — most approachable major library.
- **Aviation glyphs:** drawn in the same rounded style.

### Component patterns
- **Cards:** white fill, soft 8% shadow, radius `lg` → `xl` for hero cards.
- **Buttons:**
  - Primary: `bg-sky text-white` w/ subtle gradient `from-sky to-sky-deep`.
  - Accent: `bg-sunrise text-white`.
- **Checklist:** rounded rows, sky-tinted hover, sand fill on completed.
- **Charts:** soft gradients under area charts, rounded bars.

### Mobile-specific
- Light surfaces dominate — body text must be near-black `#0F172A` (not slate) for outdoor brightness.
- Dark mode genuinely required for any in-flight use; light mode washes out at altitude.

### Where it shines
Onboarding, marketing, community map, social feed.

### Where it strains
Numerics-dense screens look softer / less authoritative. Pilots accustomed to ForeFlight may read it as a consumer app rather than a flight tool.

---

## Comparison matrix

| Need | Glass Cockpit | Heritage | Sky & Field |
|---|:---:|:---:|:---:|
| Mobile glanceability | ★★★ | ★ | ★★ |
| Weather / charts authority | ★★★ | ★★ | ★★ |
| Checklist clarity | ★★★ | ★★ | ★★ |
| Marketing / onboarding warmth | ★★ | ★★★ | ★★★ |
| Training content readability | ★★ | ★★★ | ★★ |
| Community / social tone | ★★ | ★★ | ★★★ |
| Brand differentiation | ★★★ | ★★★ | ★ |
| Effort to build (tokens + assets) | Medium | High (custom glyphs) | Low |
| Reuses existing navy/orange | ★★★ | ★★★ | ★ |

---

## Recommendation

**Glass Cockpit** as the system, with two named exceptions:

1. **Marketing / splash:** keep cinematic photo treatment + accent orange (already shipped — no rework).
2. **Training content** (`ppc-knowledgebase`, future training-platform): allow Heritage's serif body (Source Serif 4) for long-form readability.

This gives one design system everywhere except long-form prose — where the user isn't context-switching mid-task, so the seam doesn't hurt.

---

## Next steps after a direction is picked

I'll produce in `docs/`:

1. **`STYLE_GUIDE.md`** — canonical guide for the chosen direction (full token reference, component specs, do/don't examples).
2. **`tokens.json`** — Style Dictionary–format design tokens. Single source of truth, feeds Tailwind, SwiftUI, and Compose.
3. **`tailwind.theme.js`** — drop-in replacement for the web `@theme` block in `index.css`.
4. **iOS `Color` asset catalog + Android `Color.kt`/theme** snippets — generated from the tokens for SwiftUI and Jetpack Compose.
5. **`ICONOGRAPHY.md`** — Lucide rules, custom aviation glyph spec sheet with SVGs on the 24×24 grid.

Adoption is non-breaking on web (existing navy/orange tokens stay; new tokens layer on top). The native apps get a one-time pass to swap hardcoded colors → token references.
