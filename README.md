# EU ISDS Research Assistant

A reliability-focused retrieval-augmented generation (RAG) system for **intra-EU and UK
investor-state dispute settlement (ISDS)** law. It answers questions **only** from a curated
corpus of primary and secondary sources, with citations — so a researcher can verify every
claim against a real source rather than fact-checking an AI from scratch.

> **Prototype.** A self-directed project — built to make first-pass ISDS research faster and more
> verifiable, and to understand how retrieval-based AI systems behave and fail from the inside.
> Outputs must be verified against primary sources before being relied on.

---

## The problem

In investor-state dispute settlement, a single dispute usually generates **parallel proceedings
across several forums at once** — arbitral tribunals, national courts in multiple countries, the
EU Commission — and the relevant material ends up scattered across court sites, case-reporting
services, blogs and commentary. Intra-EU and UK ISDS is a particularly dense corner of it, where
EU law, the ICSID system, ECT obligations, third-country enforcement and the UK's post-Brexit
position all interact.

General AI tools are unsafe for this work: their answers aren't tied to identifiable sources, so
everything has to be re-verified from scratch — and hallucinated citations carry real
professional consequences (e.g. *Ayinde v Hackney* [2025] EWHC 1383). The point of this tool is
therefore **verifiability within a focused area**, not breadth.

**Intended users:** LLM/PhD students, paralegals, trainees, junior associates, and NQs.

---

## The design choices that make it what it is

The interesting part of this project isn't the underlying components (those are standard) — it's
the decisions about how to make an AI tool a *lawyer* could actually trust. The core principle
throughout is **reliability over fluency**:

- **Say only what the sources support.** Every factual or doctrinal claim is tied to a numbered
  source. If the retrieved sources don't cover the question, the tool says so and refuses, rather
  than filling the gap from training data.
- **Match confidence to evidence (calibrated register).** Direct quotation for a source's exact
  words; hedged language for synthesis across sources; explicit labelling when something is
  inference; refusal when there's no support. The aim is that the *tone* signals how solid the
  answer actually is — instead of the uniformly confident prose general tools produce whether or
  not they know.
- **Never misattribute.** If asked for a specific paragraph or holding from a named court, the
  tool gives it only if it's actually in that court's retrieved text — otherwise it says it
  couldn't locate it, rather than borrowing a plausible-looking citation from another source.
- **Refuse advice, not just risky topics.** It won't predict outcomes, recommend strategy, or
  apply doctrine to a user's facts — not as a formality, but because it can't do those reliably —
  and it flags confidentiality/privilege risk when a query looks like a live matter.
- **Hold the line in conversation.** Follow-up questions reuse already-retrieved sources, and user
  insistence doesn't change the analysis — the tool won't be argued into an unsupported position.

---

## How it works (briefly)

Retrieval runs in two stages — a broad semantic search that pulls candidate passages by meaning,
then a reranking step that re-reads and re-orders them for genuine relevance — and only then does
the model write an answer, grounded in the top results and cited.

One design detail worth noting, because solving it was the core of the engineering work: source
identity (case name, court, citation, key concepts) is woven into what the search sees, so a
passage can be found by "Supreme Court" or "sunset clause" even when its raw text doesn't contain
those exact words — while the clean original text is what the model actually reads and quotes, so
that added identity never leaks into an answer.

**Stack:** Anthropic Claude (generation), Voyage AI (embeddings + reranking), ChromaDB (local
vector store), Python / Google Colab, Gradio (chat interface). The lighter-weight components were
deliberate, scope-appropriate choices for a focused prototype rather than defaults.

---

## Evaluation

Tested against **40 hand-written questions**, each with a pre-defined expected-behaviour note,
graded strictly (*fell short / met / exceeded*), with every shortfall attributed to either
retrieval or generation.

**Result: 34/40 met or exceeded the standard (85%)** — 24 met, 10 exceeded, 6 fell short. The
shortfalls split evenly between retrieval and generation, with no single dominant failure mode.
The strongest answers came on the *hardest* questions — advice-refusal traps, requests for an
opinion, and multi-source doctrinal reasoning — which is where a reliability-first design most
needs to hold.

A representative finding: the system once attributed a lower court's paragraph to the Supreme
Court — a plausible-but-wrong citation, exactly the failure the tool exists to prevent. Tracing it
showed that *both* retrieval stages were blind to which court a passage came from, and fixing only
the first stage changed nothing because the second still couldn't tell the courts apart. Isolating
the two stages to locate that was the real work.

*Self-evaluated against my own rubric on my own corpus — not an independent benchmark. The value
is as much the method (pre-defined criteria, strict grading, mechanism-level diagnosis of every
failure) as the number.*

---

## Scope

Deliberate design decisions, not shortcomings:

- Built on a **focused corpus of 25 documents**, kept intentionally small to get retrieval
  reliability right before scaling.
- Covers **intra-EU and UK ISDS specifically** — it is not a general arbitration tool.
- It is a **first-pass research aid**, not a replacement for primary research or established
  databases (Jus Mundi, Kluwer Arbitration, Westlaw), which offer far larger corpora and editorial
  functions this doesn't attempt to replicate.

## Limitations

- **Citation accuracy is improved but not guaranteed** — outputs must be verified against primary
  sources before any client-facing or court-bound use.
- **Not for confidential or privileged material** — queries are transmitted to a third-party API,
  which may carry legal-professional-privilege risk.
- **Formatting and length discipline is imperfect** on the smaller generation model used here.

---

## Roadmap

- Tune the output register to read like a concise legal research note.
- Label claims drawn from secondary commentary explicitly, and point to the primary decision.
- Add keyword search alongside the semantic search to better catch exact citations and section
  numbers.
- Return structured multi-forum "saga" timelines alongside the underlying sources.
- Expand the corpus once retrieval precision is locked in.

---

## Note on the corpus

The corpus itself is **not included** in this repository — it contains third-party copyrighted
material curated for research use. Only the system's code and evaluation methodology are published
here.
