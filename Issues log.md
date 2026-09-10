# Issues & Fixes

Significant problems found while building the tool, how each was diagnosed, and how it was fixed.
All were surfaced by a 40-question evaluation set and attributed to a stage: **retrieval** (wrong
sources reached the model) or **generation** (right sources, mishandled).

## Retrieval

- **Primary sources lost to commentary.** Plain-language queries surfaced commentary above the
  formally-worded judgments and treaty text. *Cause:* semantic search matches surface phrasing.
  *Fix:* added a cross-encoder reranker, plus contextual tagging (source identity and per-chunk
  locators woven into the text before embedding).
- **The two-stage "authority-blind" bug (headline).** Asked for the Supreme Court's holding, the
  tool gave a confident answer citing the *High Court's* paragraph as the SC's. *Cause:* both
  retrieval stages were blind to which court a passage came from; fixing the embedding stage alone
  changed nothing, because the reranker reads the bare stored text and still couldn't tell the
  courts apart. *How I found it:* running the embedding stage with the reranker off showed the SC
  chunks near the top, which localised the fault to the reranker. *Fix:* supply court identity to
  the reranker too (in memory, at scoring time only). *Result:* the SC now ranks 1 to 3 and its
  holding is quoted.
- **One long document flooded the results.** A single document took most of the top slots. *Fix:*
  a per-document cap for diversity.

## Generation

- **Internal notes leaked into answers.** Curator "routing notes" sat in the blocks the model
  read, so it quoted them as if they were law. *Fix:* routing notes made internal-only; never
  quoted or cited.
- **Misattributed pinpoint citations.** The tool could give a paragraph drawn from a different
  source than the one asked about. *Fix:* a locator is given only if it appears in that named
  source's retrieved text; otherwise the tool says it couldn't locate it.
- **Paraphrase readable as a quote.** A paraphrase could be lifted as the court's exact words.
  *Fix:* quotation marks reserved for verbatim text only; paraphrase marked as the model's own.
- **Inference stated as fact.** Evaluative conclusions and a source's hedged language were
  sometimes reported as settled. *Fix:* label and hedge inference; never strengthen a source's
  hedge.
- **No corpus/cutoff disclosure.** The tool could present the most recent document it held as the
  final word. *Fix:* for "latest / did it end?" questions, state that the answer reflects only the
  corpus up to a cutoff, and that absence is not evidence of nothing happening.
- **False premises reported as "couldn't find it."** When a question assumed something the sources
  contradicted, the tool reported an inability to locate it. *Fix:* correct the premise directly.
- **Answer length and formatting.** Some answers over-padded or were cut off mid-sentence. *Fix:*
  match length to the question, summarise long answers, raise the token ceiling. (Formatting
  discipline is still imperfect on the smaller model.)

## Pipeline

- Restructured so the corpus loads from a persistent store on reconnect instead of re-embedding
  every session.
- Evaluation harness corrected to call the model once per question, not twice.
- Evaluation questions and their expected-behaviour notes loaded directly from a Google Sheet.

## Evaluation result

- **34/40 met or exceeded (85%):** 24 met, 10 exceeded, 6 fell short.
- Shortfalls split evenly between retrieval and generation, with no dominant failure mode.
- Strongest answers came on the hardest questions (advice-refusal traps, opinion-baits,
  multi-source reasoning).
- Self-evaluated against a pre-defined rubric on the project's own corpus, not an independent
  benchmark.
