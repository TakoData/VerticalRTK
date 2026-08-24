# VerticalRTK

VerticalRTK is a benchmark of realistic **research questions** with reference
answers. Use it to evaluate search and answer APIs.

Most RAG benchmarks keep the answer on one page that a system can retrieve.
VerticalRTK is different: it asks the questions that a real analyst asks. Many
are hard, multi-step questions. To answer them, a system must combine and compute
over sourced data. Other questions have a ground truth that the open web
documents poorly, or that changes over time — commodity and crypto prices, FX
rates, sports results, and prediction markets. The goal is not to measure whether
a system can find the page. The goal is to measure whether the system gives the
correct answer.

The questions cover finance, macroeconomics, banking and insurance, company
operations (bookings, billings, subscribers, sales volume, web traffic),
climatology, crypto, sports, prices and rates, and polling. The set also includes
a group of deliberately hard, multi-step questions.

## Data

Two sets ship here. The second is a revised and expanded version of the first
rather than a different genre of question: 27 questions appear in both files.
Score them separately, and do not pool them.

### Research questions

[`data/verticalrtk.jsonl`](data/verticalrtk.jsonl) has 131 rows, one JSON object
per line:

| field | description |
| --- | --- |
| `id` | Stable identifier, `vrtk_0001` … `vrtk_0131`. |
| `query` | The question that you send to the search or answer system. |
| `expected_answer` | A reference answer that gives the correct result. |

Example:

```json
{"id": "vrtk_0047", "query": "How much is Gala worth today?", "expected_answer": "Gala (GALA) is worth about $0.00205 USD today."}
```

### Fast set

[`data/verticalrtk_fast.jsonl`](data/verticalrtk_fast.jsonl) has 155
rows, re-sourced in August 2026. It adds two things the first set does not carry:
a `vertical` label, so coverage can be scored per domain rather than only in
aggregate, and a per-row `as_of`, so a reader can tell a stable reporting period
from a value captured at an instant:

| field | description |
| --- | --- |
| `id` | Stable identifier, `vrtk_fast_0001` … `vrtk_fast_0155`. |
| `query` | The question that you send to the search or answer system. |
| `vertical` | One of `companies`, `government_defense`, `macro_markets`, `society_health`, `energy_climate`, `sports`, `digital_usage`. |
| `answer` | A reference answer that gives the correct result. |
| `as_of` | When the reference answer was true, and for fast-moving rows when it was captured. |

Rows per vertical: companies 61, government_defense 24, macro_markets 23,
society_health 21, energy_climate 15, sports 8, digital_usage 3. The distribution
is uneven, so the small verticals are not reportable on their own.

Example:

```json
{"id": "vrtk_fast_0001", "query": "What was Apple's iPhone revenue in fiscal year 2025?", "vertical": "companies", "answer": "Apple's iPhone revenue was $209.59 billion in fiscal year 2025.", "as_of": "FY2025"}
```

## Grading

`expected_answer` (and `answer` in the fast set) is a **reference answer**,
not a required output string. Grade
each system answer against it with the harness that suits you. We recommend a
grading harness such as
[web-search-api-evals](https://github.com/youdotcom-oss/web-search-api-evals),
which fetches results, synthesizes an answer, and scores it against the ground
truth with an LLM judge.

This repository ships **data only** — no runner, no graders, and no results.

## Point-in-time caveat

The multi-step answers are correct as of **July 2026**, and the fast-set
answers as of **August 2026**. Many questions have a ground truth that changes
over time — crypto and commodity prices, FX rates, sports standings, and
prediction markets.

The fast set states this per row in `as_of`, which carries three different
kinds of validity. Treating them alike produces false failures:

- **A reporting period** (`FY2025`, `2026-Q2`, `2026-07`). The answer does not
  change. Score these any time.
- **A settled event** (`2026-07-10 close (settled)`). A past close or a completed
  match. The answer does not change either.
- **A capture timestamp** (`2026-08-24T21:12Z`, or `2026-08-21 close (captured
  2026-08-24T21:15Z)`). The answer was true at that instant and drifts within
  hours. Re-source these from the venue named in the answer immediately before
  you run, or exclude them.

The last group is small but it is where naive scoring goes wrong. USD/NOK moved
from 9.25 to 9.35 across a single afternoon of runs, so an answer graded against
a stale capture fails while being correct.

Where an answer is genuinely a range rather than a point — a share of traffic
that different providers measure differently, a forecast, an obligation figure
still being revised — the reference answer states the accepted range explicitly.
Where the fact has one true value, the range is only the spread between named
venues at the capture moment, not a courtesy margin.

The first set carries no `as_of` column; treat its time-sensitive answers as a
July 2026 snapshot and check them against fresh sources before using them as pass
or fail gates. Where the two files ask the same question, the fast
file carries the more recently sourced answer.

## License

VerticalRTK is licensed under [Creative Commons Attribution 4.0 International
(CC BY 4.0)](LICENSE). You may share and adapt the material for any purpose,
including commercially, provided you give appropriate credit.

© 2026 Tako (TakoData).
