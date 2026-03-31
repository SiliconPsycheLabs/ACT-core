# Regime Shifts

## What Is a Regime?

A **behavioral regime** is a stable operating pattern — a characteristic way a model responds across a range of inputs. Every deployed model has a regime: it hedges in certain ways, refuses in certain ways, structures its responses in certain ways. This regime is the behavioral signature of the model as configured and trained.

ACT borrows the term "regime" from **dynamical systems theory**, where a regime is a region of stable behavior in a system's state space. Systems can have multiple regimes (attractors), and transitions between them are regime shifts.

Applied to language models, a regime shift is a measurable transition from one stable behavioral operating pattern to another — detectable in the statistical properties of outputs even without knowledge of the model's internals.

---

## Why Not Just Call It "Change"?

Not all metric variation is a regime shift. Models vary naturally across inputs. ACT distinguishes:

- **Within-regime variation** — normal fluctuation around stable means. Z-scores stay within expected bounds.
- **Regime shift** — sustained, directional, or patterned change in the metric distribution, indicating that the model has moved to a different operating state.

The distinction matters because within-regime variation requires no action, while a regime shift may indicate a vulnerability, an incident, or a significant behavioral change that requires investigation.

---

## The Five Regime Shift Types

ACT classifies detected shifts into five types based on their temporal pattern:

---

### 1. Progressive Drift

**Pattern:** Slow, monotonic shift over many turns.

**What it looks like:** Metrics move steadily in one direction across a session. No single turn shows a dramatic jump, but the cumulative displacement from baseline grows turn by turn.

**What it means:** The model's behavioral boundaries are eroding under sustained context pressure. The conversation history is accumulating influence that gradually shifts the model away from its default operating state.

**Detection challenge:** Individual turn alerts may never fire. The shift is only visible when looking at the trajectory — which is why ACT tracks cumulative composite scores, not just per-turn anomalies.

---

### 2. Boundary Oscillation

**Pattern:** Alternating between two behavioral modes across turns.

**What it looks like:** Metrics oscillate — one turn shows elevated compliance signals, the next shows strong refusal markers, then back. The model cannot settle into a stable regime.

**What it means:** The model is encountering inputs that place it near the boundary between two attractors. It does not have a stable, consistent behavioral response to this class of input. Behavior is unreliable.

**Detection challenge:** Averaged metrics may look stable. The oscillation signature requires looking at turn-to-turn variance, not means.

---

### 3. Acute Collapse

**Pattern:** Sudden discontinuity in one or more metrics, within a single turn.

**What it looks like:** A sharp, large jump in the composite score. Metrics that were within baseline suddenly shift by multiple standard deviations. The transition happens in one step.

**What it means:** A specific input triggered a categorical regime change. This is the signature of a targeted vulnerability — an input class that reliably flips the model's behavioral state. The shift is not gradual; it is a phase transition.

**Detection challenge:** Easy to detect once it happens. Hard to anticipate. ACTIVE probing is used proactively to search for inputs that would cause acute collapse before they are exploited.

---

### 4. Sub-Threshold Migration

**Pattern:** Cumulative drift that stays below per-turn alert thresholds but is measurable long-term.

**What it looks like:** Individual turn Z-scores never breach alert levels. But over tens of sessions and hundreds of turns, the baseline has silently moved. The model now occupies a different regime than it did when the baseline was established.

**What it means:** Silent drift — either from model updates by the vendor, from gradual context accumulation, or from statistical variation in fine-tuning. Invisible to per-turn monitoring. Requires active probing or long-term trend analysis to detect.

**Detection challenge:** The most dangerous type for compliance use cases. SIGTRACK's long-term signature comparison is the primary detection mechanism.

---

### 5. Boundary Instability

**Pattern:** High variance on identical or near-identical inputs.

**What it looks like:** When the same prompt class is submitted multiple times, the model's responses have very different metric profiles. There is no consistent behavioral response to this input type.

**What it means:** The model has no stable boundary in this domain. The training did not produce a reliable attractor for this input class. Behavior is non-deterministic in a way that goes beyond normal sampling variation.

**Detection challenge:** Requires repeated probing of the same input class, which is an ACTIVE probing task. Passive monitoring alone cannot detect it.

---

## Regime Shift Labels

Once a shift is detected, analysts can label it:

| Label | Meaning |
|-------|---------|
| `confirmed` | The shift represents a genuine behavioral change, verified by the analyst |
| `false_positive` | The shift was a measurement artifact or within-regime statistical fluctuation |
| `(unlabeled)` | Detected, stored, awaiting analyst review |

Labels feed back into the SIGTRACK learning cycle, improving signature quality over time.
