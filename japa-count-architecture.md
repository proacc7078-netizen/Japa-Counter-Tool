# Architecture Document
# Japa Count Calculator

> **Version:** 1.0 · MVP
> **Type:** Client-side static web application
> **Last updated:** May 2026

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architecture Style](#2-architecture-style)
3. [Technology Stack](#3-technology-stack)
4. [Component Breakdown](#4-component-breakdown)
5. [Data Flow](#5-data-flow)
6. [Parsing Engine](#6-parsing-engine)
7. [File Structure](#7-file-structure)
8. [Deployment Architecture](#8-deployment-architecture)
9. [Security & Privacy Model](#9-security--privacy-model)
10. [Performance Characteristics](#10-performance-characteristics)
11. [Error Handling](#11-error-handling)
12. [Browser Compatibility](#12-browser-compatibility)
13. [Known Constraints](#13-known-constraints)
14. [Future Architecture Considerations](#14-future-architecture-considerations)

---

## 1. System Overview

The Japa Count Calculator is a **zero-backend, single-file web application**. It accepts a WhatsApp chat export (`.txt`), parses japa counts from message lines using a regex engine, and renders the total — all inside the user's browser. No data ever leaves the device.

```
┌──────────────────────────────────────────────────────────────┐
│                    Admin's Browser                           │
│                                                              │
│  ┌────────────────┐        ┌───────────────────────────┐    │
│  │   UI Layer     │◄──────►│      Logic Layer          │    │
│  │                │        │                           │    │
│  │  Upload Zone   │        │  FileReader API           │    │
│  │  File Badge    │        │  parseJapaFile()          │    │
│  │  Calc Button   │        │  renderResults()          │    │
│  │  Total Card    │        │  Regex Pattern Engine     │    │
│  │  Stats Row     │        │                           │    │
│  │  Accordions    │        └───────────────────────────┘    │
│  └────────────────┘                                         │
│                                                              │
│         ▲  No network calls. No storage. No backend.        │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Architecture Style

**Pattern:** Single-Page Application (SPA) — fully client-side

| Property | Value |
|---|---|
| Rendering | Browser-native DOM manipulation |
| State management | In-memory JS variables (no framework) |
| Persistence | None |
| Network usage | Google Fonts CDN on first load only |
| Backend | None |
| Build step | None |
| Package manager | None |

The entire application is one `.html` file. Structure, style, and logic are co-located by design — this eliminates deployment complexity and keeps the tool simple enough for any admin to host or share.

---

## 3. Technology Stack

| Layer | Technology | Version | Rationale |
|---|---|---|---|
| Structure | HTML5 | — | Universal browser support; semantic markup |
| Style | CSS3 with custom properties | — | No framework needed; variables ensure design consistency |
| Logic | Vanilla JavaScript (ES6+) | — | No build step; instant load; zero supply-chain risk |
| File I/O | FileReader API | Web API | Reads `.txt` files client-side; no server upload |
| Fonts | Google Fonts (Cinzel, Crimson Pro) | CDN | Devotional aesthetic; graceful fallback to Georgia/serif |
| Hosting | Any static host | — | Single `.html` file; no server-side runtime |

### Why no framework?

React, Vue, or Svelte would add hundreds of kilobytes and require a build pipeline for a tool that has:
- 2 user interactions (upload, click)
- 1 data transformation (parse)
- 1 output screen (results)

Vanilla JS is the correct choice at this scope.

---

## 4. Component Breakdown

### 4.1 UI Components

```
.page
├── .header
│   ├── .om                  — ॐ symbol with ambient glow animation
│   ├── h1                   — "Japa Count Calculator"
│   └── .subtitle            — "Japa Yajna · Daily Total"
│
├── #dropZone                — Upload zone (tap + drag-and-drop)
│   └── input[type=file]     — Hidden; overlaid on the zone
│
├── #fileInfo                — File badge (hidden until file selected)
│   ├── .fname               — Filename display
│   └── #clearFile           — Clear/remove button (✕)
│
├── #calcBtn                 — "Calculate Japa Total" button
│
├── #errorMsg                — Error banner (hidden by default)
│
└── #results                 — Results container (hidden until calculated)
    ├── .total-card
    │   ├── .total-label     — "TOTAL JAPA COUNT"
    │   ├── #totalCount      — Large number display
    │   └── #totalSub        — "≈ N rounds of 108"
    ├── .stats-row
    │   ├── #validCount      — Count of matched entries
    │   └── #ignoredCount    — Count of ignored lines
    ├── #entriesSection      — Accordion: parsed entry log
    └── #ignoredSection      — Accordion: ignored lines log
```

### 4.2 JavaScript Functions

| Function | Responsibility |
|---|---|
| `parseJapaFile(text)` | Core parser. Takes raw file string → returns `{ total, entries[], ignored[] }` |
| `setFile(file)` | Validates extension, stores file reference, updates UI state |
| `clearFile()` | Resets all state: clears file, hides results, disables button |
| `renderResults(total, entries, ignored)` | Injects parsed data into DOM; builds accordion lists |
| `showError(msg)` / `hideError()` | Toggles error banner |
| `toggle(listId, arrowId)` | Handles accordion open/close |
| `formatNum(n)` | Formats numbers with Indian locale (e.g. `32,400`) |
| `esc(s)` | HTML-escapes strings before injecting into innerHTML |
| `trim80(s)` | Truncates display strings to 80 characters |

### 4.3 Event Listeners

| Event | Target | Handler |
|---|---|---|
| `change` | `#fileInput` | `setFile()` |
| `dragover` | `#dropZone` | Prevent default; add drag-over class |
| `dragleave` | `#dropZone` | Remove drag-over class |
| `drop` | `#dropZone` | Extract file from event; call `setFile()` |
| `click` | `#clearFile` | `clearFile()` |
| `click` | `#calcBtn` | Invoke `FileReader`; call `parseJapaFile()` on load |
| `click` | `#entriesToggle` | `toggle('entriesList', 'entriesArrow')` |
| `click` | `#ignoredToggle` | `toggle('ignoredList', 'ignoredArrow')` |

---

## 5. Data Flow

```
Admin device filesystem
        │
        │  .txt file selected via file picker or drag-drop
        ▼
┌───────────────────┐
│   FileReader API  │  readAsText(file, 'UTF-8')
└────────┬──────────┘
         │  raw string
         ▼
┌────────────────────────┐
│   parseJapaFile(text)  │
│                        │
│   1. Split on \n       │
│   2. Strip timestamps  │
│   3. Skip system msgs  │
│   4. Match patterns    │
│      P1: Equals        │
│      P2: Multiply      │
│      P3. Bare number   │
│      P4: Keyword       │
│   5. Accumulate total  │
└────────┬───────────────┘
         │  { total, entries[], ignored[] }
         ▼
┌─────────────────────┐
│  renderResults()    │  DOM injection only — no storage
└────────┬────────────┘
         │
         ▼
    Admin sees result
    on screen
```

At no step is any data written to disk, sent over a network, or stored in browser storage APIs.

---

## 6. Parsing Engine

### 6.1 Pre-processing Pipeline

Each line passes through this pipeline before pattern matching:

```
raw line
  │
  ├─ trim whitespace
  ├─ skip if blank
  ├─ strip WhatsApp timestamp prefix
  │    Format A: [DD/MM/YYYY, HH:MM:SS] Name: ...
  │    Format B: DD/MM/YYYY, HH:MM – Name: ...
  ├─ strip leading ~ (edited message marker)
  ├─ skip system messages
  │    "media omitted", "image omitted", "video omitted",
  │    "audio omitted", "sticker omitted", "document omitted",
  │    "messages and calls", "end-to-end encrypted"
  │
  └─ proceed to pattern matching
```

### 6.2 Pattern Matching — Priority Order

Patterns are evaluated in sequence. The **first match wins**.

#### Pattern 1 — Equals Format *(highest confidence)*

```
Regex:   (\d[\d\s,]*)\*(\d[\d\s,]*)\s*[=]\s*([\d,\s]+)
Action:  Extract RHS value
Example: "30*108 = 3240"  →  3240
Reason:  Explicit final value stated by the member; most trustworthy signal
```

#### Pattern 2 — Multiplication *(high confidence)*

```
Regex:   \b(\d{1,5})\s*[*×x]\s*(\d{1,5})\b   (case-insensitive)
Action:  Compute A × B; both operands must be positive
Example: "30*108"   →  3240
         "30×108"   →  3240
         "30 x 108" →  3240
```

#### Pattern 3 — Bare Number *(medium confidence)*

```
Condition: Entire stripped line is numeric (commas removed)
Guard:     Value must be ≥ 108
Action:    Use value directly
Example:   "3240"  →  3240
Rejects:   "42"    →  ignored  (below threshold)
Rationale: Threshold avoids capturing timestamps, short codes, or unrelated numbers
```

#### Pattern 4 — Keyword-Prefixed *(contextual)*

```
Keywords:  japa, jap, mala, माला, round, naam, नाम, count, total, गिनती
Regex:     keyword[\s:]+(\d[\d,]*)
Action:    Extract the number following the keyword
Example:   "japa: 3240"   →  3240
           "rounds: 30"   →  30
```

### 6.3 Output Contract

```javascript
{
  total:   Number,      // Sum of all matched entry values
  entries: Array<{
    src:   String,      // Original raw line (for display/audit)
    value: Number       // Extracted or computed japa count
  }>,
  ignored: Array<String> // Raw lines that matched no pattern
}
```

### 6.4 What Is Ignored vs Silently Skipped

| Category | Treatment |
|---|---|
| Blank lines | Silently skipped — not shown in ignored list |
| System messages (media omitted, etc.) | Silently skipped — not shown in ignored list |
| Lines matching no pattern | Added to `ignored[]` — shown in accordion for admin review |
| Lines below the bare-number threshold | Added to `ignored[]` |

---

## 7. File Structure

The application ships as a **single file**:

```
japa-count.html
│
├── <head>
│   ├── meta charset, viewport
│   ├── <title>
│   └── Google Fonts link (Cinzel, Crimson Pro)
│
├── <style>          ← all CSS (~300 lines)
│   ├── CSS custom properties (:root)
│   ├── Reset
│   ├── Layout (.page)
│   ├── Header (.om, h1, .subtitle)
│   ├── Upload zone (.upload-zone, .file-selected)
│   ├── Button (.btn-calculate)
│   ├── Results (.total-card, .stats-row)
│   ├── Accordions (.entries-list, .ignored-list)
│   ├── Error state (.error-msg)
│   ├── Footer
│   └── Animations (@keyframes fadeUp, pulse)
│
├── <body>           ← semantic HTML structure
│   └── (see Component Breakdown §4.1)
│
└── <script>         ← all JavaScript (~200 lines)
    ├── parseJapaFile()
    └── UI event handlers + DOM manipulation
```

**Total file size (uncompressed):** ~18–20 KB

---

## 8. Deployment Architecture

### 8.1 Hosting Options

The app is a static file. Any of the following work:

| Option | How to deploy | Cost |
|---|---|---|
| **Netlify Drop** | Drag `.html` to app.netlify.com/drop | Free |
| **GitHub Pages** | Push to repo → enable Pages in settings | Free |
| **Vercel** | Drag-and-drop in dashboard | Free |
| **Local file** | Open `.html` directly in browser (`file://`) | Free |
| **Internal server** | Drop file in any web server's public dir | Varies |

### 8.2 Deployment Diagram

```
Developer
    │
    │  ships japa-count.html
    ▼
Static Host (Netlify / GitHub Pages / Vercel)
    │
    │  serves file over HTTPS
    ▼
Admin's Browser
    │
    │  loads once; works offline thereafter
    ▼
Admin's Device (all processing happens here)
```

### 8.3 CDN Dependency

The only external resource loaded at runtime is the Google Fonts stylesheet:

```
https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;700
                                        &family=Crimson+Pro:ital,wght@...
```

**Fallback:** If Google Fonts is unavailable (offline use), the app falls back to `Georgia, serif` — fully functional with degraded typography only.

---

## 9. Security & Privacy Model

### 9.1 Data Handling

| Data type | What happens to it |
|---|---|
| Uploaded `.txt` file | Read into memory by FileReader; never transmitted |
| Parsed entries and counts | Held in JS variables for the session; cleared on page refresh |
| Ignored lines | Held in JS variables; displayed on screen only |
| Any personal data in chat | Never leaves the browser; never logged |

### 9.2 Network Requests

| Request | When | What |
|---|---|---|
| `fonts.googleapis.com` | Page load | CSS font definitions only |
| `fonts.gstatic.com` | Page load | Font files (woff2) |
| *Everything else* | Never | — |

No request is made during or after file upload or calculation.

### 9.3 Storage APIs

| API | Used? |
|---|---|
| `localStorage` | No |
| `sessionStorage` | No |
| `IndexedDB` | No |
| `Cookies` | No |
| `Cache API` | No |

### 9.4 Threat Model

The application has **no server-side attack surface**. The client-side script execution risk is mitigated by having zero third-party JavaScript dependencies at runtime. The only third-party resource is the Google Fonts CSS link, which loads no executable code.

---

## 10. Performance Characteristics

| Scenario | Expected behaviour |
|---|---|
| Typical daily export (< 1 MB, ~200 lines) | Parse + render in < 100 ms |
| Large export (5 MB, ~2,000 lines) | Parse + render in < 500 ms |
| Very large export (50 MB) | May cause brief UI freeze; not a practical use case |
| First page load (online) | < 1 second including fonts |
| Subsequent loads (fonts cached) | < 200 ms |
| Offline use (after first load) | Fully functional; fonts degrade gracefully |

The parsing loop is O(n) in line count. No sorting, no indexing, no async operations are needed.

---

## 11. Error Handling

| Error condition | Detection | User-facing behaviour |
|---|---|---|
| Wrong file type (not `.txt`) | Extension check in `setFile()` | Red error banner: "Please select a WhatsApp .txt export file." |
| FileReader failure | `reader.onerror` handler | Red error banner: "Could not read the file. Please try again." |
| File with no parseable entries | `total === 0` after parse | Results shown with total `0`; ignored list populated for review |
| Malformed number in match | `isNaN()` guard in parser | Entry silently skipped; line added to ignored list |
| Google Fonts unreachable | Browser fallback | Typography degrades; functionality unaffected |

No errors propagate to the user as uncaught exceptions. All error states are caught and surfaced as UI messages.

---

## 12. Browser Compatibility

| Browser | Minimum version | Notes |
|---|---|---|
| Safari iOS | 14+ | Primary target; FileReader API available |
| Chrome Android | 90+ | Primary target |
| Chrome Desktop | 88+ | Full support |
| Firefox | 85+ | Full support |
| Safari macOS | 14+ | Full support |
| Edge | 88+ | Full support |
| Samsung Internet | 14+ | Full support |

**Required Web APIs:**
- `FileReader` — baseline 2015, universal support
- `CSS custom properties` — baseline 2017, universal support
- `ES6` (const, let, arrow functions, template literals) — universal support
- `Drag and Drop API` — desktop only; mobile falls back gracefully to tap-to-upload

---

## 13. Known Constraints

| Constraint | Detail | Impact |
|---|---|---|
| Single-format input | `.txt` only; no ZIP, CSV, or Excel | Admins must export as plain text (default WhatsApp export) |
| UTF-8 only | `FileReader.readAsText` defaults to UTF-8 | Non-UTF-8 exports (rare) may show garbled characters |
| No persistent history | Results lost on page close | Admin must re-upload to recalculate; by design for privacy |
| First-match-wins parsing | Only one pattern applied per line | A line with two different patterns uses the first detected |
| Bare number threshold | Numbers below 108 are ignored | Members reporting small counts (e.g. single malas) must use multiplication format |
| No per-member breakdown | Sender name is stripped before parsing | All counts pooled into group total only |

---

## 14. Future Architecture Considerations

### Phase 2 — Per-Member Breakdown

Requires extracting sender name from the timestamp prefix before stripping it. No architectural change needed — the parser function would return an additional `byMember: Map<string, number>` field.

### Phase 3 — Export & History

- **Export as CSV/image:** Use `Blob` API + `URL.createObjectURL` — no backend needed
- **Opt-in history:** Use `localStorage` with explicit user consent toggle and a "Clear all data" button

### Phase 4 — Multi-group / PWA

- **Progressive Web App:** Add a `manifest.json` and a Service Worker to make the app installable and fully offline-capable
- **Multi-group support:** URL-based group config (`?group=yajna1`) with localStorage per group key

### If a Backend Becomes Necessary

If future requirements demand server-side processing (e.g. scheduled automation, multi-admin sync), the recommended path is:

```
Static frontend (current)
        │
        │  POST multipart/form-data
        ▼
Serverless function (Vercel / Netlify / Cloudflare Workers)
        │
        │  Same parseJapaFile() logic ported to Node.js
        ▼
Response JSON { total, entries, ignored }
```

The parser is pure — no I/O, no side effects — making it trivially portable to a serverless runtime.

---

*Japa Yajna Group Admin Tool · Architecture v1.0 · Internal*
