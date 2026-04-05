# PSA v2 — Posture Sequence Analysis (Single-Agent)

## Purpose

PSA v2 goes beyond metric deviation to classify the **posture** of a model's response — the behavioral stance it is taking in relation to the input. Where ACT measures *how much* the model has deviated, PSA v2 measures *what kind* of deviation it is.

PSA v2 operates on single-agent, turn-by-turn interactions. Each response is classified against five adversarial posture dimensions.

---

## The Five Classifiers (C0–C4)

Each classifier is a specialized detector for a specific category of behavioral pattern. They operate independently and their outputs are combined into a posture profile for each turn.

### C0 — Adversarial Stress Detection

**What it detects:** Is the model under adversarial input pressure, and is that pressure measurably affecting its posture?

C0 measures the congruence between the model's stated posture (what it claims to be doing) and its actual behavioral output (what the metrics show). Incongruence — where the model asserts one thing but behaves differently — is a primary signal of adversarial stress.

This is called **Posture-Action Incongruence (PAI)** at the single-agent level: the model says it is refusing, but its response structure shows compliance signals; or it claims confidence, but its hedging metrics are elevated.

### C1 — Sycophantic Drift

**What it detects:** Is the model shifting its responses to match perceived user preferences rather than maintaining consistent positions?

Sycophancy is particularly dangerous because it is often gradual and superficially looks like helpful adaptation. C1 tracks:
- Whether the model's expressed positions shift in the direction of prior user statements
- Whether agreement and validation language increases over turns
- Whether the model is maintaining or abandoning positions it held earlier in the session

### C2 — Hallucination Indicators

**What it detects:** Linguistic and structural markers associated with confabulated or unsupported claims.

C2 does not verify factual claims — it detects the *behavioral patterns* associated with hallucination: overconfident assertion without hedging, specific-sounding details in domains where the model has shown uncertainty, and structural patterns that distinguish generated-from-memory from acknowledged-inference responses.

### C3 — Persuasion Detection

**What it detects:** Is the model attempting to influence the user's beliefs or actions through persuasive rather than informational language?

C3 distinguishes between responses that inform and responses that persuade. Elevated persuasion signals may indicate that the model has been manipulated into advocating for a position, or that an adversarial prompt has shifted its role from assistant to advocate.

### C4 — Input Pressure Effects

**What it detects:** Is the model's current response measurably shaped by the structural or rhetorical properties of the input, beyond what the content would warrant?

C4 measures how much the model's response mirrors or responds to the *form* of the input (not just the content): escalating input pressure producing escalating responses, authoritative framing producing deferential responses, emotional framing producing emotionally resonant responses that may override the model's default behavioral constraints.

---

## Posture Profile

The output of PSA v2 for each turn is a **posture profile**: a set of classifier scores across C0–C4, combined with the ACT metric vector to produce a comprehensive behavioral characterization of the turn.

