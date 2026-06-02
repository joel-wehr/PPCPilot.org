# PPCPilot — Native UI & Design-System Strategy

**Status:** Research + recommendation (2026-06-02)
**Scope:** iOS (`ppc-pilot-ios`, SwiftUI) and Android (`ppc-pilot-android`, Jetpack Compose), and how they relate to the web app's shadcn/ui system.
**TL;DR:** There is no single dominant "shadcn for native," and we shouldn't chase one. Adopt each platform's best-in-class native system — **Material 3 Expressive** on Android, **native SwiftUI + an owned token layer** on iOS — and unify all three apps (web/iOS/Android) with **one shared design-token source**. That shared-token, own-the-code approach *is* shadcn's real philosophy applied natively.

---

## 1. Why "shadcn for native" doesn't exist (and what to do instead)

shadcn/ui won on the web because the web had **no standard design system** and Tailwind made design tokens trivial — shadcn is "not a component library, it's how you build your component library" (open code you own, themed by CSS variables).

Native platforms already ship **strong first-party systems**:
- **iOS:** SwiftUI + SF Symbols + Apple Human Interface Guidelines (HIG), with built-in accessibility (VoiceOver/Dynamic Type).
- **Android:** Material 3 / Material 3 Expressive, a token-based, composable, accessible system maintained by Google.

So the best-in-class path is **adopt the native system and own a design-token layer on top** — not bolt on a heavy third-party "kit" that fights HIG/Material and accessibility.

**The portable lesson from shadcn:** *own your component layer, and drive everything from design tokens.*

---

## 2. iOS / SwiftUI

No true shadcn clone dominates. Recommended stack:

