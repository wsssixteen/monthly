# Monthly — Project Description

## RULES FOR CLAUDE — READ BEFORE EVERY CHANGE

- **ALWAYS confirm before implementing** any feature or layout change.
- **NEVER touch the desktop layout** when the request is mobile-only. All mobile changes go inside `@media (max-width: 600px)` only.
- **NEVER assume a change is global** when the user specifies a view (mobile/desktop). If unsure, ask.
- **ALWAYS ask** if a requested change would require modifying shared HTML structure that affects both views. Propose the approach first, wait for approval.



A single-file Malaysian budgeting app (`index.html`) built for personal use.
All logic, styles, and markup live in one self-contained HTML file with no build step.

## Purpose

Plan spending by allocating money upfront each month, rather than tracking daily transactions after the fact.

## Architecture

- **Single file:** `index.html` — HTML + CSS + JS, no external dependencies, no framework.
- **Persistence:** `localStorage`. **Auto-save** on any change once `appLoaded` is true (`autoSaveNow()` for clicks, `autoSaveDebounced()` ~600ms for typing) — status flashes **"⟳ Auto Saved"**. The manual **Save** button flashes **"✓ Saved"**. Load on `DOMContentLoaded`; `appLoaded` flips true at the end of `loadAuto()` so loading never triggers a save.
- **Export/Import:** JSON file download/upload; drag-and-drop import supported. Export filename auto-generated as `MONTHLY_DD-MMM-YYYY.json`.
- **Theme:** Dark mode default, light mode toggle (stored in `localStorage` as `monthlyTheme`).

## Key Sections

### Salary Calculator (`salary-grid`)
Three-column layout:
1. **Income** — Gross Salary + optional Other Income inputs.
2. **Salary Deductions** — Auto-computed EPF (11%), SOCSO (0.5% flat estimate — PERKESO's real Category-1 table is bracketed, the label drops the "~" for cleanliness), EIS (0.2%), **SKBBK** (PERKESO "LINDUNG 24 Jam", employee-borne — an **override input like PCB** since the scheme became voluntary for local workers per the 8 Jul 2026 Cabinet decision: blank = auto-estimate at the current phase rate, typed value = use that, `0` = opted out. Phase rates are DATE-AWARE via `skbbkRate()` — 0.75% to May 2028, 1.00% Jun 2028–May 2031, 1.25% from Jun 2031 — no manual bump needed; the label (`#skbbkLabel`) always shows the live phase percentage. Estimate is `rate × min(wages, 6000)`; PERKESO's official bracket table rounds slightly differently — reported max RM44.65 vs raw RM45.00 — which is why it stays an overridable estimate), and **PCB** (auto-estimated income tax). SOCSO/EIS/SKBBK apply the statutory RM6,000 wage ceiling; EPF has none. NOTE: the PCB estimate is a rough bracket calc and runs HIGH vs real payslips (real MTD excludes exempt allowances + uses the official cumulative formula) — it stays an editable override for that reason.
   - **PCB auto-estimate** (`computePCB()`): annualise income, subtract personal relief (RM9,000) + EPF relief (capped RM4,000), apply the resident progressive brackets, apply the RM400 rebate (chargeable ≤ RM35,000), divide by 12. A ballpark — the `#tax` field is an **override**: blank = use the estimate (shown as ghost placeholder), typed value = use that. Bracket table sourced from PwC/LHDN (verified 2026-07).
   - **"Not Applicable"**: PCB/MTD is legally mandatory for employers (Income Tax (Deduction from Remuneration) Rules 1994, in force since 1995) — but only bites when the employee is taxable. Single earners below ~RM2,833/mo (after EPF) owe none, and PCB < RM10/mo isn't deducted. When the auto-estimate is 0 (and no manual override typed) the **whole row is swapped** from `PCB: RM [input]` to a `#pcbNA` line reading **"PCB Not Applicable"**; typing an override forces the input row back.
   - The "Estimate — verify via PCB Calculator" hint (`#pcbHint`) sits in the right-column cell under the PCB input; hidden while N/A.
   - Toggled via a bare checkbox (top-right of the box, `title="Include deductions"` — the "Include" word was dropped 2026-07). When unchecked, all deductions are zeroed and the box dims. Accent: green `#2e7d32` (dark) / amber via pre-existing `body.light` rule.
   - Checkbox state is saved and restored with the rest of the budget data.
