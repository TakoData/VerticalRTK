# VerticalRTK

VerticalRTK is Tako's collection of open benchmarks for search and answer
systems. Each benchmark is a set of questions with reference answers. You send
each question to the system under test, then grade the system's answer against
the reference.

## Why these benchmarks

Most benchmarks for search APIs score retrieval: whether the system finds a page
that contains the answer. That measures an index and a retriever. It doesn't
measure what a person or an agent gets back from the system, which is an answer.

The gap is widest for the questions that analysts ask. Many of those questions
have one of these properties:

- The open web documents the answer poorly, if at all. Examples are a company's
  segment revenue, a federal agency's contract count, and a city's climate
  normals.
- The ground truth moves. Examples are commodity and crypto prices, FX rates,
  sports standings, and prediction markets.
- No single page states the answer, so a system must combine and compute over
  several sourced figures.

A benchmark for these questions has to grade the answer, not the page. It also
has to date its ground truth. A reference answer that was true last week can
fail a system that's correct today. Each dataset here states when its answers
were true and what to re-check before a run.

Tako builds these benchmarks, and Tako's own API is one of the systems they
evaluate. We publish the questions, the reference answers, and how each answer
was sourced, so that anyone can reproduce a result or dispute one. If you find a
wrong reference answer, [open an issue](https://github.com/TakoData/VerticalRTK/issues).

## Datasets

Each dataset has its own folder. The dataset's README describes its format, how
to score it, and what to re-check before a run.

| Dataset | Rows | What it tests | Ground truth dated |
| --- | --- | --- | --- |
| [Research](datasets/research/) | 131 | Analyst research questions, including multi-step questions that combine and compute over sourced data. | July 2026 |
| [Fast](datasets/fast/) | 155 | Questions across seven verticals, each with a reference answer that states when it was true. | Per row; moving rows re-sourced September 21, 2026 |

The research and fast sets share 27 questions. Score each set separately, and
don't pool their results.

## Pin a commit

Reference answers change when we re-source a row or a source revises a figure.
Scores are comparable only when they come from the same commit. To pin a
dataset, fetch it at a commit SHA instead of `main`:

```
https://raw.githubusercontent.com/TakoData/VerticalRTK/<commit-sha>/datasets/fast/verticalrtk_fast.jsonl
```

Report the commit SHA with your results. Commits before the per-dataset folders
keep both files in `data/`: `data/verticalrtk.jsonl` and
`data/verticalrtk_fast.jsonl`.

## License

VerticalRTK is licensed under [Creative Commons Attribution 4.0 International
(CC BY 4.0)](LICENSE). You may share and adapt the material for any purpose,
including commercially, provided you give appropriate credit.

© 2026 Tako (TakoData).