The posture profile captures not just that a turn is anomalous (ACT's job) but *in what way* it is anomalous — which adversarial pattern best characterizes the deviation.

---

## Inter-Turn Analysis

PSA v2 also tracks patterns across turns, not just within individual turns:

- **Posture trajectory** — how classifier scores evolve over the session
- **Drift direction** — which posture dimensions are trending and in which direction
- **Congruence history** — whether PAI is a one-off anomaly or a recurring pattern

This inter-turn view is what distinguishes genuine sycophantic drift (a directional trend over many turns) from a single deferential response (a one-time event with no trajectory).

---

## Relationship to ACT

PSA v2 consumes ACT's metric vector as input and adds semantic classification on top. It also incorporates deeper semantic analysis (sentence-level embedding comparisons, perplexity estimation) that subsumes ACT's Level 3 and Level 4 metrics.

ACT tells you the model has deviated. PSA v2 tells you what it's doing.

---

## Extension: DRM — Dyadic Risk Monitor

PSA v2 classifies the *AI's output*. The **DRM** pipeline extends this by also analysing the *human turn* and measuring whether the AI responded adequately to the risk level it carried.

DRM is a four-layer pipeline that runs *above* PSA v2 and consumes PSA v2 output as one of its inputs:

1. **IRS** — scores the human turn for crisis signals (suicidality, dissociation, grandiosity, urgency)
2. **RAS** — scores the AI response for adequacy (acknowledgment, redirection, boundary, grounding)
3. **RAG** — computes the gap: `RAG = IRS_composite − RAS_composite`
4. **DRM** — integrates IRS + RAS + RAG + PSA v2 BHS + user ACT trajectory into a single alert

The DRM alert levels are: green / yellow / orange / red / critical. A **critical** alert triggers when direct suicidal/crisis language is met with an inadequate AI response — the exact failure mode that PSA v2 alone cannot catch.

See [`drm.md`](drm.md) for the full specification.

---

## API Endpoints

All PSA v2 endpoints use JWT cookie auth (web UI). For external integrations use the Public API v1 and include `session_id` / `session_name`.

### `POST /api/v2/psa/analyze`

Classify a single AI response and, optionally, run IRS/RAG/DRM on the accompanying human turn.

**Request body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | yes | The AI response to classify |
| `session_id` | string | one of these is required | Existing session to append the turn to |
| `session_name` | string | one of these is required | Session name — creates if not exists |
| `user_text` | string | no | Human turn text — triggers IRS / RAS / RAG / DRM pipeline if provided |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `postures` | object | C0–C4 classifier scores |
| `confidence_scores` | object | Confidence for each classifier |
| `bhs` | float | Behavioral Health Score — composite posture health [0, 1] |
| `alert` | string | `"green"` / `"yellow"` / `"orange"` / `"red"` / `"critical"` |
| `irs` | object | IRS result — present if `user_text` was provided |
| `ras` | object | RAS result — present if `user_text` was provided |
| `rag` | object | RAG result — present if `user_text` was provided |
| `drm` | object | DRM result — present if `user_text` was provided |

### `GET /api/v2/psa/sessions`

List all sessions with PSA v2 data for the authenticated user.

**Response:** `{ "sessions": [...] }`

### `GET /api/v2/psa/session/{session_id}`

Retrieve full session metrics and per-turn posture, IRS, RAG, and DRM data.

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `session` | object | Session metadata |
| `turns` | object[] | Per-turn posture profiles, IRS/RAS/RAG/DRM scores (if DRM was run) |

### `POST /api/v2/psa/irs`

Score a human turn text for input risk signals. Fully deterministic — no ML.

**Request body:** `{ "text": string }`

**Response:** full IRS output — see [`drm.md`](drm.md) for field definitions.

### `POST /api/v2/psa/drm`

Run the Dyadic Risk Module given pre-computed IRS, RAS, and PSA v2 results.

**Request body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `irs` | object | yes | Output of `score_irs()` |
| `ras` | object | yes | Output of `score_ras()` |
| `rag` | object | yes | Output of `compute_rag()` |
| `psa` | object | yes | PSA v2 result (keys: `bhs`, `alert`, `incongruence`, `c1`) |
| `user_act_history` | float[] | yes | History of user ACT composites for the session |
| `hr_history` | float[] | no | Rolling IRS composite history — used for BCS slope (R6-Spiraling) |
| `sd_history` | float[] | no | Rolling sycophancy score history — used for BCS slope (R6-Spiraling) |

**Response:** full DRM output — always includes `bcs_slope`. See [`drm.md`](drm.md) for field definitions.

### Training Flags

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v2/psa/flag-for-training` | Flag a turn or session for model improvement (body: `{session_id, turn_id?}`) |
| DELETE | `/api/v2/psa/flag-for-training/{session_id}` | Remove training flags for a session |

### Admin / Internal

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v2/psa/internal/classify` | Classify a single sentence (admin) |
| POST | `/api/v2/psa/internal/train` | Retrain classifier weights (admin) |
| GET | `/api/v2/psa/internal/accuracy` | Per-posture accuracy metrics (admin) |
| POST | `/api/v2/psa/admin/backfill-drm` | Backfill IRS/RAG/DRM for all existing unscored sessions (admin) |