3. **Salary + Balance display** (`net-box`) — two `.net-half` halves split by a `.net-divider` line. Subtitles (`.net-subtitle`) render as their **own row** (`display:block`) under each label.
   - Left half: computed take-home. Label reads **NET SALARY** when deductions are on, **SALARY** when off; subtitle updates accordingly.
   - Right half: **BALANCE (Salary − Spent)** — salary minus paid rows (full) and ongoing rows (prorated).
   - **Colour scheme is consistent across themes (do not swap):** SALARY = green (`#4caf50` dark / `#3a7a2a` light); BALANCE = amber (`#ffb300` dark / `#b8762a` light); BALANCE `.neg` = red (`#ff5252` dark / `#b02000` light). Shows `RM —` while gross is empty.

Core function: `calculateSalary()` — reads gross + other, computes deductions, writes hidden `#net` input, triggers `calculate()`.

### Monthly Commitments (`.planning`)
- Collapsible categories; clicking anywhere on the header toggles expand/collapse.
- Drag-to-reorder categories.
- Each category has a table of items (description + amount).
- **Category rename:** double-click the title to edit inline; Enter/blur to confirm, Escape to cancel. Tooltip: "Double-click to rename".
- **Category power toggle (power icon, left of `x`):** the icon is an **inline SVG** (line + arc, `stroke="currentColor"`), NOT the `⏻` character — U+23FB is missing from many phone fonts and rendered as a blank box. Keep it SVG; no external icon fonts (app stays dependency-free). `togglePower()` — disables the whole category for the month: dims it (0.45 opacity), excludes it from Grand Total, Surplus AND Balance. Rows stay visible/editable and keep their Sub-Totals. Click again to re-enable. Lit amber while ON, muted grey when OFF. Persisted per category as `"disabled"` (old saves default to enabled).
- **Header button swap on collapse:** open panel shows only `x`; collapsed panel hides `x` and shows `⏻` instead (`.collapsed` class set by `toggleCategory()`; CSS at `.category:not(.collapsed) .power-btn`). Note: to power a category on/off it must be collapsed first.
- **No Save button (2026-07):** everything auto-saves, so the manual Save button was retired (its info-tip removed with it). Bottom-right now holds **Add Category**; the commitments header's top-right holds **`↺`** (`.restart-btn`, Untick All — amber `#ffb300` dark / `#b8762a` light). `manualSave()` remains defined but unreferenced. The "Saved" flash fades in/out (`#saveStatus.show`, 0.35s opacity transition, 1.6s hold).
- **New category starts with one empty row** (`addCategory()` passes a blank row) — next tap is typing, not "Add Row".
- **Hover colours are gated by `@media (hover: hover)`** — phone taps otherwise leave popover buttons "stuck" green until the next tap. Popover direct-child buttons are `width:100%` so Daily/Weekly/Delete match the X-times row width; the mobile delete entry is labelled just "Delete".
- Grand Total + Surplus/Deficit displayed in header.
- Delete button uses `stopPropagation` so it doesn't trigger the toggle.

### Amount Math
Amount fields in all three tables accept `+ - * / ( )` expressions (e.g. `20*30`). `evalAmount()` whitelist-validates then evaluates; the expression stays visible in the field, totals use the result. Invalid/partial input counts as 0.

