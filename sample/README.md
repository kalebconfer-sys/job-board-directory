# jobs-index — sample pack

Five CSVs and a manifest. Everything here is real output from the running
system on the date in `manifest.json`; nothing is mocked, trimmed to flatter,
or hand-corrected.

Read `boards-completeness.csv` first. It is the file the product is about.

---

## What is in the pack

| File | Rows | What it is |
|---|---|---|
| `boards-completeness.csv` | **every board, not a sample** | Per board, per fetch: how many roles came back, what the vendor said it had, and the verdict. |
| `coverage-by-platform.csv` | one per ATS + total | The same verdict rolled up, including the share of boards that are provable at all. |
| `change-events-sample.csv` | ~1,000 | Openings and closures, each carrying a confidence grade **and the written basis for that grade**. |
| `postings-sample.csv` | 1,000 | One posting per company, so the sample reads as breadth rather than one big board. |
| `snapshot-series.csv` | every recorded day | The point-in-time series, including the days it is missing. |

---

## `boards-completeness.csv` — the only column that matters

`verdict` takes exactly four values. They are different records, not shades of
one number:

| verdict | meaning |
|---|---|
| `complete` | The vendor published a total and our fetch matched it. This employer's board is provably whole on this date. |
| `count mismatch` | The vendor published a total and our fetch **did not** match it. Reported, not hidden, not silently retried into agreement. |
| `unverifiable` | The platform publishes no total, so completeness cannot be proven. Never claimed. |
| `unreachable` | The fetch failed. **This is not zero roles.** |

That last distinction is the entire pitch. A company that shows 800 postings
last week and 0 today is one of four things — a real hiring freeze, an ATS
migration, a collector that lost access, or a parse failure — and a posting
count treats all four identically. Only the first is a signal. Here they are
four different rows.

`vendor_publishes_count` tells you, before you look at anything else, whether
a verdict on that board was even possible. Four platforms publish a total:
Greenhouse, SmartRecruiters, Workday, BambooHR. The rest do not, and their
boards are labelled `unverifiable` rather than assumed good.

---

## `change-events-sample.csv` — grades, with reasons

Every row carries `confidence` and `basis`. `basis` is a sentence, not a code,
because the grade is only useful if you can audit why it was assigned.

The sample is **stratified across grades on purpose.** A sample containing only
high-confidence rows would hide the grading, and the grading is the thing being
sold. Expect to see rows we have graded down, including:

- *"no snapshot between X and Y, a N-day gap — the event is real but this date
  is not; it happened somewhere inside that window"*
- *"same company and title is live at another location — this is one requisition
  changing where it is advertised, not a hiring event"*
- *"source platform could not be identified from the posting URL, so
  completeness cannot be assessed"*

Identity note, because it changes the numbers: a posting is keyed on **the
vendor's own posting id**, not on a hash of its title. Before that fix, a role
being retitled was recorded as the old one closing and a new one opening.
Measured 2026-09-03 against a live re-fetch of 25 Greenhouse boards: **30 of
173 high-confidence closures — 17.3% — were still on the board.** One platform,
one date, the high-confidence bucket; not extrapolated to the whole index. It
is exactly the kind of defect that quietly inflates a churn signal.

---

## `snapshot-series.csv` — including the days that are missing

History is appended, never back-filled. A day that was not recorded stays not
recorded, and every event spanning the hole is graded `low` with the gap named
in its basis.

There is a real gap in this series. It is visible in this file rather than
smoothed over, and the events on the following recorded day are graded down
because of it. That is the behaviour to check for in any point-in-time vendor:
what the series does when collection fails.

Two guards produce refusals rather than rows:

- **An empty index is refused.** A zero-role snapshot would fabricate a
  mass-closure event across the whole universe.
- **A day swinging more than ±40% against the previous day is refused.** A
  swing that large is a broken fetch, not a hiring event. Overriding it takes
  an explicit single-use flag with a written reason, which is stored in the
  snapshot row and does not persist to the next day.

---

## Limits of this sample, stated up front

1. **History is short.** Point-in-time recording begins on the first date in
   `snapshot-series.csv`. This is not a multi-year backtest asset, and no
   amount of money buys the missing years. It accrues daily.
2. **Not every board can be proven.** The share that can is in
   `coverage-by-platform.csv`; the rest are labelled `unverifiable`.
3. **Coverage is an order of magnitude below the largest vendors.** The largest
   advertises 175,000+ career sites. This is a depth-of-evidence product, not a
   breadth product — if the requirement is maximum raw volume at unknown
   completeness, the honest answer is that this is not the right dataset.
4. **Company names are not guessed.** A board whose employer name cannot be
   resolved confidently ships under its raw vendor token, and an event that
   cannot be mapped to a board is dropped rather than filed under the wrong
   employer.

## Collection, in one paragraph

Each platform is read through the same public, unauthenticated JSON endpoint
the vendor's own careers widget calls — no logged-in access, no paid API, no
purchased feed, no resold third-party data. Traffic is identified
(`User-Agent: jobs-index/0.1`), never a spoofed or rotating browser agent, and
never routed through proxies. `robots.txt` is checked before a path is adopted,
not after; at least one otherwise-attractive source is excluded on that basis
alone. There is no CAPTCHA-solving capability of any kind.

**The dataset contains no personal data.** Every field describes an employer or
a vacancy: no candidate records, no applicant data, no names, no contact
details, no resumes, no inferred individuals. Nothing is joined to a person.

Full diligence answers: `DATA-PRODUCT-DDQ.md`.

Regenerate this pack with `python make_sample.py`.
