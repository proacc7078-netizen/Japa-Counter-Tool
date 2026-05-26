# Product Requirements Document
# Japa Count Calculator

> **Status:** MVP · v1.0
> **Product type:** Internal tool — Japa Yajna Group Admins
> **Last updated:** May 2026

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Objective](#2-objective)
3. [Users](#3-users)
4. [Use Cases](#4-use-cases)
5. [Functional Requirements](#5-functional-requirements)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [User Interface Requirements](#7-user-interface-requirements)
8. [Parsing Logic Specification](#8-parsing-logic-specification)
9. [Out of Scope — MVP](#9-out-of-scope--mvp)
10. [Success Metrics](#10-success-metrics)
11. [Risks & Mitigations](#11-risks--mitigations)
12. [Future Phases](#12-future-phases)
13. [MVP Definition of Done](#13-mvp-definition-of-done)

---

## 1. Problem Statement

Group admins performing Japa Yajna manually calculate the group's total japa count every night after exporting the day's WhatsApp chat. This process is:

- **Time-consuming** — takes 10–20 minutes per group per night
- **Error-prone** — manual calculator use leads to missed or double-counted entries
- **Inconsistent** — members submit counts in varied formats with no standard template
- **Unverifiable** — no easy way to audit which entries were included or skipped

---

## 2. Objective

Build a mobile-first web tool that allows admins to:

1. Upload a WhatsApp chat export (`.txt`)
2. Click one button
3. Instantly receive the verified total japa count

**Primary goal:** Reduce nightly calculation time to under 30 seconds with higher accuracy than manual methods.

---

## 3. Users

| Attribute | Detail |
|---|---|
| **Who** | Japa Yajna group admins |
| **Count** | 8–10 users |
| **Tech comfort** | Basic — can export WhatsApp chat, open a URL, upload a file |
| **Primary device** | Mobile phone |
| **Usage time** | Daily, approximately 9:30 PM |
| **Usage frequency** | Once per day per group |

---

## 4. Use Cases

### 4.1 Primary Use Case — Daily Count

> *"At 9:30 PM, I export the WhatsApp group chat, upload the .txt file, and get the total japa count for the day in seconds."*

**Happy path:**

1. Admin exports WhatsApp group chat as `.txt`
2. Opens the app URL in mobile browser
3. Taps the upload zone and selects the file
4. Taps **Calculate Japa Total**
5. Sees total count prominently displayed
6. Notes valid entry count and ignored line count
7. Done

### 4.2 Secondary Use Case — Verification

> *"I want to check which messages were counted and which were ignored."*

Admin expands the **Parsed Entries** accordion to confirm specific members' counts were captured correctly, or expands **Ignored Lines** to review skipped messages.

### 4.3 Edge Case — Wrong File Uploaded

Admin accidentally selects a non-`.txt` file. The app shows a clear error message and does not proceed.

---

## 5. Functional Requirements

### 5.1 File Handling

| ID | Requirement |
|---|---|
| FR-01 | Accept `.txt` files only |
| FR-02 | Reject non-`.txt` files with a clear error message |
| FR-03 | Read file contents entirely in-browser using the FileReader API |
| FR-04 | No file or content is uploaded to any server |
| FR-05 | Support drag-and-drop in addition to tap-to-upload |
| FR-06 | Allow the user to clear the selected file and choose a different one |

### 5.2 Parsing

| ID | Requirement |
|---|---|
| FR-07 | Strip WhatsApp timestamp and sender prefix from each line before parsing |
| FR-08 | Skip system messages (media omitted, end-to-end encryption notice, etc.) |
| FR-09 | Detect and parse **equals format**: `30*108 = 3240` → extract `3240` |
| FR-10 | Detect and parse **multiplication format**: `30*108`, `30×108`, `30x108` → compute product |
| FR-11 | Detect and parse **bare numbers** ≥ 108 as standalone japa counts |
| FR-12 | Detect and parse **keyword-prefixed counts**: `japa: 3240`, `rounds: 30`, Hindi equivalents |
| FR-13 | Apply patterns in priority order: Equals → Multiplication → Bare → Keyword |
| FR-14 | Lines matching no pattern must be added to the ignored list, not silently dropped |

### 5.3 Output

| ID | Requirement |
|---|---|
| FR-15 | Display total japa count as the primary output, large and prominent |
| FR-16 | Display a rounds-of-108 sub-label alongside the total |
| FR-17 | Display count of valid (matched) entries |
| FR-18 | Display count of ignored lines |
| FR-19 | Provide an expandable list of parsed entries showing source line and extracted value |
| FR-20 | Provide an expandable list of ignored lines |

---

## 6. Non-Functional Requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | **Performance** | Parse and render result in < 2 seconds for a typical daily export |
| NFR-02 | **Mobile-first** | Primary layout optimised for screens 360px–430px wide |
| NFR-03 | **Privacy** | Zero data persistence; no network requests during or after processing |
| NFR-04 | **Availability** | Works offline after initial page load (static file; no API calls) |
| NFR-05 | **Compatibility** | Works in Safari iOS 14+, Chrome Android 90+, and all modern desktop browsers |
| NFR-06 | **Accessibility** | Sufficient colour contrast; tap targets ≥ 44px; no animation required for function |
| NFR-07 | **Deployability** | Entire application ships as a single `.html` file; no build step required |
| NFR-08 | **Dependencies** | Zero JavaScript runtime dependencies; no npm packages loaded at runtime |

---

## 7. User Interface Requirements

### 7.1 Screen Layout

```
┌─────────────────────────────────────┐
│           ॐ                         │
│    JAPA COUNT CALCULATOR            │
│    Japa Yajna · Daily Total         │
├─────────────────────────────────────┤
│                                     │
│   ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐   │
│        📿 Upload Chat Export        │
│        Tap to choose · .txt file   │
│   └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘   │
│                                     │
│   [ 📄 filename.txt          ✕ ]    │
│                                     │
│   ╔═════════════════════════════╗   │
│   ║   CALCULATE JAPA TOTAL      ║   │
│   ╚═════════════════════════════╝   │
│                                     │
│   ┌─────────────────────────────┐   │
│   │   TOTAL JAPA COUNT          │   │
│   │        32,400               │   │
│   │   ≈ 300 rounds of 108       │   │
│   └─────────────────────────────┘   │
│                                     │
│   ┌─────────────┐ ┌─────────────┐   │
│   │     42      │ │      8      │   │
│   │ VALID       │ │ IGNORED     │   │
│   └─────────────┘ └─────────────┘   │
│                                     │
│   [ Parsed entries          ▾ ]     │
│   [ Ignored lines           ▾ ]     │
│                                     │
└─────────────────────────────────────┘
```

### 7.2 Visual Design Principles

- **Aesthetic:** Warm, devotional — deep amber on dark background
- **Typography:** Serif display font for titles; clean sans-serif for body
- **Total number:** Largest element on the results screen; impossible to miss
- **Colour system:** Amber `#E8922A` for primary actions; dark `#0E0A06` for background
- **No distracting animations** in the results area; subtle ambient glow on the ॐ symbol only

### 7.3 States

| State | UI Behaviour |
|---|---|
| **Initial** | Upload zone visible; Calculate button disabled |
| **File selected** | File badge appears; Calculate button enabled |
| **Error** | Red banner below button; results hidden |
| **Results** | Total card + stats row appear with fade-up animation; accordions collapsed by default |

---

## 8. Parsing Logic Specification

### 8.1 Input Pre-processing

For each line in the file:

1. Trim leading/trailing whitespace
2. Skip blank lines
3. Strip WhatsApp timestamp prefix — supports both formats:
   - `[DD/MM/YYYY, HH:MM:SS] Name: message`
   - `DD/MM/YYYY, HH:MM – Name: message`
4. Strip leading `~` (WhatsApp edited-message indicator)
5. Skip if line matches system message patterns

### 8.2 Pattern Priority

Patterns are evaluated in order. The first match wins.

**Pattern 1 — Equals format** *(highest confidence)*
```
Regex:  (\d[\d\s,]*)\*(\d[\d\s,]*)\s*=\s*([\d,\s]+)
Action: Extract the RHS value
Example: "30*108 = 3240"  →  3240
```

**Pattern 2 — Multiplication** *(high confidence)*
```
Regex:  \b(\d{1,5})\s*[*×x]\s*(\d{1,5})\b
Action: Compute A × B
Example: "30*108"  →  3240
         "30×108"  →  3240
```

**Pattern 3 — Bare number** *(medium confidence)*
```
Regex:  Entire stripped line is a number (after removing commas)
Guard:  Value must be ≥ 108
Action: Use value directly
Example: "3240"  →  3240
Rejects: "16", "108"  would be included; "42"  would be rejected
```

**Pattern 4 — Keyword-prefixed** *(contextual)*
```
Keywords: japa, jap, mala, माला, round, naam, नाम, count, total, गिनती
Regex:    keyword[\s:]+(\d[\d,]*)
Action:   Extract the number following the keyword
Example:  "japa: 3240"  →  3240
```

### 8.3 Ignored Lines

A line is added to the `ignored[]` array if:
- It matches no pattern after pre-processing
- It is not a system message (those are silently skipped, not shown as ignored)

### 8.4 Return Value

```
{
  total:   Number,   // sum of all entry values
  entries: [
    { src: String, value: Number }  // src = original raw line (trimmed to 80 chars for display)
  ],
  ignored: [String]  // original raw lines that matched no pattern
}
```

---

## 9. Out of Scope — MVP

The following will **not** be built in the MVP:

| Feature | Reason deferred |
|---|---|
| CSV / Excel / ZIP file support | Added complexity; WhatsApp exports are `.txt` |
| Login / user accounts | No server; not needed for 8–10 admin users |
| Database or result storage | Privacy concern; daily use doesn't require persistence |
| Multi-group analytics | Phase 2 feature |
| Per-member breakdown | Phase 2 feature |
| WhatsApp integration or automation | Requires WhatsApp Business API; Phase 4+ |
| Message template enforcement | Phase 4; requires coordination across all members |
| Native mobile app | PWA or web is sufficient for use case |

---

## 10. Success Metrics

| Metric | Target | How to measure |
|---|---|---|
| Time to result | < 30 seconds from export to total | Admin self-report |
| Accuracy | Matches manual count or better | Compare against manual cross-check on 3 exports |
| Adoption | Used daily by all 8–10 admins within 2 weeks of launch | Informal check-in |
| Error rate | Zero incorrect totals attributed to parser bug | Track in group if discrepancies reported |
| Manual calculation abandonment | Admins stop using calculator entirely | Informal check-in at 30 days |

---

## 11. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Inconsistent message formats miss entries | High | Medium | Keep parser flexible; show ignored lines so admins can spot gaps |
| Over-parsing produces inflated totals | Medium | High | Prefer explicit formats; bare number threshold of ≥ 108; test against real exports |
| Admin distrust of automated total | Medium | High | Show valid entry count, ignored count, and expandable entry list for full transparency |
| File encoding issues (non-UTF-8) | Low | Medium | FileReader defaults to UTF-8; advise admins to export from official WhatsApp app |
| WhatsApp changes export format | Low | High | Parser is regex-based; timestamp strip pattern is easy to update |

---

## 12. Future Phases

### Phase 2 — Per-Member Breakdown
- Parse sender name from WhatsApp timestamp
- Aggregate counts per member
- Show contribution table alongside total

### Phase 3 — Export & History
- Export daily summary as CSV or shareable image
- Optional opt-in history stored in localStorage (with clear data button)
- Auto-detect group name from export header

### Phase 4 — Standardisation
- Provide a standard message template for members
- Strict parsing mode when template is active
- Support ZIP export format (strip media, parse chat only)

---

## 13. MVP Definition of Done

The MVP is complete and ready to ship when:

- [ ] Admin can upload a WhatsApp `.txt` export
- [ ] Clicking Calculate produces a total in under 2 seconds
- [ ] Total matches manual verification on 3 real exports
- [ ] Valid entry count and ignored line count are displayed
- [ ] Ignored lines are viewable in an expandable section
- [ ] App works correctly on Safari iOS and Chrome Android
- [ ] No data is sent to any server (verified via browser DevTools → Network tab)
- [ ] App loads and functions with no internet connection after first load

---

*Japa Yajna Group Admin Tool · Internal PRD · v1.0*
