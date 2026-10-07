# Analysis set

The analysis set measures whether a search or answer system can do multi-step
analysis over structured data. Each of the 42 questions needs a series of
sourced values and a computation over them, such as a streak, a ranking, a
ratio, or a count of periods that meet a condition. No web page states any of
the answers, so a system answers correctly only when it retrieves the full
series and computes the result itself.

These are the questions behind Tako's post "Agents Need Structured Data for Data
Work", which races four agents across them.

## Format

[`verticalrtk_analysis.jsonl`](verticalrtk_analysis.jsonl) has 42 rows, one
JSON object per line:

| field | description |
| --- | --- |
| `id` | Stable identifier, `vrtk_analysis_0001` … `vrtk_analysis_0042`. |
| `title` | A short name for the question, as the post's explorer shows it. |
| `query` | The question that you send to the search or answer system. |
| `vertical` | One of `companies`, `economy`, `labor`, `trade`, `public_finance`, `federal_contracts`, `housing`, `crime`, `transportation`. |
| `shape` | The computation the question needs, such as `streak`, `peer_ranking`, `record_gap`, or `ratio_then_scan`. |
| `source` | The data that the reference values were computed from. |
| `captured` | The date that the reference values were read from the source, as `YYYY-MM-DD`. See [Answer validity](#answer-validity). |
| `fields` | The values to grade, one object for each value the question asks for. See [Grading](#grading). |

Each field has these keys:

| key | description |
| --- | --- |
| `name` | The key that the system's answer uses for this value. |
| `type` | `int`, `float`, `str`, `bool`, or `month`. A `month` is a string in the form `YYYY-MM`. |
| `value` | The reference value. |
| `tolerance` | For a `float`, the allowed error: `abs`, `rel`, or both. `null` for every other type. |
| `description` | What the value means and the form it takes, such as the unit or the precision. |

33 questions ask for two values, 6 ask for three, and 3 ask for four, for 96
values in all.

Example:

```json
{"id": "vrtk_analysis_0005", "title": "JetBlue operating margin", "query": "Through the quarter that ended in June 2026, how many quarters in a row has JetBlue's operating margin (operating income over total revenue) been below its level a year earlier, and when did the first of those quarters end? Use its reported quarterly income statements.", "vertical": "companies", "shape": "streak", "source": "Reported quarterly income statements (JetBlue Airways Corporation; operating income and total revenues)", "captured": "2026-10-06", "fields": [{"name": "consecutive_quarters", "type": "int", "value": 5, "tolerance": null, "description": "The count of quarters in the run, ending with the quarter ended June 2026."}, {"name": "run_start_month", "type": "month", "value": "2025-06", "tolerance": null, "description": "The month in which that fiscal quarter ended, as YYYY-MM, such as 2025-06."}]}
```

## Grading

Send the system the `query`, together with each field's `name`, `type`, and
`description`. Ask it to return a JSON object with one key for each field
`name`. Then grade each field against its `value`:

| `type` | Correct when |
| --- | --- |
| `int` | The answer equals the value. |
| `float` | `abs(answer - value) <= tolerance.abs`, or `abs(answer - value) / abs(value) <= tolerance.rel`. |
| `str` | The answer matches the value after trimming whitespace, ignoring case. Names follow the question's spelling. |
| `month` | The answer matches the value after trimming whitespace. |
| `bool` | The answer equals the value. |

A missing field, or a value of the wrong type, is incorrect. Score a question
as correct only when every one of its fields is correct. This folder ships
**data only**: no runner, no graders, and no results.

## Answer validity

Every question names its last period, such as "through the quarter that ended
in June 2026" or "through the May 2026 reading", so its answer doesn't move as
new data arrives. A source can still revise a period that it already
published:

- Statistics offices revise inflation, unemployment, and employment series.
- USAspending records deobligations against earlier fiscal years.
- Redfin revises recent months of sale prices and inventory.
- The Real-Time Crime Index adds counts from agencies that report late.
- Companies restate earlier quarters.

`captured` gives the date that each row's reference values were read. Re-check
the rows that matter to you before a run.