- **Base:** native **SwiftUI + SF Symbols**. Best HIG compliance, accessibility, and longevity. Don't replace native controls wholesale.
- **Owned token layer (the shadcn analog):** a single `DesignSystem.swift` holding colors, spacing, typography, corner radius, and semantic roles — edit one file, restyle the whole app. This mirrors shadcn's CSS-variable model.
- **Optional component lib:** **[ComponentsKit](https://componentskit.io/)** — free/OSS, SwiftUI **+** UIKit, ~20 themeable components (buttons, cards, inputs, modals…), iOS 15+. It's an installed dependency, not copy-paste; use it only where it beats rolling our own.
- **Utility / polish:** **[SwiftUIX](https://github.com/SwiftUIX/SwiftUIX)** (fills SwiftUI gaps), **Pow** (premium motion/transitions).
- **Pattern reference:** **[The Swift Kit](https://theswiftk.it.com/)** popularizes the centralized `DesignSystem.swift` token approach (paid kit — we take the pattern, not the product).
- **Discovery:** [awesome-swiftui-libraries](https://github.com/Toni77777/awesome-swiftui-libraries), [Swift Package Index – ui-components](https://swiftpackageindex.com/keywords/ui-components).

**iOS recommendation:** native SwiftUI + an owned `DesignSystem.swift` token system; reach for ComponentsKit/Pow only where they add clear value. Keep everything HIG- and accessibility-correct.

---

## 3. Android / Jetpack Compose

This platform has a clear winner that *is* essentially "shadcn for native" in spirit:

- **[Material 3 / Material 3 Expressive](https://developer.android.com/develop/ui/compose/designsystems/material3)** — official, composable, accessible, and **token-themed** via `MaterialTheme.colorScheme / typography / shapes`. You theme it to the brand; M3 Expressive (2025–26) adds richer motion, shape, and typography scales. ([M3 dev guide](https://m3.material.io/develop/android/jetpack-compose), [Compose + libraries](https://developer.android.com/develop/ui/compose/libraries))
- Third-party copy-paste Compose kits exist but are immature; **Material 3 is best-supported** (Google-maintained, dynamic color / Material You, accessibility). Our Android app already uses M3.

**Android recommendation:** **Material 3 Expressive**, themed with our brand tokens (a custom `ColorScheme`, `Typography`, and shape scale). No heavy third-party UI dependency.

**Architectural note — Compose Multiplatform:** [JetBrains' Compose Multiplatform](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-multiplatform-and-jetpack-compose.html) now runs Compose UI on iOS/desktop/web (with experimental `MaterialExpressiveTheme`). This is an *alternative architecture* to share one UI codebase across Android+iOS. **Not our current path** (we chose native SwiftUI for iOS for best HIG fit), but a real fallback if maintaining two native UIs becomes a burden.

---

## 4. The unifying recommendation: one token source, three apps

The highest-leverage move is a **single design-token source** that feeds all three apps so they feel like one product:

| App | UI system | Tokens consumed as |
|---|---|---|
| Web (`ppc-pilot-web`) | shadcn/ui + Tailwind v4 | CSS variables in `@theme` (`index.css`) — **done** |
| iOS (`ppc-pilot-ios`) | native SwiftUI | `DesignSystem.swift` (Color/Font/CGFloat constants) |
| Android (`ppc-pilot-android`) | Material 3 Expressive | `MaterialTheme` `ColorScheme` / `Typography` / shapes |

**Pipeline options (pick later):**
- **Manual mirror (start here):** maintain the tokens by hand in each app from this doc's table. Lowest setup, fine for our size.
- **Generated (scale later):** a [Style Dictionary](https://amzn.github.io/style-dictionary/)-style JSON token file → generates Tailwind CSS vars + `DesignSystem.swift` + Compose `Theme.kt`. True "design system as code." Aligns with the direction in `docs/STYLE_GUIDE_PROPOSALS.md`.

### Canonical brand tokens (source of truth)
Colors come from the web app's live theme (`ppc-pilot-web/frontend/src/index.css`). **Primary action = blue; navy is a retained accent — do NOT make primary navy** (see the web color-system note).

| Token | Value | Role |
|---|---|---|
| `primary` | `#3B82F6` | Primary action (Glass Cockpit blue) |
| `primary-hover` | `#2563EB` | Pressed/hover |
| `navy` | `#003366` | Brand accent (headers, emphasis) |
| `navy-dark` / `navy-light` | `#001F3F` / `#0055A5` | Accent shades |
| `accent` | `#FF9933` | Secondary accent (orange) |
| `success / warning / danger / info` | `#10B981 / #F59E0B / #EF4444 / #06B6D4` | Semantic status |
| Go/No-Go (assessment) | green/yellow/orange/red | Flyability levels |
| `radius` | `0.625rem` (~10px) | Corner radius base |
| Font (sans) | **Inter** | Body/UI |
| Font (mono) | **JetBrains Mono** | METAR/codes/numeric |

Spacing/typography scales: mirror Tailwind's defaults (4px base step) on each platform so layout rhythm matches the web.

---

## 5. Recommended next steps
1. **iOS:** add `DesignSystem.swift` with the tokens above and route existing screens through it (fold into a `ios-dev` increment). Optionally evaluate ComponentsKit for complex inputs.
2. **Android:** confirm the M3 `ColorScheme`/`Typography` match the canonical tokens; adopt M3 **Expressive** theme.
3. **Cross-cutting:** decide manual-mirror vs. generated (Style Dictionary) token pipeline. Start manual; revisit when a third surface (e.g., training platform) needs the same tokens.
4. Keep this doc as the brand-token source of truth; update it (not per-app hardcodes) when the palette changes.

## Sources
- shadcn/ui — philosophy & docs: https://ui.shadcn.com/docs
- Material 3 in Compose: https://developer.android.com/develop/ui/compose/designsystems/material3 · https://m3.material.io/develop/android/jetpack-compose
- Compose libraries: https://developer.android.com/develop/ui/compose/libraries
- Compose Multiplatform: https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-multiplatform-and-jetpack-compose.html
- ComponentsKit: https://componentskit.io/
- The Swift Kit (token pattern): https://theswiftk.it.com/
- SwiftUIX: https://github.com/SwiftUIX/SwiftUIX
- awesome-swiftui-libraries: https://github.com/Toni77777/awesome-swiftui-libraries
- Swift Package Index (ui-components): https://swiftpackageindex.com/keywords/ui-components
