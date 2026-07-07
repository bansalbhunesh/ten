# Wikipedia → Google Sheets — RPA Workflow Flowchart

**Tennr · Implementation Engineer take-home · Bhunesh Bansal · 7 July 2026**

A detailed flowchart for an RPA workflow that extracts **Year, Winner, Score, Runners-up**
from the first 10 rows of Wikipedia's
[List of FIFA World Cup finals](https://en.wikipedia.org/wiki/List_of_FIFA_World_Cup_finals)
table and appends each row to a Google Sheet via the **Google Sheets API
(`spreadsheets.values:append`), configured exactly as a Postman request** — built
**exclusively from the 9 steps in the provided IE Step Library**.

## The deliverable

| File | What it is |
|---|---|
| [`exports/flowchart.png`](exports/flowchart.png) | The flowchart — full-resolution image (primary deliverable) |
| [`exports/flowchart.pdf`](exports/flowchart.pdf) | Same document as a paginated A3 PDF |
| [`flowchart.html`](flowchart.html) | Source of the document — open in any browser |

## Workflow at a glance

```
START
  └─ 1  Open page ······· https://en.wikipedia.org/wiki/List_of_FIFA_World_Cup_finals
  └─ 2  Loop ×10 (index i = 1…10 ↔ table row tbody/tr[i+1] ↔ sheet row i+1)
        ├─ 2.1 ExtractHTML   Year        …/tr[{{index}}+1]/th/a  → "1930"
        ├─ 2.2 Parse number  "1930" → 1930 (Int; doubles as a row-validity guard)
        ├─ 2.3 ExtractHTML   Winner      …/tr[{{index}}+1]/td[1] → "Uruguay"
        ├─ 2.4 ExtractHTML   Score       …/tr[{{index}}+1]/td[2] → "4–2"
        ├─ 2.5 ExtractHTML   Runners-up  …/tr[{{index}}+1]/td[3] → "Argentina"
        ├─ 2.6 Detour        append guard: all fields non-empty?
        └─ 2.7 Call API      POST …/values/Sheet1!A:D:append  (one row per iteration)
  └─ 3  Call API (optional QA) — GET values, expect 11 rows (header + 10)
END — Sheet1!A2:D11 = the 1930–1974 finals
```

## What makes this submission rigorous

- **Everything is verified in a live browser, not assumed.** All four XPaths were evaluated
  in a live Chrome session against every one of the 10 target rows — 40/40 checks returned
  the expected text (7 Jul 2026). The expected-result table and the per-iteration execution
  trace in the flowchart show real measured values, not placeholders.
- **Three silent breakages in the May-2024 recipe are caught and handled.** Since Tennr's
  resource video was recorded: (1) the table prefix drifted from
  `//*[@id="mw-content-text"]/div[1]/table[4]` to `…/div[2]/section[2]/table[3]`
  (section-wrapped markup — the old prefix resolves to nothing); (2) the sortable header no
  longer moves into `<thead>`, so data row *i* is now `tr[i+1]`, not `tr[i]`; (3) the
  country-link path varies row to row (`td[1]/a` on 1930 but `td[1]/span/a` on 1954), so the
  workflow extracts whole cells — the only tails uniform across all 10 rows.
- **Every library annotation is used deliberately** — the 1-based loop index maps 1:1 to
  `tr[i]`, and the loop's *skip-to-next-index-on-error* rule is leaned on as the
  error-handling strategy (per-row appends mean one bad row can never block the other nine).
- **Edge cases are first-class content**: the missing 1942/1946 rows, the 1950 footnote
  marker, en-dash scores vs. `valueInputOption=RAW`, the never-sort-before-extracting trap,
  and non-idempotent re-runs.
- **The unused library steps are accounted for** (Fill text field, Click, Check Box), with
  the reason each one is unnecessary — including why *Click* is deliberately avoided.

## Rebuilding the exports

```sh
chrome --headless --window-size=1160,7000 --force-device-scale-factor=2 \
       --screenshot=exports/flowchart.png flowchart.html
chrome --headless --no-pdf-header-footer --print-to-pdf=exports/flowchart.pdf flowchart.html
```
