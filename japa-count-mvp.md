# MVP Document
# Japa Count Calculator

> **Version:** 1.0
> **Status:** Ready to ship
> **Last updated:** May 2026

---

## Table of Contents

1. [What Is the MVP](#1-what-is-the-mvp)
2. [Problem Being Solved](#2-problem-being-solved)
3. [Who It Is For](#3-who-it-is-for)
4. [The One Core Job](#4-the-one-core-job)
5. [MVP Scope](#5-mvp-scope)
6. [Feature List](#6-feature-list)
7. [What Is Explicitly Not Built](#7-what-is-explicitly-not-built)
8. [Definition of Done](#8-definition-of-done)
9. [Acceptance Criteria](#9-acceptance-criteria)
10. [Test Plan](#10-test-plan)
11. [Launch Checklist](#12-launch-checklist)
12. [Success Metrics](#12-success-metrics)
13. [Known Limitations](#13-known-limitations)
14. [After MVP — What Comes Next](#14-after-mvp--what-comes-next)

---

## 1. What Is the MVP

A **single HTML file** that an admin opens in their phone browser, uploads a WhatsApp chat export, taps one button, and instantly sees the group's total japa count for the day.

Nothing more. Nothing less.

The entire product ships as `japa-count.html` — no installation, no login, no backend, no app store.

---

## 2. Problem Being Solved

Every night at around 9:30 PM, Japa Yajna group admins:

1. Export the WhatsApp group chat as a `.txt` file
2. Manually scroll through hundreds of messages
3. Identify lines that contain japa counts
4. Add them up on a calculator
5. Post the total to the group

This takes **10–20 minutes** and produces **errors** — missed entries, double counts, wrong sums.

The MVP eliminates steps 2–4 entirely. The admin exports, uploads, and reads the total. Done in under 30 seconds.

---

## 3. Who It Is For

| Attribute | Detail |
|---|---|
| **Users** | Japa Yajna group admins |
| **Count** | 8–10 people |
| **Device** | Mobile phone (primary) |
| **Tech level** | Basic — can export a WhatsApp chat and open a URL |
| **Usage pattern** | Once daily, ~9:30 PM |
| **Trust requirement** | High — they post the total publicly; it must be right |

---

## 4. The One Core Job

> **Upload a WhatsApp .txt export. Get the correct total japa count.**

Every feature in the MVP exists to serve this job. If a feature does not serve this job, it is not in the MVP.

---

## 5. MVP Scope

### In Scope

```
✓  Accept .txt file (tap to upload or drag and drop)
✓  Parse japa counts from WhatsApp message lines
✓  Display total count
✓  Display number of valid entries parsed
✓  Display number of ignored lines
✓  Show expandable list of parsed entries (for verification)
✓  Show expandable list of ignored lines (for transparency)
✓  Clear error message on wrong file type
✓  Mobile-first layout
✓  Works in any browser, no installation
✓  No data leaves the device
```

### Out of Scope

```
✗  CSV / Excel / ZIP support
✗  Login or user accounts
✗  Database or history storage
✗  Per-member breakdown
✗  Multi-group support
✗  WhatsApp integration or automation
✗  Native mobile app
✗  Push notifications
✗  Admin dashboard
```

---

## 6. Feature List

### F1 — File Upload

| Property | Detail |
|---|---|
| **What** | Admin selects a WhatsApp `.txt` export from their device |
| **How** | Tap the upload zone → native file picker opens |
| **Also** | Drag and drop supported on desktop |
| **Validation** | Only `.txt` files accepted; others trigger an error message |
| **After selection** | Filename badge appears; Calculate button enables |
| **Reset** | Clear button (✕) removes the file and resets the form |

### F2 — Japa Count Parser

| Property | Detail |
|---|---|
| **What** | Scans every line of the chat for japa count entries |
| **Input** | Raw `.txt` file content as UTF-8 string |
| **Pre-processing** | Strips WhatsApp timestamps, sender names, system messages |
| **Pattern 1** | Equals format — `30*108 = 3240` → extracts `3240` |
| **Pattern 2** | Multiplication — `30*108`, `30×108`, `30x108` → computes product |
| **Pattern 3** | Bare number ≥ 108 — `3240` → uses directly |
| **Pattern 4** | Keyword-prefixed — `japa: 3240`, `rounds: 30` → extracts number |
| **Priority** | Patterns evaluated in order above; first match wins |
| **Output** | `{ total, entries[], ignored[] }` |

### F3 — Total Display

| Property | Detail |
|---|---|
| **What** | Shows the summed japa count as the primary screen element |
| **Format** | Indian locale number formatting — `32,400` not `32400` |
| **Sub-label** | Rounds of 108 — `≈ 300 rounds of 108` |
| **Size** | Largest text on screen — impossible to miss |
| **Position** | Top of results area; visible without scrolling |

### F4 — Summary Stats

| Property | Detail |
|---|---|
| **What** | Two counters below the total |
| **Valid entries** | Count of lines that matched a pattern and were summed |
| **Ignored lines** | Count of lines that matched no pattern |
| **Purpose** | Quick confidence signal — admin sees how many entries were captured |

### F5 — Parsed Entries Accordion

| Property | Detail |
|---|---|
| **What** | Expandable list of every line that was parsed and its extracted value |
| **Default** | Collapsed |
| **Content** | Source line (truncated to 80 chars) + parsed value, right-aligned |
| **Purpose** | Lets admin verify specific members' counts were captured correctly |

### F6 — Ignored Lines Accordion

| Property | Detail |
|---|---|
| **What** | Expandable list of every line that was not parsed |
| **Default** | Collapsed |
| **Content** | Raw line text (truncated to 80 chars) |
| **Purpose** | Transparency — admin can spot missed entries and follow up |
| **Note** | System messages (media omitted, etc.) are silently excluded — not shown here |

### F7 — Error Handling

| Scenario | Message shown |
|---|---|
| Non-`.txt` file selected | "Please select a WhatsApp .txt export file." |
| FileReader API failure | "Could not read the file. Please try again." |
| File with zero parseable entries | Total shows `0`; ignored list populated |

---

## 7. What Is Explicitly Not Built

These are not deferred — they are **consciously excluded** from the MVP because they add complexity without serving the core job today.

| Not built | Why |
|---|---|
| **Per-member breakdown** | Admins only need the group total right now. Adding names requires sender parsing and a list UI that increases screen complexity. Phase 2. |
| **History / storage** | The admin posts the total to the group and is done. No one needs to retrieve yesterday's count from the app. Privacy is also a concern with chat data. |
| **CSV or Excel support** | WhatsApp exports are `.txt`. No admin has asked for other formats. |
| **Login / accounts** | 8–10 admins sharing a URL is sufficient. Auth adds friction and infrastructure. |
| **Multi-group analytics** | Each admin runs the tool for their own group. Aggregation is a future admin need. |
| **Native app** | A URL shared in WhatsApp is faster to adopt than an App Store install for 8–10 people. |
| **WhatsApp API integration** | Requires WhatsApp Business API approval, webhook infrastructure, and ongoing maintenance. Solves a Phase 4 problem. |

---

## 8. Definition of Done

The MVP is **done** when all of the following are true:

- [ ] Admin can open the app on a mobile browser without installing anything
- [ ] Admin can upload a WhatsApp `.txt` export by tapping the upload zone
- [ ] Tapping "Calculate Japa Total" produces a result in under 2 seconds
- [ ] The total displayed matches manual verification on 3 separate real exports
- [ ] Valid entry count and ignored line count are displayed
- [ ] Parsed entries list is viewable (expandable)
- [ ] Ignored lines list is viewable (expandable)
- [ ] Wrong file type shows a clear error message
- [ ] The app works on Safari iOS 14+ and Chrome Android 90+
- [ ] No data is sent to any server (verified via browser DevTools → Network tab)
- [ ] The app loads and functions with no internet connection after initial load (fonts degrade; function intact)
- [ ] All 8–10 admins can use it without assistance

---

## 9. Acceptance Criteria

### AC-01 — File Upload

```
Given the app is open
When the admin taps the upload zone
Then the native file picker opens

Given a .txt file is selected
Then a file badge appears showing the filename
And the Calculate button becomes enabled

Given a non-.txt file is selected
Then an error message appears
And the Calculate button remains disabled
```

### AC-02 — Calculation

```
Given a valid .txt file is selected
When the admin taps "Calculate Japa Total"
Then the result appears in under 2 seconds

Given a file containing "30*108 = 3240" in a message
Then that line contributes 3240 to the total

Given a file containing "30*108" in a message
Then that line contributes 3240 to the total

Given a file containing "3240" as a standalone number in a message
Then that line contributes 3240 to the total

Given a file containing "japa: 3240" in a message
Then that line contributes 3240 to the total

Given a file containing only text messages with no numbers
Then the total is 0
And all message lines appear in the ignored list
```

### AC-03 — Results Display

```
Given the calculation completes
Then the total is displayed as the largest text on screen
And the total uses Indian number formatting (e.g. 32,400)
And a rounds-of-108 sub-label is shown below the total
And a valid entry count is shown
And an ignored line count is shown
```

### AC-04 — Accordions

```
Given the results are displayed
Then the Parsed Entries accordion is collapsed by default
And the Ignored Lines accordion is collapsed by default

When the admin taps "Parsed entries"
Then the list expands showing each matched line and its value

When the admin taps "Ignored lines"
Then the list expands showing each unmatched line
```

### AC-05 — Reset

```
Given a file is selected
When the admin taps the clear button (✕)
Then the file badge disappears
And the Calculate button becomes disabled
And any previous results are hidden
```

### AC-06 — Privacy

```
Given any file is uploaded and calculated
When the browser's Network tab is observed
Then zero requests are made to any external URL during or after calculation
```

---

## 10. Test Plan

### 10.1 Test Exports Required

Before shipping, test against **3 real WhatsApp exports** from actual Japa Yajna groups:

| Export | What to verify |
|---|---|
| Export A — typical day | Total matches manual count; valid entry count is correct |
| Export B — mixed formats | Lines with `*`, `×`, bare numbers, and keywords all parsed correctly |
| Export C — messy day | Non-japa messages (greetings, media, reactions) appear in ignored list only |

### 10.2 Manual Test Cases

| # | Test | Expected result |
|---|---|---|
| T01 | Upload Export A; tap Calculate | Total matches manual calculation |
| T02 | Upload Export B; check entries accordion | All format variants shown with correct values |
| T03 | Upload Export C; check ignored accordion | Greetings, stickers, reactions in ignored list |
| T04 | Upload a `.jpg` file | Error message shown; button stays disabled |
| T05 | Upload a `.csv` file | Error message shown; button stays disabled |
| T06 | Tap ✕ after selecting a file | File badge gone; button disabled; no results |
| T07 | Upload file; calculate; upload new file | Previous results hidden when new file selected |
| T08 | Open app in Safari iOS (iPhone) | Layout correct; tap targets work; result readable |
| T09 | Open app in Chrome Android | Layout correct; tap targets work; result readable |
| T10 | Open DevTools → Network → upload and calculate | Zero network requests observed during parse |
| T11 | Turn off Wi-Fi after page load; calculate | App works fully; fonts may degrade |
| T12 | File with only `108` on a line | Should be included (value equals threshold) |
| T13 | File with only `42` on a line | Should be ignored (below threshold) |
| T14 | Line: `30*108 = 3240` | Should extract `3240` (not compute `3240` again) |
| T15 | Very long filename | File badge truncates gracefully; no overflow |

### 10.3 Cross-browser Test Matrix

| Browser | OS | Version | Pass? |
|---|---|---|---|
| Safari | iOS | 16+ | — |
| Chrome | Android | 110+ | — |
| Chrome | macOS | Latest | — |
| Safari | macOS | 16+ | — |
| Firefox | macOS/Windows | Latest | — |
| Edge | Windows | Latest | — |

---

## 11. Launch Checklist

### Pre-launch

- [ ] Run full test plan against 3 real exports
- [ ] Verify zero network requests during calculation (DevTools)
- [ ] Test on at least one iPhone and one Android device
- [ ] Confirm all 8–10 admins have the URL
- [ ] Send a 2-line usage guide to admins:
  - "Export your WhatsApp group chat as .txt"
  - "Open [URL], upload the file, tap Calculate"

### Deployment

- [ ] Upload `japa-count.html` to chosen static host (Netlify / GitHub Pages / Vercel)
- [ ] Confirm HTTPS is enabled (required for file picker on mobile browsers)
- [ ] Confirm the URL is short and shareable (use a custom domain or short link if needed)
- [ ] Share URL in the admin WhatsApp group

### Post-launch (first week)

- [ ] Check in with admins after day 1 — did it work?
- [ ] Collect any formats that were missed (ignored but should have been parsed)
- [ ] Note any usability friction
- [ ] Decide if a parser update is needed before day 7

---

## 12. Success Metrics

### Primary Metric

> **Time to result < 30 seconds** from export to total posted in the group

Measured by: admin self-report in the first week.

### Supporting Metrics

| Metric | Target | Timeframe |
|---|---|---|
| Accuracy | Total matches manual check on all 3 test exports | Before launch |
| Adoption | All 8–10 admins using it daily | By day 7 |
| Manual calculation abandonment | Zero admins still using calculator manually | By day 14 |
| Error rate | Zero reports of wrong totals | Ongoing |
| Time saved per admin per night | 10+ minutes | By day 30 |

### What Success Looks Like at 30 Days

- Admins open the URL from a WhatsApp bookmark
- Upload, calculate, post total — without thinking about it
- Nobody has opened a calculator for japa counts in weeks

---

## 13. Known Limitations

These are accepted for the MVP. They are documented so admins know what to expect and so the team knows what to watch.

| Limitation | Impact | What admin should do |
|---|---|---|
| Members with unusual formats may be missed | Their count goes into ignored list | Admin can check ignored list and add manually if needed |
| Bare numbers below 108 are not parsed | Single-mala reports may be missed | Members should use `1*108` or `japa: 108` format |
| No per-member view | Admin cannot see who contributed what | Phase 2 feature |
| No history | Previous days' totals not stored | Admin keeps their own record (WhatsApp post) |
| Offline use degrades fonts | Typography falls back to Georgia/serif | Functionality is not affected |
| Non-UTF-8 exports may garble characters | Very rare; only affects old Android exports | Use official WhatsApp app to export |

---

## 14. After MVP — What Comes Next

The MVP ships when it reliably produces the correct total. Once adoption is confirmed and the tool is trusted, the following phases become relevant:

### Phase 2 — Per-Member Breakdown
Parse the sender name alongside the count. Show a table of who contributed how much. Useful for group accountability and recognition.

### Phase 3 — Export & History
Let admins download the daily summary as a CSV or image. Optionally store the last N days' totals locally (with a clear data option).

### Phase 4 — Standardisation
Provide a standard message template for members to copy. Tighter parsing when the template is active. Support ZIP export format.

### Phase 5 — Automation
Scheduled parsing via WhatsApp Business API or a shared group email. Remove the manual export step entirely.

---

> The MVP is done when it works for 3 real exports. Ship it then. Improve after.

---

*Japa Yajna Group Admin Tool · MVP v1.0 · Internal*
