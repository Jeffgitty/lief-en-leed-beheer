# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **See also:** `../CLAUDE.md` (root) contains additional architecture detail for this project.

## Running

Open `index.html` directly in a browser, or serve locally:

```bash
npx serve .
# or
python -m http.server 8080
```

No build step. All CSS, HTML, and JS live in a single file (~1750 lines): `<style>` block → HTML structure → `<script>` block.

## Key utilities (bottom of `<script>`)

| Function | Purpose |
|---|---|
| `esc(s)` | HTML-escape all user strings before `innerHTML` insertion |
| `fmt(n)` | Format as `€ 1,23` (Dutch locale, non-breaking space) |
| `genId()` | Generate a unique string ID (`Date.now()+random`) |
| `today()` | Return today as `YYYY-MM-DD` |
| `empty(icon, msg)` | Render a centred empty-state placeholder div |
| `GY()` | Read selected year from `#yearSelect` |
| `GYD(y)` | Get-or-create `D.jaren[y]` with empty arrays |
| `save()` | Write `D` to `localStorage` key `liefleed_v1` |

## Income cash-flow timing

`inkKasMaand(i)` determines **when money actually arrives**:
- Regular payment → returns `i.maand`
- Advance payment (`i.vooruit === true`) → returns `i.betaalMaand`

Use `inkKasMaand` for cumulative balance calculations. Use `i.maand` for grouping income by intended month.

Advance payments share a `grp` ID. Deleting one prompts to delete all entries in the group.

## Colleague lifecycle helpers

```
colActiefInMaand(c, yr, m)  → boolean
colActiefInJaar(c, yr)      → boolean
colStartMaandInJaar(c, yr)  → 1-12
colEindMaandInJaar(c, yr)   → 1-12
```

A colleague with no `eindjaar` defaults to year `9999`, month `12` (i.e. no end date). Always use these helpers instead of reading `startjaar`/`eindjaar` directly.

## Month name array

`MN` is 1-indexed: `MN[1] === 'Januari'`, `MN[0] === ''`. Always use `MN[m]` for display; never hardcode Dutch month names.

## Data model addendum

`D` also carries `fondsnaam` and `fondsSubtitel` (editable via the ✏️ button in the header). These default to `'Lief & Leed Beheer'` and the subtitle string respectively when empty.

Budget for a **default** category → `D.catBudgets[naam]`  
Budget for a **custom** category → `category.budget` on the object in `D.categorieen`  
`getCatBudget(naam)` reads either transparently.

## XSS rule (critical)

All user-controlled values into `innerHTML` must go through `esc()`. Never put user data in `onclick="..."` string interpolation; use `data-*` attributes:

```js
// Wrong — breaks on names with quotes
`onclick="fn('${name}')"`

// Correct
`data-naam="${esc(name)}" onclick="fn(this.dataset.naam)"`
```

This includes `JSON.stringify` inside onclick strings (produces double quotes that break attributes).

## Charts

Chart.js 4.4.1 via CDN. Module-level vars: `chartLine`, `chartBar`, `chartPie`. Always call `.destroy()` before creating a new `Chart` on the same canvas element.

## Print

Two print modes controlled by body classes:
- `print-jaar` → shows only `#tab-jaaroverzicht`
- `print-maand` → shows only `#maandPrintArea`

`printJaaroverzicht()` and `printMaandoverzicht()` add/remove these classes around `window.print()`.