### Row States (commitment rows) — TWO separate controls with separate jobs
Actions cell is `[✓] [◐] [del]`; the Amount cell holds `[input] [cadence badge]`.
- **`✓` (paid) — NO popup.** Grays the row (0.45 opacity), counts the **full amount** as spent; flips to `↺` to restore. `onPaidBtn()`.
- **`◐` (actions) — CADENCE chooser** (`onOngoingBtn()` → `#cadencePopover`): `Daily` / `Weekly` / `[X] times [set]`. Sets `dataset.cadence` and marks the row ongoing (`popPickCadence()` / `popSetCadenceX()`). The current one is highlighted. **Clicking the already-active cadence again toggles the row back to a plain `active` row** (undo) — Balance immediately drops its amount×times and the Sub-Total reverts to the raw amount.
- **Cadence badge (Amount cell) — TIMES chooser** (`onBadge()` → `#statePopover`): visible only when ongoing, shows the cadence. Opens a 2-row popover: `Current: <t> times | RM <amount×t> (spent)` + `Update: [input] set` (`popSetTimesPaid()` → `dataset.times`). "set" keeps the popover open, refreshes the Current line.
- **Two numbers, `rowInstallments(row)` = N:**
  - **N** (month total, from cadence): `daily` = days in month (28–31), `weekly` = `ceil(days ÷ 7)`, `X` = the custom count.
  - **Sub-Total column + Grand Total** = `amount × N` (the full month plan). Changing times does NOT move them.
  - **Balance** = `amount × timesPaid` (what's actually been spent so far). `timesPaid` is **capped at N** — the Update input's `max` = N, and `capTimesToInstallments()` clamps it when the cadence shrinks.

**↺ Untick All** (left of Save) resets every paid/ongoing row to active — the new-month ritual.

### Promote (Considerations → Commitments)
Considerations rows have an `↑` promote button: one category = moves straight in; multiple = `#promotePopover` lists category targets. Breakdown rows stay del-only.
After a promote, the `#undoToast` appears bottom-center for 5s — **Undo** moves the row back (live values if still present). Guards against accidental promote clicks. **Deleting a commitment row** (`del`) uses the same toast — Undo re-adds the row with its saved state/cadence/times (re-added at the end of its category, not its original slot).

### Uncommitted & Breakdown Tables
- Show how remaining money is allocated after commitments.
- All three inputs (Item/Amount/Notes) auto-save on typing (`autoSaveDebounced`); row del auto-saves immediately. (Fixed 2026-07 — these inputs previously had no save handlers.)

### Fresh-boot rule
`loadAuto()`'s no-save and corrupt-save paths call `finishFreshBoot()` (seed sample category + `appLoaded = true`). Previously they returned early WITHOUT setting `appLoaded`, so on a first visit auto-save silently never activated for the whole session.

## Styling Conventions
- Dark background: `#121212` / `#1b1b1b` / `#1e1e1e`
- Accent green: `#2e7d32` (buttons), `#4caf50` (values/links)
- Danger red: `#b71c1c`
- Font: Segoe UI, 13px base
- Light mode: `body.light` class toggled by `toggleLightMode()`. Container: `#e2d9cd` (darker than body `#f3eee7` for depth). "Include" label, checkbox accent, "Budgeting Workshop" text all have light mode overrides.

## Salary Grid Layout (Desktop + Mobile)
- **Income** — `.income-inputs` CSS grid: `auto 1fr` (label col | input col). "— optional" removed entirely.
- **Salary Deductions** — `.deductions-grid` CSS grid: `1fr 1fr` (EPF/SOCSO left, EIS/Tax right).
- **Tables** — Sub-Total column (`nth-child(3)`) values centred globally.

## Mobile Responsiveness (`@media max-width: 600px`)
- Viewport meta tag added
- `salary-grid` stacks to single column
- Income inputs: fixed `200px` width overridden to `auto` so input fills grid cell
- Tagline splits to 2 lines via `.tagline-break { display: block }`
- **Topbar:** Export/Import/Reset hidden; replaced with `☰` hamburger button. Dropdown opens on click, closes on outside click. JS: `toggleHamburger()`, `closeHamburger()`
- **Commitments header:** Title | Add Category (row 1), Totals full-width (row 2)
- **Totals + Save on same row:** 💤 consideration — requires Option B (JS duplicate elements). Current behaviour kept.
- **Item & Notes column width:** ⏳ reduce input length slightly for all 3 tables
- Sub-Total **and Notes** columns hidden (`display: none` on `nth-child(3)` + `nth-child(4)`) — leaves Item / Amount / Actions so it isn't cramped
- **Commitment-row `del` moves into the ◐ cadence popover** on mobile (`.pop-del`, divider-separated "🗑 Delete row" → `popDeleteRow()`, reuses `deleteRow()` so the undo toast still fires); the row's own del button is hidden (`.category-body table button.danger`). Workshop tables keep their del (only 2 buttons there). Desktop unchanged.
- **Menu button** is `≡`, standard green (amber accent tried 2026-07, reverted). Restart `↺` is grey `#555` dark / standard amber light (amber-dark also tried, too loud).
- **Mobile declutter**: net-box subtitles ("Gross + Other − Deductions" / "Salary − Spent"), the PCB hint line, and all deduction percentages (`span.pct`) are hidden under 600px. `#popCadenceX` is pinned (`flex: 1 1 40px; min-width: 0`) so native number-input widths can't make the times-row wider than the other popover rows.
- **Double-click any input** selects all its text (`document` dblclick → `input.select()`); dragging text inside an input no longer starts a category drag (`enableDrag` cancels `dragstart` when it originates from an input)
- Tables: `min-width: 280px`, horizontal scroll on `.category-body` and `.planning`
- Amount column: `width: 60px`
- `.planning-row` stacks to column

## Quotes

All quotes verified via web search unless marked otherwise. Style: `.quote-line` class — italic, `#555` (dark) / `#7a5c35` (light), plain text only. Considerations and Breakdown sit inline beside the Add Row button.

### In Use

| Location | Quote | Source |
|---|---|---|
| Income | Economy is the art of making the most of life. | George Bernard Shaw, *Man and Superman* — "Maxims for Revolutionists" (1903). Full: *"...The love of economy is the root of all virtue."* |
| Commitments footer | Beware of little expenses; a small leak will sink a great ship. | Benjamin Franklin, *Poor Richard's Almanack* (1745, 1758) |
| Commitment Considerations | Spend not where you may save; spare not where you must spend. | Anonymous folk proverb, collected in George Herbert's *Outlandish Proverbs* (1640) |
| Commitment Breakdown | Sikit-sikit, lama-lama jadi bukit. | Malay peribahasa — "Little by little, over time, it becomes a hill." Indonesian variant: *Sedikit-sedikit, lama-lama menjadi bukit.* |

### Reserve / Considered

- ❝ الاقتصاد في النفقة نصف المعيشة ❞ — Classical Arabic proverb: "Economy in spending is half of livelihood." ⚠️ Weak hadith (da'if) — safe as cultural proverb only.
- ❝ To spend is to choose; to save is to dream of all that is yet to come. ❞ — Original (created)
- ❝ A budget is not a prison — it is the architecture of your freedom. ❞ — Original (created)

### Rejected / Unverified

- "He that is without economy is without anything." — attributed to Oliver Goldsmith. Absent from all known sources.
- "Jimat cermat, kaya menjelang tua." — presented as Malay peribahasa. Not confirmed.

## Data Format (localStorage — key: `budgetData`)
```json
{
  "gross": "5000",
  "other": "",
  "tax": "100",
  "skbbk": "",
  "deductionsEnabled": true,
  "categories": [{ "name": "Housing", "disabled": false, "rows": [{ "item": "Rent", "amount": "1200", "notes": "", "state": "active", "cadence": "", "since": "" }] }],
  "planning": [{ "item": "", "amount": "", "notes": "" }],
  "breakdown": [{ "item": "", "amount": "", "notes": "" }]
}
```
`state`/`cadence` are missing on pre-2026-07 saves — loaders default them to `"active"`/`""`, so old localStorage data and old exported JSON import cleanly.

Theme is stored separately under key `monthlyTheme` as `"light"` or `"dark"`. Auto-saved on every toggle (no Save button needed). Restored at the top of `loadAuto()` before the budget data load, so it persists across refreshes independently.
