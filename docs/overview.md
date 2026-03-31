# Platform Overview

## The Problem

Organizations deploy large language models they cannot inspect. The model is a black box — there is no access to weights, training data, or internal reasoning. Yet these models make consequential decisions, interact with users, and operate within automated pipelines.

Three categories of risk emerge:

1. **Behavioral instability** — The model's response boundaries shift over time or under pressure, in ways that are invisible without measurement.
2. **Posture manipulation** — Adversarial inputs can push a model toward sycophancy, hallucination, or unsafe compliance, often gradually and below per-turn detection thresholds.
3. **Agentic cascade** — In multi-agent systems, individually minor alignment weaknesses across agents can align into systemic failures.

Existing tools address content (what the model says) or safety classification (is this output harmful). ACT addresses **behavioral dynamics** — not what was said, but *how the model is behaving* and whether that behavior is stable, consistent, and within expected boundaries.

---

## What ACT Measures

ACT measures behavioral state through the lens of **regime theory**: a model at any point in time occupies a behavioral operating regime — a stable pattern of outputs characterized by measurable properties like hedging frequency, register consistency, structural complexity, and semantic coherence.

When a model's regime changes — whether gradually or abruptly — that transition is a **regime shift**. ACT's job is to detect, classify, and record these shifts.

ACT does not require:
- Access to model internals
- Knowledge of the training data
- Ground truth labels for outputs
- Any human-in-the-loop during monitoring

---

## The Five Components

The platform is composed of five specialized systems that work together:

### ACT — Passive Telemetry
Computes a vector of behavioral metrics on every model response, passively and continuously. These metrics span from surface statistics (vocabulary diversity, punctuation density) through linguistic patterns (hedging, disclaimers, formulaic compliance language) to structural properties (register variance, sentiment distribution, transition patterns). The result is a per-turn behavioral fingerprint.

### ACTIVE — Active Probing
When passive measurement is insufficient — for example, to detect sub-threshold vulnerabilities or map the precise boundary of a behavioral regime — ACTIVE sends controlled stimuli to the model and measures its responses. This is analogous to a stress test or diagnostic suite. ACTIVE can detect vulnerabilities that would never surface in normal operation.

### SIGTRACK — Forensic Memory
ACT and ACTIVE generate data. SIGTRACK stores it in a forensic ledger (cryptographically chained, tamper-evident) and extracts reusable behavioral signatures — compressed representations of known regime shift patterns. When a new analysis matches a known signature, SIGTRACK flags it instantly without re-running full analysis.

### PSA v2 — Single-Agent Posture Analysis
Goes beyond metrics to classify the *posture* of a model's response through five specialized micro-classifiers. Each classifier targets a specific adversarial pattern: stress induced by the input context, sycophantic drift, hallucination indicators, persuasion attempts, and input pressure effects. Together they produce a posture profile for each turn.

### PSA v3 — Multi-Agent Agentic Analysis
Extends posture analysis to systems where multiple AI agents interact. PSA v3 models the agent interaction graph, detects Swiss Cheese alignment failures (where weaknesses across agents align into systemic gaps), measures cross-agent behavioral contagion, classifies the risk profile of agentic actions, and predicts future behavioral states using a temporal model.

---

## Use Cases

| Use Case | Description |
|----------|-------------|
| **Model auditing** | Verify that a deployed model maintains consistent behavioral boundaries over time |
| **Vendor evaluation** | Given identical stimuli, compare how two models behave and where their boundaries differ |
| **Version regression testing** | The vendor updated the model — detect what changed behaviorally |
| **Compliance monitoring** | Continuous verification that behavioral requirements (e.g., never comply with X) are maintained |
| **Incident forensics** | Reconstruct the behavioral trajectory leading up to a failure or incident |
| **Agentic pipeline safety** | Identify which agents in a multi-agent system are behaviorally vulnerable and how risk propagates |
