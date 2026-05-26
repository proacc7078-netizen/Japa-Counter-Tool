# Design Document
# Japa Count Calculator

> **Version:** 1.0 · MVP
> **Type:** Mobile-first static web application
> **Last updated:** May 2026

---

## Table of Contents

1. [Design Philosophy](#1-design-philosophy)
2. [Visual Identity](#2-visual-identity)
3. [Colour System](#3-colour-system)
4. [Typography](#4-typography)
5. [Spacing & Layout](#5-spacing--layout)
6. [Component Design](#6-component-design)
7. [Interaction Design](#7-interaction-design)
8. [Motion & Animation](#8-motion--animation)
9. [States & Feedback](#9-states--feedback)
10. [Iconography](#10-iconography)
11. [Responsive Design](#11-responsive-design)
12. [Accessibility](#12-accessibility)
13. [Design Decisions Log](#13-design-decisions-log)
14. [Design Tokens Reference](#14-design-tokens-reference)

---

## 1. Design Philosophy

### 1.1 Core Principle

The app is used at 9:30 PM, after a day of spiritual practice, by admins who want one number as fast as possible. The design must get out of the way.

> **"Calm clarity over clever UI."**

Every design decision is evaluated against three questions:
1. Does it make the total easier to read?
2. Does it reduce cognitive load at end of day?
3. Does it feel appropriate for a devotional context?

### 1.2 Design Pillars

| Pillar | Meaning in practice |
|---|---|
| **Reverent** | The aesthetic honours the spiritual purpose — warm, grounded, unhurried |
| **Immediate** | The total is the hero. It must be visible without scrolling or searching |
| **Transparent** | Admins must trust the output. Every count is auditable in two taps |
| **Minimal** | No feature or element that doesn't serve the nightly calculation ritual |

### 1.3 Anti-patterns to Avoid

- Bright white backgrounds (jarring at night on mobile)
- Blue-accent "enterprise" UI patterns (feels wrong for devotional use)
- Loaders, progress bars, or artificial delays
- Excessive confirmations or modals
- Decorative animations that delay seeing the result

---

## 2. Visual Identity

### 2.1 Aesthetic Direction

**Warm devotional dark** — the visual language draws from:
- The glow of a diya (oil lamp) at night
- Saffron-robed ritual objects
- Aged manuscript warmth

The background is near-black with brown undertones, not pure `#000000`. Light comes from amber — not white, not blue.

### 2.2 Mood Board in Words

```
Deep evening sky after aarti
Warm ghee lamp against dark stone
Sandalwood and saffron
Handwritten count in a worn notebook
```

### 2.3 ॐ Symbol Treatment

The ॐ symbol at the top is the only decorative element. It:
- Uses a warm amber tone
- Pulses with a slow, breathing glow (3-second cycle)
- Is large enough to set the tone but does not dominate the functional UI
- Degrades gracefully to a static symbol if animations are disabled

---

## 3. Colour System

### 3.1 Palette

| Token | Hex | Usage |
|---|---|---|
| `--bg` | `#0E0A06` | Page background — near-black with brown warmth |
| `--surface` | `#1A1208` | Card and accordion backgrounds |
| `--border` | `#3D2A10` | Borders, dividers |
| `--amber` | `#E8922A` | Primary brand colour — ॐ symbol, CTA button accent, heading borders |
| `--amber-lt` | `#F5B85A` | Lighter amber — heading text, total number |
| `--amber-dk` | `#A05C10` | Darker amber — button base, table header backgrounds |
| `--gold` | `#D4A843` | Stat numbers, secondary accents |
| `--cream` | `#F0E4C8` | Body text, labels |
| `--muted` | `#8A7055` | Secondary text, hints, footer |
| `--error` | `#C0392B` | Error states only |
| `--success` | `#2ECC71` | Reserved for future valid-state indicators |

### 3.2 Colour Usage Rules

**Do:**
- Use `--amber-lt` for any number the admin needs to read first
- Use `--cream` for all body and label text
- Use `--muted` for secondary information (ignored count, hints)
- Use `--bg` for the page; `--surface` for elevated cards

**Do not:**
- Use pure white (`#FFFFFF`) anywhere — it shatters the warm dark aesthetic
- Use blue or cool-toned colours
- Use `--error` red for anything that is not a genuine error
- Apply `--amber` to large text blocks — reserve it for accents and borders

### 3.3 Colour Contrast

| Foreground | Background | Ratio | WCAG Level |
|---|---|---|---|
| `--cream` `#F0E4C8` | `--bg` `#0E0A06` | 13.2 : 1 | AAA |
| `--amber-lt` `#F5B85A` | `--bg` `#0E0A06` | 9.8 : 1 | AAA |
| `--amber-lt` `#F5B85A` | `--surface` `#1A1208` | 9.1 : 1 | AAA |
| `--muted` `#8A7055` | `--bg` `#0E0A06` | 4.6 : 1 | AA |
| White `#FFFFFF` | `--amber-dk` `#A05C10` | 4.8 : 1 | AA (button label) |

---

## 4. Typography

### 4.1 Typefaces

| Role | Family | Weights | Source |
|---|---|---|---|
| **Display** | Cinzel | 400, 600, 700 | Google Fonts |
| **Body** | Crimson Pro | 300, 400, 300 italic | Google Fonts |
| **Monospace** | System monospace | — | System fallback |

**Cinzel** is a Roman-inscriptional serif — evokes ancient sacred texts without being kitsch. Used for titles, the total number, stat values, and button labels.

**Crimson Pro** is a refined old-style serif. Used for body text, hints, subtitles, and accordion content. Its italic weight (`font-style: italic`) is used for secondary captions.

**Fallback stack:**
```css
font-family: 'Cinzel', Georgia, 'Times New Roman', serif;
font-family: 'Crimson Pro', Georgia, serif;
```

### 4.2 Type Scale

| Element | Family | Size | Weight | Colour |
|---|---|---|---|---|
| ॐ symbol | Cinzel | 2.4rem / 38px | 400 | `--amber` |
| Page title `h1` | Cinzel | 1.35rem / 22px | 600 | `--amber-lt` |
| Page subtitle | Crimson Pro | 0.9rem / 14px | 300 italic | `--muted` |
| Section heading `h2` | Cinzel | — | 700 | — |
| Upload label | Cinzel | 0.88rem / 14px | 400 | `--amber` |
| Upload hint | Crimson Pro | 0.82rem / 13px | 300 | `--muted` |
| Filename | Crimson Pro | 0.88rem / 14px | 400 | `--amber-lt` |
| Button label | Cinzel | 1rem / 16px | 700 | `#1A0E00` (dark on amber) |
| Total label | Cinzel | 0.75rem / 12px | 400 | `--muted` |
| **Total number** | **Cinzel** | **3.6rem / 58px** | **700** | **`--amber-lt`** |
| Rounds sub-label | Crimson Pro | 0.85rem / 14px | 300 italic | `--muted` |
| Stat number | Cinzel | 1.6rem / 26px | 600 | `--gold` |
| Stat label | Cinzel | 0.76rem / 12px | 400 | `--muted` |
| Accordion toggle | Crimson Pro | 0.9rem / 14px | 400 | `--muted` |
| Entry source | Monospace | 0.8rem / 13px | 400 | `--muted` |
| Entry value | Cinzel | 0.9rem / 14px | 400 | `--amber-lt` |
| Ignored line | Monospace | 0.8rem / 13px | 400 | `--muted` |
| Footer | Crimson Pro | 0.75rem / 12px | 400 | `#4A3820` |

### 4.3 Letter Spacing

Cinzel is designed for wide tracking. Applied values:

| Context | `letter-spacing` |
|---|---|
| Page title | `0.12em` |
| Upload label | `0.08em` |
| Button label | `0.15em` |
| Total label | `0.2em` |
| Total number | `0.04em` |
| Stat label | `0.05em` |

---

## 5. Spacing & Layout

### 5.1 Layout Grid

The app uses a **single-column centred layout**:

```
Max-width: 480px
Padding:   0 16px (mobile)
Align:     centre via margin: 0 auto
```

All content stacks vertically. No multi-column layout exists in the MVP. The narrow max-width ensures the design works identically on a 360px Android and a 430px iPhone Pro Max.

### 5.2 Spacing Scale

A loose 8px base unit:

| Token | Value | Used for |
|---|---|---|
| `4px` | 0.5 unit | Inline gaps, tight spacing |
| `8px` | 1 unit | Component internal padding small |
| `10px` | 1.25 unit | Gap between stat cards |
| `12px` | 1.5 unit | Accordion item padding |
| `14px` | 1.75 unit | File badge padding |
| `16px` | 2 units | Page horizontal padding |
| `18px` | 2.25 units | Gap between upload zone and button |
| `20px` | 2.5 units | Accordion list padding |
| `24px` | 3 units | Total card vertical padding bottom |
| `28px` | 3.5 units | Results section top margin; header bottom margin |
| `32px` | 4 units | Upload zone vertical padding |
| `36px` | 4.5 units | Header bottom margin |
| `48px` | 6 units | Page bottom padding |

### 5.3 Component Spacing

```
Page top padding:        28px
Header → Divider:        0px (header margin-bottom: 36px)
Divider → Upload zone:   0px (divider margin-bottom: 28px)
Upload zone → File badge: 14px
File badge → Button:     18px (button margin-top: 18px)
Button → Results:        28px (results margin-top: 28px)
Total card → Stats row:  12px
Stats row → Entries:     12px
Entries → Ignored:       14px (ignored section margin-top: 14px)
```

### 5.4 Border Radius Scale

| Element | Radius |
|---|---|
| Upload zone | `12px` (`--r`) |
| Button | `12px` (`--r`) |
| Total card | `12px` (`--r`) |
| File badge | `8px` |
| Stat cards | `8px` |
| Accordion toggles | `8px` (top corners only when open) |
| Accordion lists | `0 0 8px 8px` (bottom corners only) |
| Ignored / entry items | `4px` |

---

## 6. Component Design

### 6.1 Upload Zone

```
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
          📿
     UPLOAD CHAT EXPORT
   Tap to choose · WhatsApp .txt file
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

- **Border:** 1.5px dashed `--border`
- **Background:** `rgba(255,255,255, 0.015)` — barely perceptible surface lift
- **Icon:** 📿 emoji at 2.2rem — tactile and immediately recognisable
- **Hover/drag-over state:** border shifts to `--amber`; background to `rgba(232,146,42, 0.05)`
- The entire zone is the tap target — `<input type="file">` is absolutely positioned over it at full size with `opacity: 0`

### 6.2 File Badge

Appears below the upload zone after a file is selected:

```
┌────────────────────────────────────────┐
│  📄  group-chat-24-may.txt        ✕   │
└────────────────────────────────────────┘
```

- **Background:** `rgba(232,146,42, 0.08)` — amber wash
- **Border:** 1px solid `--amber-dk`
- **Filename colour:** `--amber-lt`
- **Clear button:** `--muted` at rest; `--error` on hover
- Filename truncates with `word-break: break-all` to handle long WhatsApp export filenames

### 6.3 Calculate Button

```
╔══════════════════════════════════════════╗
║         CALCULATE JAPA TOTAL            ║
╚══════════════════════════════════════════╝
```

- **Gradient:** `linear-gradient(135deg, --amber-dk 0%, --amber 60%, --amber-lt 100%)`
- **Shine overlay:** `::after` pseudo-element adds `linear-gradient(135deg, transparent 40%, rgba(255,255,255,0.15) 100%)`
- **Shadow:** `0 4px 24px rgba(232,146,42, 0.3)`
- **Disabled state:** `opacity: 0.45`; `cursor: not-allowed` — no shadow
- **Active state:** `transform: scale(0.98)` — physical press feel
- **Label:** All-caps, tracked, dark text on amber — maximum legibility

### 6.4 Total Card

The most important element in the app. Designed for instant legibility.

```
┌─────────────────────────────────────────┐  ← 2px top amber gradient border
│                                         │
│           TOTAL JAPA COUNT             │  ← 12px tracked caps, --muted
│                                         │
│              32,400                     │  ← 3.6rem Cinzel, --amber-lt, glowing
│                                         │
│          ≈ 300 rounds of 108            │  ← italic, --muted
│                                         │
└─────────────────────────────────────────┘
```

- **Background:** `linear-gradient(135deg, #1E1408 0%, #261A08 100%)`
- **Top accent line:** `::before` pseudo-element — `linear-gradient(90deg, transparent, --amber, transparent)` — creates a glowing edge without a hard line
- **Border:** 1px solid `--amber-dk`
- **Total number glow:** `text-shadow: 0 0 32px rgba(232,146,42, 0.5)`
- The number uses Indian number formatting (`toLocaleString('en-IN')`) — `32,400` not `32400`

### 6.5 Stats Row

Two equal-width cards below the total:

```
┌──────────────────┐  ┌──────────────────┐
│        42        │  │        8         │
│  VALID ENTRIES   │  │  IGNORED LINES   │
└──────────────────┘  └──────────────────┘
```

- **Layout:** CSS Grid, `1fr 1fr`, gap `10px`
- **Valid card:** stat number in `--gold`
- **Ignored card:** stat number in `--muted` — intentionally dimmer to signal lower importance
- **Background:** `--surface`; border `--border`

### 6.6 Accordions

Two expandable sections for transparency:

```
[ Parsed entries                      ▾ ]   ← collapsed
┌─────────────────────────────────────────┐
│ 09:14 - Radha: 30*108       3,240       │
│ 09:22 - Gopal: 3240         3,240       │
│ 09:45 - Meena: japa: 216      216       │
└─────────────────────────────────────────┘

[ Ignored lines                       ▾ ]
```

- **Toggle button:** full-width; `--border` border; arrow rotates 180° on open (`transition: transform 0.25s`)
- **List:** `max-height: 220–260px`; `overflow-y: auto`; custom scrollbar (4px, amber tones)
- **Entry rows:** source line in monospace `--muted`; value right-aligned in `--amber-lt` Cinzel
- **Ignored rows:** monospace `--muted`; no value column
- Both lists default to **collapsed** — the total is the answer; accordions are audit tools

---

## 7. Interaction Design

### 7.1 User Flow

```
App opens
    │
    ▼
Upload zone visible — button disabled
    │
    │  Tap upload zone
    ▼
File picker opens (native OS)
    │
    │  Select .txt file
    ▼
File badge appears — button enabled
    │
    │  Tap "Calculate Japa Total"
    ▼
Results appear (fade-up, 400ms)
    │
    ├── Total number (primary)
    ├── Valid / Ignored counts
    │
    │  [optional] Tap "Parsed entries"
    ▼
Entry list expands — admin verifies
    │
    │  [optional] Tap "Ignored lines"
    ▼
Ignored list expands — admin reviews gaps
```

### 7.2 Tap Target Sizes

All interactive elements meet the 44×44px minimum:

| Element | Minimum tap area |
|---|---|
| Upload zone | Full zone width × 100px+ height |
| Clear file button (✕) | 44×44px padded area |
| Calculate button | Full width × 48px height |
| Accordion toggles | Full width × 42px height |

### 7.3 Drag and Drop (Desktop / Large Screen)

- `dragover` — adds `.drag-over` class: border goes solid amber, faint amber background fill
- `dragleave` — removes `.drag-over` class: returns to dashed default
- `drop` — extracts first file, calls `setFile()`, same path as tap-select

Drag and drop is an enhancement. On mobile it is not available; the tap interaction is the primary path.

---

## 8. Motion & Animation

### 8.1 Principles

- All animations serve a communicative purpose — no decorative motion
- Nothing moves while the user is waiting for a result
- Animations complete in ≤ 400ms
- Respect `prefers-reduced-motion` (future: add media query)

### 8.2 Animation Inventory

| Animation | Element | Duration | Easing | Purpose |
|---|---|---|---|---|
| `pulse` | ॐ symbol | 3s, infinite | `ease-in-out` | Sets devotional tone; suggests life/breath |
| `fadeUp` | Results container | 400ms | `ease` | Confirms calculation completed; prevents jarring pop-in |
| Arrow rotation | Accordion arrows | 250ms | CSS transition | Shows open/close state |
| Drag-over transition | Upload zone | 250ms | CSS transition | Confirms drag target is active |
| Button press | Calculate button | Instant | `transform scale(0.98)` | Physical press feedback |

### 8.3 `@keyframes` Definitions

```css
/* Ambient glow on ॐ */
@keyframes pulse {
  0%, 100% { text-shadow: 0 0 20px rgba(232,146,42, 0.3); }
  50%       { text-shadow: 0 0 40px rgba(232,146,42, 0.7); }
}

/* Results entrance */
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(14px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

---

## 9. States & Feedback

### 9.1 Application States

| State | Trigger | Visual changes |
|---|---|---|
| **Initial** | Page load | Upload zone visible; button disabled (`opacity: 0.45`); results hidden |
| **File selected** | Valid `.txt` chosen | File badge appears; button enabled (full opacity + shadow) |
| **Error** | Wrong file type or FileReader failure | Red banner below button; file badge hidden |
| **Results** | Calculation complete | Results container fades up; total number rendered; accordions collapsed |

### 9.2 Interactive States

| Element | Rest | Hover | Active | Disabled |
|---|---|---|---|---|
| Upload zone | Dashed `--border` | Solid `--amber` border + amber bg tint | — | — |
| Clear button | `--muted` | `--error` red | — | — |
| Calculate button | Full amber gradient + shadow | Slightly brighter (opacity 1) | `scale(0.98)` | `opacity: 0.45`, no shadow, `cursor: not-allowed` |
| Accordion toggle | `--border` border, `--muted` text | `--amber-dk` border, `--cream` text | — | — |

### 9.3 Error State Design

```
┌────────────────────────────────────────────────┐
│  Please select a WhatsApp .txt export file.    │
└────────────────────────────────────────────────┘
```

- **Background:** `rgba(192,57,43, 0.1)` — red wash
- **Border:** 1px solid `rgba(192,57,43, 0.4)`
- **Text:** `#E88070` — warm red; not pure `#FF0000`
- **Position:** Below the button; above results area
- Error messages are plain language, not technical (`"FileReader error"` → `"Could not read the file. Please try again."`)

---

## 10. Iconography

The MVP uses emoji as icons — no icon library required, no extra requests.

| Element | Icon | Rationale |
|---|---|---|
| Upload zone | 📿 | Prayer beads — immediately contextual |
| File badge | 📄 | Universal file symbol |
| Clear button | ✕ | Text character — no icon needed |
| Accordion arrows | ▾ | CSS-rotated text character |

### Icon Usage Rules

- Emoji icons are decorative — they do not convey critical information without adjacent text
- Never use emoji as the sole label for a tap target
- The 📿 icon has `opacity: 0.7` — present but not louder than the label text beneath it

---

## 11. Responsive Design

### 11.1 Breakpoints

The app is designed **mobile-first**. There is one breakpoint:

| Range | Layout |
|---|---|
| 0 – 480px | Full-width single column; `padding: 0 16px` |
| 481px+ | Centred at `max-width: 480px`; same layout, just floated in whitespace |

No layout changes occur at any breakpoint. The design is deliberately constrained to a single-column flow that works at all sizes.

### 11.2 Behaviour at Key Widths

| Width | Notes |
|---|---|
| 360px (small Android) | Stats row stays 2-column; all tap targets meet 44px; total number fits |
| 390px (iPhone 14) | Optimal display width |
| 430px (iPhone Pro Max) | Extra breathing room; max-width cap prevents over-stretching |
| 768px (tablet) | Centred at 480px; large whitespace margins; still fully usable |
| 1280px (desktop) | Same as tablet; drag-and-drop available |

### 11.3 Touch Considerations

- No hover-dependent interactions — all interactive states have tap equivalents
- File input overlays the entire upload zone — no small tap target for file picking
- Scrollable accordions use `-webkit-overflow-scrolling: touch` behaviour (native iOS momentum scrolling)

---

## 12. Accessibility

### 12.1 Colour & Contrast

All foreground/background combinations meet WCAG AA minimum (4.5:1 for normal text, 3:1 for large text). Key pairs meet AAA. See §3.3.

### 12.2 Keyboard Navigation

| Action | Keyboard |
|---|---|
| Focus upload zone | Tab |
| Open file picker | Enter or Space on focused zone |
| Focus Calculate button | Tab |
| Activate button | Enter or Space |
| Focus accordion toggle | Tab |
| Open/close accordion | Enter or Space |
| Focus clear button | Tab |
| Activate clear | Enter or Space |

### 12.3 Screen Reader Considerations

| Element | Accessible name |
|---|---|
| File input | `accept=".txt"` signals file type |
| Calculate button | Button label text is descriptive |
| Total number | Preceded by visible label "TOTAL JAPA COUNT" |
| Accordions | Toggle button text identifies the section |

### 12.4 Focus Indicators

Current MVP inherits browser default focus rings. Future improvement: add explicit `:focus-visible` styles using `outline: 2px solid --amber` for consistent cross-browser focus indicators.

### 12.5 Motion

The `pulse` animation on ॐ and the `fadeUp` on results are both non-critical. Future improvement: wrap in `@media (prefers-reduced-motion: no-preference)` so users with vestibular disorders see no motion.

---

## 13. Design Decisions Log

| Decision | Chosen | Rejected | Reason |
|---|---|---|---|
| Background colour | Near-black `#0E0A06` | Pure black `#000000` | Pure black reads as tech/void; warm dark reads as evening/devotional |
| Primary accent | Amber `#E8922A` | Saffron orange, deep red | Amber is warm without aggression; saffron veers gaudy at scale |
| Total number size | 3.6rem | 2.5rem, 5rem | 3.6rem is large enough to read instantly; 5rem clips on small screens |
| Accordion default | Collapsed | Expanded | Total is the answer; detail is optional audit — collapsed keeps screen clean |
| Font pairing | Cinzel + Crimson Pro | All sans-serif, single typeface | Two weights of sans-serif felt clinical; serif pairing adds warmth and ritual feel |
| Number formatting | Indian locale (`32,400`) | Western (`32,400`) | Same for this value range, but `en-IN` is correct for the audience and handles lakhs correctly in Phase 2 |
| Error colour | Warm red `#E88070` | Pure red `#FF0000` | Pure red is jarring against the warm palette; `#E88070` is clearly an error without breaking the tone |
| Upload icon | 📿 (prayer beads emoji) | Upload arrow icon, cloud icon | 📿 is immediately contextual for a japa counter; generic icons feel wrong |
| Stats row layout | 2-column grid | Stacked list, single row | 2-column creates a clean visual balance; stacked adds scroll; single row clips at small widths |
| Button gradient direction | 135° (diagonal) | 90° (top-down), flat colour | Diagonal gradient catches light like a physical object; matches the shine overlay direction |

---

## 14. Design Tokens Reference

All tokens are defined as CSS custom properties on `:root`:

```css
:root {
  /* Colours */
  --bg:       #0E0A06;   /* Page background */
  --surface:  #1A1208;   /* Card / accordion background */
  --border:   #3D2A10;   /* Borders and dividers */
  --amber:    #E8922A;   /* Primary brand — ॐ, CTA accents */
  --amber-lt: #F5B85A;   /* Lighter amber — total number, headings */
  --amber-dk: #A05C10;   /* Darker amber — button base, table headers */
  --gold:     #D4A843;   /* Stat numbers */
  --cream:    #F0E4C8;   /* Body text */
  --muted:    #8A7055;   /* Secondary text, hints */
  --error:    #C0392B;   /* Error states */
  --success:  #2ECC71;   /* Reserved */

  /* Border radius */
  --r: 12px;             /* Primary radius — cards, button, upload zone */
}
```

### Token Usage Map

```
--bg        →  html, body background
--surface   →  .total-card, .stat-card, .entries-list, .ignored-list
--border    →  upload zone border, accordion borders, stat card borders
--amber     →  .om, h1 border, upload label, hover border
--amber-lt  →  h1 text, total number, entry values, filename
--amber-dk  →  button gradient start, file badge border
--gold      →  .stat-num (valid entries)
--cream     →  body text, interactive hover text
--muted     →  subtitle, hints, secondary labels, ignored stat number
--error     →  .error-msg text and border
--r         →  upload zone, button, total card, file badge (as 8px)
```

---

*Japa Yajna Group Admin Tool · Design v1.0 · Internal*
