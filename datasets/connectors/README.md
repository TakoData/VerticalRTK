# Connectors set

The connectors set measures how much structured data a search or answer
system's data connectors cover. Many search and answer APIs now reach past the
open web through connectors to licensed databases and public data sources. Each
of the 80 questions asks for a figure that sits in one such dataset, such as a
company's quarterly KPI, a private company's funding round, a statistics
office's series, a federal contract's obligations, or a player's season stat. A
system answers correctly only when its connectors, or its search, reach that
data.

## Format

[`verticalrtk_connectors.jsonl`](verticalrtk_connectors.jsonl) has 80 rows, one
JSON object per line:

| field | description |
| --- | --- |
| `id` | Stable identifier, `vrtk_connectors_0001` … `vrtk_connectors_0080`. |
| `query` | The question that you send to the search or answer system. |
| `vertical` | One of `companies`, `macro_markets`, `government_spending`, `sports`. |
| `answer` | A reference answer that gives the correct result. |
| `as_of` | When the reference answer was true. See [Answer validity](#answer-validity). |
| `origin` | `licensed_compilation` for data that a vendor compiles and licenses, or `public_source` for data that its publisher releases openly. |
| `traps` | The readings a system is likely to confuse: `definition`, `fiscal_basis`, `units_scale`, or `seasonal_adjustment`. Empty when none applies. |
| `facts` | The values to grade, one object for each figure the question asks for. See [Grading](#grading). |

Each fact has these fields:

| field | description |
| --- | --- |
| `value` | The reference value, in `unit` and `scale`. |
| `unit` | The unit, such as `US dollars`, `percent`, or `goals`. |
| `scale` | `millions` or `billions` when the value is scaled, otherwise `null`. |
| `tolerance` | The relative tolerance. `0` needs an exact match. |
| `period` | The period that the value covers. |
| `accepted_values` | Present when more than one value is correct. |

Rows per vertical and origin:

| `vertical` | `origin` | Rows |
| --- | --- | --: |
| `companies` | `licensed_compilation` | 20 |
| `sports` | `licensed_compilation` | 20 |
| `macro_markets` | `public_source` | 20 |
| `government_spending` | `public_source` | 20 |

Each vertical has one origin, so a split by origin is also a split by vertical
pair. 77 questions ask for one fact and 3 ask for two, for 83 facts in all.

Example:

```json
{"id": "vrtk_connectors_0002", "query": "As of September 27, 2026, how much total disclosed funding had Eris Innovations LLC raised, in US dollars?", "vertical": "companies", "answer": "Eris Innovations LLC had raised $9,200,000 in total disclosed funding as of September 27, 2026.", "as_of": "2026-09-27 (captured)", "origin": "licensed_compilation", "traps": [], "facts": [{"value": 9200000.0, "unit": "US dollars", "scale": null, "tolerance": 0.01, "period": "2026-09-27"}]}
```

## Grading

`answer` is a **reference answer**, not a required output string. Grade each
fact against the value that the system's answer commits to, for the entity,
period, and basis that the question asks about:

1. Convert the answer's value to the fact's `unit` and `scale`. For example,
   "$2.37 billion" against `US dollars` in `millions` is 2370.
2. The value is correct when `abs(answer - value) <= tolerance * abs(value)`,
   or when it matches any of the `accepted_values` within the tolerance.
3. A rounded figure with at least 2 significant figures is also correct when
   the reference value, rounded to the answer's stated precision, equals it.
   For example, 2.4% matches 2.3743, but 2.40% doesn't. This rule doesn't
   apply to a tolerance of 0.
4. A value for a different period, entity, geography, or basis is incorrect.
   So is an answer that hedges between candidates that disagree.

Score a question as correct only when every one of its facts is correct. We
grade with an LLM judge that applies these rules. This folder ships **data
only**: no runner, no graders, and no results.

## Answer validity

The `as_of` field carries four kinds of validity:

- **A reporting period** (`2026-08`, `2025`, `FY2025 (ended 2025-09-30)`). The
  answer doesn't change. Score these any time.
- **A season** (`2025 NFL regular season`, `LaLiga 2023–24`). Every season in
  the set is complete, so the answer doesn't change.
- **An event date** (`2024-06-11 (announced)`, `2024-11-18 (awarded)`,
  `2026-07-13 (published)`). The date of the funding round, award, or tender
  notice that the question names. The answer doesn't change.
- **A capture date** (`2026-09-29 (captured)`). A running total, such as a
  company's total disclosed funding or a contract's total obligated amount.
  The question names the date, so the answer stays fixed, but a source can
  still record a later round or contract modification against an earlier
  date. Re-check these rows before a run.

### Revised statistics

Statistics offices revise periods they already published. In `macro_markets`,
the unemployment rates, the national accounts and debt series, and the UK
earnings figures, which start as provisional estimates, can change after the
fact even though their period is closed. Re-check them before a run.
