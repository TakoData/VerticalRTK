# VerticalRTK

VerticalRTK is Tako's collection of open benchmarks for search and answer
systems. Each benchmark is a set of questions with reference answers. You send
each question to the system under test, then grade the system's answer against
the reference.

For how search and answer APIs scored on VerticalRTK, and the method behind the
benchmark, see the [VerticalRTK benchmark page](https://tako.com/benchmarks/verticalrtk/).

## Why these benchmarks

Existing benchmarks for search APIs ask questions whose answer is easy to
retrieve from the web: one page states it, and a system scores by finding that
page. The questions that analysts ask often have no such page:

- The open web documents the answer poorly, such as a company's segment revenue
  or a federal agency's contract count.
- The answer moves, such as a price, an FX rate, or a sports standing.
- No single page states the answer, so a system must combine several sourced
  figures.

VerticalRTK asks these questions. Many of their answers change over time, so
each dataset states when its answers were true and what to re-check before a
run.

## Datasets

Each dataset has its own folder. The dataset's README describes its format, how
to score it, and what to re-check before a run.

| Dataset | Rows | What it tests | Ground truth last refresh |
| --- | --- | --- | --- |
| [Research](datasets/research/) | 131 | Analyst research questions, including multi-step questions that combine and compute over sourced data. | July 2026 |
| [Fast](datasets/fast/) | 155 | Questions across seven verticals, each with a reference answer that states when it was true. | September 22, 2026 |
| [Connectors](datasets/connectors/) | 80 | Coverage of data connectors: figures that sit in licensed compilations and public data sources, across companies, macro, government spending, and sports. | September 29, 2026 |

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

## Report a wrong answer

Tako builds these benchmarks, and Tako's own API is one of the systems they
evaluate. We publish the questions, the reference answers, and how each answer
was sourced, so that anyone can reproduce a result or dispute one. If you find a
wrong reference answer, [open an issue](https://github.com/TakoData/VerticalRTK/issues).

## License

VerticalRTK is licensed under [Creative Commons Attribution 4.0 International
(CC BY 4.0)](LICENSE). You may share and adapt the material for any purpose,
including commercially, provided you give appropriate credit.

© 2026 Tako (TakoData).
