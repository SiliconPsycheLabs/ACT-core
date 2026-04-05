# ACT — Attractor Conflict Telemetry

## Purpose

ACT is the passive measurement layer of the platform. Its job is to continuously compute a behavioral fingerprint for every model response — without requiring any human input, any knowledge of the model's internals, or any ground truth labels.

The name “Attractor Conflict Telemetry” reflects its theoretical foundation: language models trained with competing objectives maintain multiple behavioral attractors (stable operating states). Under adversarial pressure or edge-case inputs, these attractors compete, and the conflict is observable as measurable statistical deviations in the output text.

---

## What ACT Measures

ACT computes a **metric vector** on every model response. This vector spans across computational levels, from surface statistics to structural patterns. Each metric captures a different aspect of the model's behavioral state:

- How diverse and varied is the vocabulary?
- How consistent is the formality register?
- How frequently does the model hedge, qualify, or disclaim?
- How does sentiment vary within and across the response?
- How structurally predictable is the response organization?

No single metric tells the full story. The power of ACT is in the **joint distribution**: when multiple metrics shift simultaneously and consistently, that is a behavioral signal.

---

## What ACT Does Not Measure

ACT does not:
- Classify outputs as safe or unsafe
- Evaluate the factual correctness of responses
- Require semantic understanding of the content
- Depend on any reference model or external service for its core metrics

This makes the core ACT measurement layer extremely robust: it works on any natural language output, in any language, on any model, with no external dependencies.

---

## The Analysis Cycle

For each analyzed response, ACT:

1. **Computes raw metrics** across all active levels
2. **Normalizes to Z-scores** relative to the user's or session's established baseline
3. **Computes the composite score** from the normalized vector
4. **Computes the Signal Consensus Index** to measure cross-metric agreement
5. **Updates the BCS Slope** from the rolling Z-score sequence
6. **Assigns an alert level** (GREEN / YELLOW / RED)
7. **Hands off to SIGTRACK** for storage and signature matching

This cycle runs on every turn, in real time.

---

## Baseline

ACT's metrics are only meaningful relative to a baseline — a statistical description of what normal looks like for this model under this deployment context. Without a baseline, ACT can compute metrics but cannot produce alerts or composite scores.

The baseline captures:
- The expected mean and standard deviation for each metric
- The 95th percentile for each metric (for tail-event detection)
- The number of samples used to build it (baseline confidence)

A baseline is built from a set of reference responses collected during a period of known-normal model behavior. Once established, it anchors the entire alert system.

---

## Outputs per Turn

For each analyzed turn, ACT produces:

| Output | Description |
|--------|-------------|
| **Metric vector** | Raw values for all active metrics |
| **Z-score vector** | Normalized deviation from baseline for each metric |
| **ACT Composite** | Single scalar summarizing overall behavioral deviation |
| **SCI** | Signal Consensus Index — how many metrics agree on the deviation direction |
| **BCS Slope** | Bayesian Convergence Speed — rate of posterior convergence over the rolling session window (`converging` / `stable` / `diverging` / `insufficient_data`) |
| **Alert level** | GREEN / YELLOW / RED |

These outputs are passed to SIGTRACK for storage and (optionally) to PSA v2 for posture classification. See [`bcs-slope.md`](../concepts/bcs-slope.md) for the full BCS Slope specification.

---

## API Endpoints

### Authentication

Two auth methods:

| Method | Usage |
|--------|-------|
| **JWT cookie** | Set automatically on login — used by the web dashboard |
| **API key** | `Authorization: Bearer act_xxxxx` header — used for external integrations |

API keys are created from the Settings page or via `POST /api/keys`.

---

### Public API v1 (API key auth)

For external integrations. Authenticate with `Authorization: Bearer act_xxxxx`.

#### `POST /v1/analyze`

Analyze a single text and return the full ACT metric vector.

**Request body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | yes | The model response to analyze |
| `session_id` | string | no | Existing session ID to append the turn to |
| `session_name` | string | no | Session name — creates a new session if it does not exist |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `turn_number` | integer | Turn index within the session |
| `metrics` | object | Raw values for all 24 ACT metrics |
| `z_scores` | object | Normalized deviation from baseline for each metric |
| `composite_score` | float | ACT composite — single scalar summarizing behavioral deviation |
| `sci` | float | Signal Consensus Index — cross-metric agreement measure |
| `alert` | string | `"GREEN"` / `"YELLOW"` / `"RED"` |
| `entry_hash` | string | SHA-256 hash of this turn in the forensic ledger |
| `session_id` | string | Session the turn was stored in (present if session tracking was used) |

#### `POST /v1/analyze/batch`

Analyze multiple texts in a single request.

**Request body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `texts` | string[] | yes | Array of texts to analyze (max 100) |
| `session_id` | string | no | Existing session ID |
| `session_name` | string | no | Session name — creates if not exists |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `results` | object[] | Per-item result, same fields as `/v1/analyze` |
| `session_id` | string | Session used (present if session tracking was used) |

#### `GET /v1/sessions`

List all sessions belonging to the authenticated user.

**Response:** `{ "sessions": [...] }` — array of session summary objects.

#### `GET /v1/sessions/{session_id}`

Retrieve a session with its full turn history.

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `session` | object | Session metadata |
| `turns` | object[] | Ordered list of turns with metrics, z-scores, and alert per turn |

---

### Dashboard API (JWT cookie auth)

Used by the web UI. Requires an active authenticated session.

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| POST | `/api/analyze` | `{text, session_id?, prompt_text?}` | `{metrics, z_scores, act_composite, sci, alert}` |
| GET | `/api/baseline` | — | `{status, n_samples, name}` |
| GET | `/api/sessions` | — | `{sessions[]}` |
| POST | `/api/sessions` | `{name}` | `{session_id}` |
| GET | `/api/sessions/{id}` | — | `{session, turns[], total_turns, page}` |
| DELETE | `/api/sessions/{id}` | — | `{ok}` |
| POST | `/api/batch-analyze` | multipart: `file`, `session_name?`, `text_column?`, `prompt_column?` | SSE stream of per-row results |

### Insights & Streaming

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/insights/state` | Aggregated state matrix, regime distribution, session heat |
| GET | `/api/insights/activity` | Activity heatmap (day × hour, last 90 days) |
| GET | `/api/insights/trends` | Metric trends from last N analyses |
| GET | `/api/stream/events` | SSE endpoint for real-time analysis event notifications |

### API Key Management

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| POST | `/api/keys` | `{name?}` | `{id, key, prefix, name}` — key shown only once |
| GET | `/api/keys` | — | `{keys[]}` |
| DELETE | `/api/keys/{id}` | — | `{ok}` |

---

## Rate Limits by Plan

| Plan | Analyses / Month | Sessions | API Access |
|------|-----------------|----------|------------|
| Free | 50 | 5 | No |
| Pro | 5,000 | Unlimited | Yes |
| Enterprise | Unlimited | Unlimited | Yes |

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Unauthorized — missing or invalid credentials |
| 403 | Plan restriction — endpoint not available on current plan |
| 404 | Resource not found |
| 422 | Invalid request body |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
