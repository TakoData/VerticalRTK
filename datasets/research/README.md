# Research set

The research set holds 131 research questions with reference answers. The
questions are the kind that an analyst asks. Many are multi-step: to answer
them, a system must combine and compute over sourced data. Others have a ground
truth that the open web documents poorly, or that changes over time, such as
commodity and crypto prices, FX rates, sports results, and prediction markets.

The questions cover finance, macroeconomics, banking and insurance, company
operations (bookings, billings, subscribers, sales volume, web traffic),
climatology, crypto, sports, prices and rates, and polling. The set also
includes a group of deliberately hard, multi-step questions.

## Format

[`verticalrtk.jsonl`](verticalrtk.jsonl) has 131 rows, one JSON object per line:

| field | description |
| --- | --- |
| `id` | Stable identifier, `vrtk_0001` … `vrtk_0131`. |
| `query` | The question that you send to the search or answer system. |
| `expected_answer` | A reference answer that gives the correct result. |

Example:

```json
{"id": "vrtk_0047", "query": "How much is Gala worth today?", "expected_answer": "Gala (GALA) is worth about $0.00205 USD today."}
```

## Grading

`expected_answer` is a **reference answer**, not a required output string.
Grade each system answer against it with the harness that suits you. We
recommend a grading harness such as
[web-search-api-evals](https://github.com/youdotcom-oss/web-search-api-evals),
which fetches results, synthesizes an answer, and scores it against the ground
truth with an LLM judge.

This folder ships **data only**: no runner, no graders, and no results.

## Point-in-time caveat

The answers are correct as of **July 2026**. The set carries no `as_of` column,
so treat its time-sensitive answers as a July 2026 snapshot. Before you use them
as pass or fail gates, check them against fresh sources.

## Overlap with the fast set

27 questions also appear in the [fast set](../fast/), one of them reworded.
Where the two sets ask the same question, the fast set carries the more recently
sourced answer. Score the two sets separately, and don't pool their results.
