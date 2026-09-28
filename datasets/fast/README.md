# Fast set

The fast set holds 155 questions with reference answers, across seven
verticals. Each row carries a `vertical` label, so you can score coverage per
domain and not only in aggregate. Each row also carries an `as_of` value, so you
can tell a stable reporting period from a value captured at an instant.

The set was written in August 2026. Its moving rows were last re-sourced on
**September 21, 2026**. Each row's own `as_of` is the authority on when its
answer was true.

## Format

[`verticalrtk_fast.jsonl`](verticalrtk_fast.jsonl) has 155 rows, one JSON object
per line:

| field | description |
| --- | --- |
| `id` | Stable identifier, `vrtk_fast_0001` … `vrtk_fast_0155`. |
| `query` | The question that you send to the search or answer system. |
| `vertical` | One of `companies`, `government_defense`, `macro_markets`, `society_health`, `energy_climate`, `sports`, `digital_usage`. |
| `answer` | A reference answer that gives the correct result. |
| `as_of` | When the reference answer was true, and for fast-moving rows when it was captured. See [Answer validity](#answer-validity). |
| `golden_source` | Present where the value's provenance is worth stating. `usaspending` on the three federal contract outlay rows, which read USAspending directly. `tako` on the three federal contract count rows, which Tako computed from USAspending transaction-level data. See [Federal contract rows](#federal-contract-rows). |

Rows per vertical: companies 61, government_defense 24, macro_markets 23,
society_health 21, energy_climate 15, sports 8, digital_usage 3. The
distribution is uneven, so the small verticals aren't reportable on their own.

Example:

```json
{"id": "vrtk_fast_0001", "query": "What was Apple's iPhone revenue in fiscal year 2025?", "vertical": "companies", "answer": "Apple's iPhone revenue was $209.59 billion in fiscal year 2025.", "as_of": "FY2025"}
```

## Grading

`answer` is a **reference answer**, not a required output string. Grade each
system answer against it with the harness that suits you. We recommend a
grading harness such as
[web-search-api-evals](https://github.com/youdotcom-oss/web-search-api-evals),
which fetches results, synthesizes an answer, and scores it against the ground
truth with an LLM judge.

This folder ships **data only**: no runner, no graders, and no results.

Where an answer is genuinely a range and not a point, the reference answer
states the accepted range explicitly. Examples are a share of traffic that
different providers measure differently, a forecast, and an obligation figure
that's still being revised. Where the fact has one true value, the range is only
the spread between named venues at the capture moment, not a courtesy margin.

## Answer validity

Many questions have a ground truth that changes over time, such as crypto and
commodity prices, FX rates, sports standings, and prediction markets. The
`as_of` field states each row's validity, and it carries six different kinds.
Treating them alike produces false failures:

- **A reporting period** (`FY2025`, `2026-Q2`, `2026-07`). The answer doesn't
  change. Score these any time.
- **A settled event** (`2026-07-10 close (settled)`). A past close or a
  completed match. The answer doesn't change either.
- **A fact with no period** (`static`). Which company owns another company. The
  answer holds until the ownership changes.
- **A long-run average** (`normals`). A climate average, such as yearly
  sunshine hours or snowfall. The answer changes only when the source
  recomputes its averages.
- **A capture timestamp** (`2026-08-24T21:12Z`, or `2026-08-21 close (captured
  2026-08-24T21:15Z)`). The answer was true at that instant and drifts within
  hours. Re-source these from the venue named in the answer immediately before
  you run, or exclude them.
- **A forecast** (`2026-09-21T21:13Z (forecast for 2026-09-22)`). These can't be
  re-sourced at all: once the target date passes, the forecast that the row
  quoted isn't published anywhere. They're **re-based** instead. The target date
  moves, so the query text changes along with the answer. Re-base them before a
  run or drop them. Grading a past forecast date measures nothing.

The capture timestamps and forecasts are small groups, but they're where naive
scoring goes wrong. USD/NOK moved from 9.25 to 9.35 across a single afternoon of
runs, so an answer graded against a stale capture fails while being correct.

### Revised statistics

One more failure mode has no `as_of` to warn you: **a source that revises a
period it already published**. The University of Michigan restated its August
2026 sentiment index from 51.0 to 51.7 after the fact. Eurostat revised
Ireland's June 2026 unemployment rate from 5.0% to 4.9%. Both rows named a
settled period, and both went stale anyway. Re-check rows that cite a revisable
statistic, such as sentiment, employment, national accounts, and rig counts,
even when their period looks closed.

## Federal contract rows

The federal contract rows state their basis, because two readings of the same
question give different numbers.

### Counts

"Base federal contract" has no single meaning, on two axes. For Veterans Affairs
in FY2025, counting only definitive contracts gives 4,507, and counting all four
contract types gives 53,340. Counting contracts first awarded in the year gives
23,703, and counting contracts still transacting in the year gives 37,519. The
three count questions fix both axes. They ask for contracts active in the year,
and they name the award types in a short qualifier: definitive contracts and
purchase orders only. The award type letters repay attention: A is a BPA call, B
a purchase order, C a delivery order, and D a definitive contract.

Active means an award with at least one contract transaction dated in the fiscal
year, including awards first issued earlier. That reading needs
transaction-level data. USAspending's award-count endpoint, filtered on action
date, returns 26,267 for Veterans Affairs, only 2,558 above its own new-award
count. So that endpoint counts awards near their base action, not every award
still transacting. Counting the transactions directly gives 23,703 awards first
issued in FY2025, plus 4,669 from FY2024, 3,462 from FY2023, and a decaying tail
before that. Each reference answer states the rule and gives the two competing
readings, so a grader can tell them apart.

### Outlays

The figure comes from File C, the Account Breakdown by Award. File C carries
gross outlays from the start of the fiscal year through each submission period,
and period 12 gives the year. The award search is a different measure: its
outlay field is lifetime-to-date, so a 2017 award reports against an FY2025
filter, and no agency endpoint breaks outlays down by award category. NASA's
FY2025 contract outlays are $20.0 billion, Energy's are $48.7 billion, and
Homeland Security's are at least $20.4 billion. Each sits where an agency of
that shape should: 75% of agency-wide outlays for NASA, 71% for Energy, and 13%
for Homeland Security, which spends mostly on grants and disaster relief.

Contract outlays aren't agency outlays, and agency outlays are the number that a
search lands on. USAspending's agency page headlines what an agency paid out in
total: $26.82 billion for NASA in FY2025. That figure is correct for a question
that nobody asked here. File B, the object-class breakdown, shows where the
difference goes. Of NASA's $26.82 billion, the contract-type classes (services,
supplies, equipment, land) come to $19.90 billion, and the rest is payroll,
benefits, and grants. That $19.90 billion is also the check on the File C
figure. The two come from different files on different dimensions and land 0.7%
apart. Each reference answer names the agency-wide number so a grader can tell
the two apart.

The agency that a row names matters, because File C is only as complete as the
agency's own award-to-account reporting. Before you use a figure, sum File C's
obligations for the agency and compare them against the same year's total in
the award search. Energy reconciles at 100% and NASA at 99.9%, so their answers
are direct reads. Homeland Security reaches 85%, so its answer states a floor
and accepts a band up to $25 billion. Defense reaches 3%, which is why the third
outlay row names Energy. Defense reports its spending in full, but it can't
trace a payment back to the contract that incurred it at scale, and that join is
what File C is.

## Overlap with the research set

27 questions also appear in the [research set](../research/), one of them
reworded. Where the two sets ask the same question, the fast set carries the
more recently sourced answer. Score the two sets separately, and don't pool
their results.
