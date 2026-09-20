# Handoff — Marlowe SMS Campaign Builder (prototype)

Context document for an AI agent or developer picking up `campaign-prototype-en.html`. Read this before editing the file.

---

## 1. What this is

A single-file, clickable **product prototype** of an SMS campaign builder for a small retail store's loyalty club. It is a portfolio piece, not production software. Everything runs client-side against generated mock data. No SMS is ever sent, there is no backend, and there are no network calls apart from a Google Fonts stylesheet.

The design goal is that a non-technical shop owner can send a campaign confidently, in three decisions: **who receives it, what it says, when it goes out** — with cost and constraints visible the whole time.

### History (matters for interpreting the code)

The original was Persian (RTL, Vazir/Yekan Bakh font, Toman currency, Iranian names, Thursday-is-weekend logic). It was then converted to English/LTR, then fully localized to a US retail context. Two consequences to keep in mind:

- The layout is **mirrored from an RTL original**. It reads correctly in LTR, but if you ever restore the Persian version, direction-sensitive CSS must flip again (see §7).
- There may be residual assumptions from the original market. If something looks oddly specific, check it against a US context before preserving it.

---

## 2. File structure

One self-contained `.html` file, roughly 64 KB, in three parts:

| Part | Contents |
|---|---|
| `<head>` | Google Fonts link (Inter + Inter Tight), then all CSS in one `<style>` block |
| `<body>` | Static markup: topbar, three step cards, summary aside, mobile bar, sheet, modal shell, test panel |
| `<script>` | `ICONS` object (inlined Phosphor SVG paths), mock data, state, render functions, event wiring |

There is **no build step and no dependencies**. Open the file in a browser and it runs. Keep it that way — do not introduce npm, a bundler, or a framework unless explicitly asked.

---

## 3. Design system

Tokens live in `:root`. Always use them; do not hardcode hex values or radii in new rules.

**Color**
- Accent `--blue:#2563EB` (hover `--blue-700`, tints `--blue-50` / `--blue-100`)
- Neutrals: `--ink:#0E121B`, `--ink-2:#39404D`, `--muted:#6B7280`, `--muted-2:#98A0AE`
- Lines: `--line:#E3E5EA`, `--line-2:#EFF0F3`; surfaces `--bg:#F7F8FA`, `--card:#FFFFFF`
- Status: `--green`, `--red` + `--red-50/100`, `--amber` + `--amber-50/100`

**Meaning rules** — blue means *selected* or *in progress*, never "primary button". Primary buttons (`.cta`) are near-black `--ink`. Red is blocking, amber is a non-blocking warning. Don't break this mapping.

**Radius** — deliberately tight: `--r-xl:10px` (cards, modals), `--r-lg:8px`, `--r-md:6px` (buttons, fields), `--r-sm:5px` (chips, badges, menu items). No full-capsule pills anywhere. Circles are reserved for the radio dot and the success tick.

**Type** — `--sans` is Inter (body, 15px/1.55, tracking `-.006em`); `--display` is Inter Tight (all headings, the campaign total, the mobile cost figure). Headings use tracking `-.018em`, the page H1 `-.028em`. Micro-labels (section labels, table headers, time groups) are 11px uppercase with `.04em` tracking. All numeric output uses `font-variant-numeric: tabular-nums` so figures don't jitter on recalculation.

**Depth** — hierarchy comes from borders and spacing, not shadows. Shadows appear only on floating layers (menus, modals, test panel).

**Spacing** — 4px grid.

---

## 4. Data and state

### Mock data
`mulberry32(20260909)` is a **seeded** PRNG, so the dataset is identical on every load. Do not replace it with `Math.random()` — screenshots and demos depend on stability. If you change the seed or the `make(...)` calls, every recipient count in this document becomes wrong.

1,200 customers are built by four `make(count, daysMin, daysMax, buysMin, buysMax, consentCount)` calls. Each customer: `{name, days, buys, spend, consent}`. `spend` is USD, generated as `buys × $28–150`.

### Segments (`SEGMENTS`)
Each has `{id, title, desc, rule}` where `rule` is a predicate over a customer. Current opted-in counts:

| id | title | opted-in |
|---|---|---|
| `sleeping` | Lapsed customers | 700 |
| `loyal` | Regulars | 455 |
| `big` | Top spenders (> $800 total) | 177 |
| `new` | One-time buyers | 160 |
| `all` | Everyone in the club | 1,016 |

