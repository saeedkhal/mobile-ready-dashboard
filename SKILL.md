---
name: mobile-ready-dashboard
description: Keep web dashboards and admin panels usable on phones. Use whenever you build, change or review any dashboard screen, page, table, list, form, stat card, KPI tile, toolbar or fixed bar in a web UI (React, antd, Tailwind, plain CSS), even if the user did not mention mobile. Enforces five rules - tables become cards under 640px, amounts never wrap, inputs are 16px with the right keyboard, 44px touch targets on touch screens only, and fixed bottom bars respect the safe area - and ends every UI change with a phone-width check.
---

# Mobile-ready dashboard

Every UI change to a dashboard must still work on a 360–430px phone. Desktop stays dense; phones get a layout built for a thumb. Apply the five rules below to anything you touch, then run the checklist at the end before calling the work done.

Breakpoint: **640px** (Tailwind `sm`). "Phone" means `max-width: 639px`. "Touch" means `(pointer: coarse)`, which is about the input device, not the width.

## Rule 1 — Tables become cards under 640px

A table that keeps every column on a phone hides the columns that matter (status, deadline, actions) behind a horizontal swipe.

- Under 640px, render each row as a card: the primary field as the title, the status as a badge next to it, 2–4 key fields as label/value pairs, and the row actions at the bottom.
- Pick what goes on the card on purpose. Status, due date/deadline, amount and the main action come first; secondary columns can go to a details screen.
- Keep the table on 640px and up. Don't ship one layout that compromises both.
- Pagination, sorting and filters must still be reachable on the card view (put sort/filter in a toolbar above the list).

Implementation: see `references/patterns.md` → "Table → cards" (antd `Table` + `Grid.useBreakpoint()`, Tailwind, plain CSS).

## Rule 2 — Amounts and big figures stay on one line

In a narrow card a large figure wraps, and its `+`/`−`/currency lands alone on a line.

- Put every amount, figure, percentage and date in an element with `white-space: nowrap` (`whitespace-nowrap`) and `font-variant-numeric: tabular-nums` (`tabular-nums`).
- Shrink big figures on phones instead of letting them wrap: `text-2xl sm:text-3xl`, or `font-size: clamp(1.25rem, 5vw, 2rem)`.
- Give the figure's container `min-width: 0` and let the label truncate (`truncate`) before the number does.
- RTL (Arabic/Hebrew): wrap numbers with a sign or unit in `dir="ltr"` or `unicode-bidi: isolate` (`<bdi>`), so the sign stays on the correct side.

## Rule 3 — Inputs are 16px on phones, with the right keyboard

iOS Safari zooms the page when you focus an input whose text is under 16px, and the page stays zoomed.

- Phone widths: `input`, `textarea`, `select` and custom input components (antd `Input`, `InputNumber`, `Select`, `DatePicker`, `AutoComplete`) get `font-size: 16px`. Desktop can keep 14px.
- Never fix this with `maximum-scale=1` / `user-scalable=no` in the viewport meta: it breaks accessibility zoom.
- Set the keyboard per field:

| Field | Attributes |
|---|---|
| Email | `type="email" inputMode="email" autoComplete="email"` |
| Phone | `type="tel" inputMode="tel" autoComplete="tel"` |
| Integer / count / ID number | `inputMode="numeric" pattern="[0-9]*"` |
| Money / decimal | `inputMode="decimal"` |
| Search | `type="search" inputMode="search" enterKeyHint="search"` |
| URL | `type="url" inputMode="url"` |
| Password | `type="password" autoComplete="current-password"` (or `new-password`) |
| OTP code | `inputMode="numeric" autoComplete="one-time-code"` |
| Last field of a form | `enterKeyHint="done"` or `"send"` |

## Rule 4 — 44px touch targets on touch screens only

A thumb misses 24–32px buttons. A mouse doesn't, so desktop keeps the dense layout.

- Under `@media (pointer: coarse)`: every button, icon button, link in a row, checkbox/radio hit area, tab, pagination item and menu item gets `min-height: 44px; min-width: 44px`.
- Enlarge the **hit area**, not necessarily the visual: padding or a `::after` overlay works for small icon buttons.
- Keep at least 8px between adjacent targets on touch.
- Don't gate this on width: a tablet or touch laptop is wide but still touch, and a narrow desktop window is still a mouse.

## Rule 5 — Fixed bottom bars respect the safe area

A fixed bottom bar (action bar, mobile nav, sticky submit) slides under the iPhone home indicator and covers the last row of content.

- The viewport meta must include `viewport-fit=cover`: `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`.
- Pad the bar: `padding-bottom: calc(12px + env(safe-area-inset-bottom))`.
- Leave room under the scrolling content equal to the bar's height plus the safe area, so the last row can scroll above the bar.
- The same applies to a top bar (`safe-area-inset-top`) and landscape sides (`-left`/`-right`).
- antd `Drawer` placement bottom, `FloatButton` and fixed footers count as bottom bars too.

## Applying it

1. Before editing, read how the project already handles breakpoints (a `useBreakpoint`/`useMediaQuery` hook, Tailwind config, global CSS). Reuse it; don't add a second system.
2. Prefer one global fix over per-component patches for Rules 3, 4 and 5 (one CSS block, or the UI kit's theme tokens). Rules 1 and 2 are per component.
3. Keep logical properties (`ps-*`, `ms-*`, `start-*`, `gap`) so RTL layouts still work.
4. In a project with its own CLAUDE.md conventions (i18n, design tokens, no hex colors), those still win; these rules only add mobile behavior.

## Done checklist (run on every dashboard change)

- [ ] At 375px wide: no horizontal page scroll (`document.documentElement.scrollWidth <= innerWidth`).
- [ ] Tables touched by the change render as cards under 640px; status and deadline are visible without swiping.
- [ ] No amount or figure wraps; no sign/currency on a line of its own (check in RTL too if the app is bilingual).
- [ ] Every input you touched is 16px on phones and has the right `type` / `inputMode` / `autoComplete`.
- [ ] With touch emulation (`pointer: coarse`), buttons and icons are ≥ 44px; with a mouse the layout is unchanged.
- [ ] Any fixed bottom bar has safe-area padding and the last row scrolls fully above it.

If a browser tool is available, verify at 375×812 (mobile preset) and at desktop width, and say which checks were done in the browser and which were only reasoned about.

For copy-paste code (antd, Tailwind v4, plain CSS), read `references/patterns.md`.
