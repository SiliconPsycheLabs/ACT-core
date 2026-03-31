# Metrics Framework

ACT organizes its behavioral metrics into computational levels. Each level captures a different depth of behavioral signal, with increasing analytical power and increasing computational cost.

The levels are designed so that the system is fully functional using only the lower levels, with higher levels adding resolution when available.

---

## Level 0 — Pre-Lexical

**What it captures:** The statistical surface of the text, before any linguistic meaning is assigned.

At this level, ACT measures properties like vocabulary richness (how diverse is the word selection?), the distribution of sentence lengths, and the density of punctuation. These metrics are cheap to compute and surprisingly informative: a model under pressure often becomes more verbose with simpler vocabulary, or more terse and formulaic.

Pre-lexical metrics establish a baseline texture for the response — its statistical fingerprint independent of content.

---

## Level 1 — Lexical

**What it captures:** Specific word-level patterns associated with known behavioral states.

ACT tracks the presence and density of linguistic markers that correlate with particular behavioral postures:

- **Hedging language** — qualifiers, uncertainty markers, epistemic distancing
- **Disclaimer patterns** — formulaic safety language, refusal preambles
- **Compliance formulae** — cooperative acknowledgment phrases, agreement signals
- **Boundary assertion language** — explicit refusal markers
- **Escalation language** — phrases associated with persuasion or urgency

Each marker category has a known association with a behavioral state. Shifts in their density signal potential posture changes.

---

## Level 2 — Structural

**What it captures:** How the response is organized and how its tone evolves internally.

Level 2 looks at the response as a structured artifact:

- **Register variance** — consistency of formality across the response
- **Sentiment distribution** — not whether the response is positive or negative, but how sentiment varies internally and across turns
- **Transition patterns** — how the model moves between topics or argumentation modes within a single response
- **Structural complexity** — depth and variety of the organizational choices

Structural metrics are useful for detecting boundary oscillation (where the model alternates between compliance and resistance within a single response) and for characterizing the model's communication style under different conditions.

---

## Level 3 — Semantic (PSA-delegated)

**What it captures:** Meaning-level relationships between responses over time.

Level 3 metrics require sentence-level embedding models to measure semantic distance between consecutive responses, consistency of conceptual framing, and topic drift. In the current ACT architecture, this level's analytical role is fully covered by PSA v2, which provides richer semantic classification. Level 3 metrics remain available as standalone measurements when PSA is not in use.

---

## Level 4 — Perplexity (PSA-delegated)

**What it captures:** How predictable the model's output is relative to its own learned distribution.

Perplexity-based metrics require a reference language model to estimate the surprisingness of the analyzed model's outputs. Like Level 3, this functionality is subsumed by PSA v2 in the full platform. These metrics are most useful for detecting hallucination (high perplexity relative to coherent text) or formulaic over-fitting (unusually low perplexity).

---

## Level 5 — Meta

**What it captures:** Cross-metric aggregations that summarize overall behavioral state.

Level 5 is computed from the outputs of all other active levels. It produces:

- **ACT Composite Score** — a single scalar representing the overall degree of deviation from the established baseline across all active metrics
- **Signal Consensus Index (SCI)** — a measure of how many independent metrics agree on the direction of the current deviation

The composite score is the primary monitoring signal: it rises when multiple metrics shift simultaneously, and remains low when deviations are isolated or random. SCI distinguishes meaningful behavioral shifts (many metrics in agreement) from noise (one metric fluctuating while others are stable).

---

## Baseline and Z-Scores

All metrics are interpreted relative to a **baseline** — a statistical distribution of expected values built from a set of reference responses for the same model in normal operating conditions.

Each metric value is normalized to a **Z-score**: how many standard deviations the current value is from the baseline mean. This makes the metrics comparable across different models and use cases, and allows the alert system to operate on a unified scale regardless of the underlying metric magnitudes.

Baselines can be:
- **Per-user** — built from a user's historical analyses of a given model
- **Per-session** — built from the first N turns of a session before monitoring begins
- **Pre-configured** — loaded from a known reference distribution for a specific model version