### State
A single flat `state` object: selected segment, custom-filter mode + filter values, `cap` (trim-to-fit limit), template id, message text, send timing, budget, balance, test mode, sent flag. There is no state library and no diffing — **every change calls `render()`, which repaints everything.** Follow that pattern; it keeps the prototype simple.

### `audience()`
The single source of truth for recipients. Returns `{inGroup, noConsent, eligible, list}`. Consent filtering and the `cap` trim (highest spenders first) both happen here. Never compute recipients anywhere else.

---

## 5. Business rules

- **Cost**: `COST_PER = 0.02` USD per message. Campaign cost = recipients × `COST_PER`.
- **Consent**: customers without `consent` are silently excluded and reported in the audience note. This is a hard rule — never send to them.
- **Opt-out**: `"Reply STOP to unsubscribe"` is appended automatically to every message and shown in the preview. It is not editable.
- **Two separate limits**: *campaign budget* (`state.budget`, $25 default) is the ceiling the user sets for this send; *account balance* (`state.credit`, $40 default) is the credit in their SMS account. They are checked independently and explained in a note in the summary panel — users confuse them, so keep the distinction explicit.
- **Character limit**: `LIMIT = 160` (GSM-7). Over that, the counter warns about multi-part billing. This was 70 in the Persian original (UCS-2) — do not revert it.
- **Trim to fit**: when over budget or balance, the fix sets `state.cap` to how many recipients fit, prioritizing highest spenders. This is always reversible via the warning's "Restore all" action.

### Validation (`problems()`)
Returns an array of `{level, title, body, fix}`. `level: "error"` blocks sending and disables the CTA; `level: "warn"` informs only. Every issue should carry a one-click `fix` where one sensibly exists. Current checks: no recipients, empty message, scheduled time in the past, over budget, insufficient balance, plus a warning when recipients were trimmed.

When adding a check, add it to `problems()` — the CTA state, the issue list, and the footer note all derive from it automatically.

---

## 6. Test states

Reachable from the "Test states" panel in the topbar or via URL: `?test=empty`, `?test=budget`, `?test=credit`. `setMode()` resets the whole state, so these are also the reset path.

| Mode | Setup | Demonstrates |
|---|---|---|
| `normal` | budget $25, balance $40, 700 recipients, cost $14.00 | Happy path |
| `empty` | custom filters: ≤7 days, ≥20 orders, ≥$3,000 | Zero-recipient empty state |
| `budget` | budget $8.00 | Over-budget error, trims to 400 |
| `credit` | balance $8.00 | Insufficient-balance error, trims to 400 |

Keep these working. They are how the prototype is demoed, and they are the only coverage the edge cases have.

---

## 7. Gotchas

- **Money is floating point.** Budget and balance comparisons carry a `+1e-9` epsilon so that trimming to exactly fit doesn't re-trigger the error. Preserve it, or move to integer cents.
- **Fonts load from Google Fonts.** Offline, the page falls back to the system UI stack and still looks fine, but screenshots will differ. If the file must be fully self-contained, the fonts need to be inlined as base64.
- **Icons are inline Phosphor SVG paths** in the `ICONS` object, hydrated via `data-ico` / `data-size` attributes by `hydrateIcons()`. Any HTML injected after render must be followed by a `hydrateIcons()` call, or icons render blank. 20 icons are defined; adding one means pasting its path into `ICONS`.
- **The summary panel is rendered twice** — desktop aside and mobile sheet share `receiptHTML()` and both need `bindReceipt()`. If you add an interactive element there, wire it in `bindReceipt`, not inline.
- **Direction-sensitive CSS** (mirrored from RTL): range-slider gradient direction, the chat bubble's asymmetric corner, receipt scroll thumb side, sheet close-button side, tooltip alignment, table last-column alignment. These are the first things to break if direction changes.
- **`state.sent` freezes the UI.** `render()` returns early once a campaign is confirmed; only "Start a new campaign" clears it.
- **The desktop summary has a custom scrollbar** (native one hidden, `.receipt-thumb` synced on scroll). It needs a `ResizeObserver` and re-syncs on content change.

---

## 8. If you're asked to extend it

Likely next steps, in rough order of value:

1. **Estimated results** — projected opens or redemptions, so cost isn't the only number shown.
2. **Campaign history** — a list of past sends; currently the flow has no memory.
3. **Message length and segment count per recipient** once tags are filled, shown per-customer rather than for the sample customer only.
4. **A/B message split.**
5. **Real Persian/English toggle** rather than two diverged files.

Whatever you add: keep it one file, keep the seeded data, keep the three-step spine, and keep cost and constraints visible at all times. The value of this prototype is that it never lets someone send blind.
