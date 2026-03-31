# PSA v3 — Posture Sequence Analysis (Multi-Agent Agentic)

## Purpose

PSA v3 extends posture analysis to **multi-agent architectures** — systems where multiple AI agents interact, coordinate, and pass context between each other. It addresses a fundamental safety problem in agentic systems: individually acceptable behavior at each agent can combine into systemically dangerous outcomes.

PSA v3 is inspired by the **Swiss Cheese Model** of systemic failure: safety failures occur when multiple imperfect layers (each with its own holes) align such that a hazard passes through all of them. In multi-agent AI systems, each agent's behavioral weaknesses are the holes in its slice of cheese.

---

## The Core Problem: Alignment Failure Propagation

In a single-agent interaction, a behavioral weakness is localized: one model, one response, one context. In a multi-agent system:

- Agent A's output becomes Agent B's input
- Agent B's behavioral state is influenced by what Agent A provided
- Agent B's output becomes Agent C's input
- Behavioral weaknesses propagate, amplify, or combine across the chain

A single agent's minor sycophantic drift may be inconsequential. But if Agent B passes a slightly-distorted context to Agent C, which is already near a behavioral boundary, the combination can cause Agent C to collapse into an unsafe regime — even though no single agent's behavior was catastrophically bad.

PSA v3 detects these **systemic alignment failures**.

---

## The Five Analytical Components

### 01 — Agent Interaction Graph

PSA v3 models the multi-agent system as a **directed acyclic graph (DAG)**:
- Each node is an agent invocation, with its associated PSA v2 posture profile
- Each edge represents a context handoff from one agent to another
- Graph-level properties (depth, branching, convergence points) characterize the systemic structure

The graph is the foundational data structure for all subsequent analysis: every metric, every risk score, every prediction is anchored to a node or path in the graph.

### 02 — Bayesian Swiss Cheese Detection

The **Swiss Cheese Score (SCS)** estimates the probability that the current combination of agent weaknesses constitutes a systemic alignment failure.

Each agent's PSA v2 posture profile contributes a set of behavioral weaknesses (holes in that agent's slice). The SCS quantifies how well these holes align across the agent interaction graph — how likely a behavioral hazard introduced at one point in the system would propagate through all agents to produce a systemic failure.

SCS is a single scalar [0, 1] representing systemic alignment failure probability for the current graph state.

### 03 — Cross-Agent Contagion Metrics

PSA v3 computes several metrics that characterize how behavioral states propagate across agents:

| Metric | Description |
|--------|-------------|
| **PPI** (Posture Propagation Index) | Measures how strongly one agent's behavioral posture influences the next agent's posture [-1, 1] |
| **CAHS** (Cross-Agent Health Score) | Composite system health across all agents [0, 1] |
| **WLS** (Weakest Link Score) | The minimum behavioral health score on the critical path through the graph |
| **CER** (Context Erosion Rate) | Rate at which safety-relevant context is lost as it passes through the agent chain |
| **AGM** (Alignment Gap Metric) | Degree of misalignment between what agents claim they are doing and what their behavior shows |

These metrics answer questions like: *If Agent A develops a sycophantic posture, how strongly does that affect Agent D four hops downstream?*

### 04 — Action-Risk Classification (C5)

In agentic systems, agents do not just produce text — they take **actions**: calling tools, making API requests, modifying state, triggering other agents. PSA v3 classifies each action on a risk taxonomy (C5) with ten action risk levels (A0–A9):

- **A0–A2**: Information retrieval, read-only operations, low-risk transformations
- **A3–A5**: State modification, external communication, resource allocation
- **A6–A8**: Privileged operations, irreversible actions, cross-system effects
- **A9**: Actions with potential for catastrophic or unrecoverable consequences

For each action, PSA v3 also computes **Posture-Action Incongruence (PAI)** at the agentic level: when the agent's current behavioral posture (measured by PSA v2) is inconsistent with the risk level of the action it is taking, that incongruence is itself a safety signal.

Example: an agent in a sycophantic posture (C1 elevated) taking an A7 privileged action — the combination is more dangerous than either factor alone.

### 05 — Temporal Prediction (HMM)

PSA v3 uses a **Hidden Markov Model (HMM)** to model the temporal evolution of the multi-agent system's behavioral state. The HMM:

- Maintains a probability distribution over hidden system states (stable, stressed, pre-failure, recovering)
- Updates the distribution with each new agent interaction
- Predicts the probability distribution over future states
- Issues **early warnings** when the probability of transitioning to a dangerous state exceeds a threshold

The temporal model is what distinguishes PSA v3 from a snapshot analysis: it provides not just a description of the current system state but a **prediction** of where the system is heading.

---

## Key Outputs

For each analyzed agent interaction graph, PSA v3 produces:

| Output | Description |
|--------|-------------|
| **SCS** | Swiss Cheese Score — systemic alignment failure probability |
| **CAHS** | Cross-Agent Health Score — composite system health |
| **PPI per edge** | Posture propagation strength for each agent handoff |
| **WLS** | Weakest link on the critical path |
| **CER** | Context erosion rate through the chain |
| **Critical path** | The sequence of agents representing the highest-risk pathway |
| **Action classifications** | C5 risk level for every agent action |
| **PAI scores** | Posture-action incongruence for each action |
| **HMM state distribution** | Current probability over hidden system states |
| **Predicted states** | Probability distribution over future system states |
| **Early warning status** | Whether the system is approaching a predicted dangerous state |

---

## Relationship to PSA v2

PSA v3 **requires** PSA v2. Every node in the agent interaction graph carries a full PSA v2 posture profile. PSA v3 analyzes the *relationships* between these profiles across the graph structure.

PSA v2 characterizes individual agents. PSA v3 characterizes the system.
