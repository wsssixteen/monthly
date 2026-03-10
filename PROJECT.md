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
- **Persistence:** Manual save via Save button → `localStorage`. Load on `DOMContentLoaded`.
- **Export/Import:** JSON file download/upload; drag-and-drop import supported. Export filename auto-generated as `MONTHLY_DD-MMM-YYYY.json`.
- **Theme:** Dark mode default, light mode toggle (stored in `localStorage` as `monthlyTheme`).

## Key Sections

### Salary Calculator (`salary-grid`)
Three-column layout:
1. **Income** — Gross Salary + optional Other Income inputs.
2. **Salary Deductions** — Auto-computed EPF (11%), SOCSO (~0.5%), EIS (0.2%) + manual Income Tax input.
   - Toggled via an **"Include" checkbox** (top-right of the box). When unchecked, all deductions are zeroed and the box dims.
   - Checkbox state is saved and restored with the rest of the budget data.
3. **Salary display** (`net-box`) — Shows computed take-home.
   - Label reads **NET SALARY** when deductions are on, **SALARY** when off.
   - Subtitle updates accordingly ("Gross + Other − Deductions" vs "Gross + Other").

Core function: `calculateSalary()` — reads gross + other, computes deductions, writes hidden `#net` input, triggers `calculate()`.

### Monthly Commitments (`.planning`)
- Collapsible categories; clicking anywhere on the header toggles expand/collapse.
- Drag-to-reorder categories.
- Each category has a table of items (description + amount).
- **Category rename:** double-click the title to edit inline; Enter/blur to confirm, Escape to cancel. Tooltip: "Double-click to rename".
- Grand Total + Surplus/Deficit displayed in header.
- Delete button uses `stopPropagation` so it doesn't trigger the toggle.

### Uncommitted & Breakdown Tables
- Show how remaining money is allocated after commitments.

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
- Sub-Total column hidden (`display: none` on `nth-child(3)`)
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
  "deductionsEnabled": true,
  "categories": [{ "name": "Housing", "rows": [{ "item": "Rent", "amount": "1200", "notes": "" }] }],
  "planning": [{ "item": "", "amount": "", "notes": "" }],
  "breakdown": [{ "item": "", "amount": "", "notes": "" }]
}
```
Theme is stored separately under key `monthlyTheme` as `"light"` or `"dark"`. Auto-saved on every toggle (no Save button needed). Restored at the top of `loadAuto()` before the budget data load, so it persists across refreshes independently.
