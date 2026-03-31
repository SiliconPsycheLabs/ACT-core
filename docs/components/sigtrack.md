# SIGTRACK — Signature Tracking for Regime and Attractor Conflict Knowledge

## Purpose

SIGTRACK is the memory and pattern recognition layer of the platform. It serves three functions:

1. **Forensic ledger** — Every analysis turn is stored in a tamper-evident, cryptographically chained record
2. **Signature extraction** — Confirmed regime shift patterns are compressed into reusable behavioral signatures
3. **Fast matching** — Incoming analysis is matched against the signature library to detect known patterns instantly

Without SIGTRACK, each analysis is stateless. With SIGTRACK, every new analysis is compared against the entire history of known behavioral patterns.

---

## The Forensic Ledger

Every turn analyzed by ACT is stored as an entry in the SIGTRACK ledger. Each entry contains:
- The full metric vector and Z-score vector for that turn
- The composite score and alert level
- The response text and (optionally) the prompt text
- A cryptographic hash that chains this entry to the previous one

The chaining is critical: if any entry in the ledger is modified after the fact, the hash chain breaks. This makes the ledger **tamper-evident** — it can be used as forensic evidence in incident investigations.

The ledger answers questions like:
- What was the model doing in the 10 turns before the incident?
- Has this behavioral pattern appeared before?
- When did the model's behavior first start deviating from baseline?

---

## Behavioral Signatures

A **signature** is a compressed representation of a known regime shift pattern. It captures:

- The **temporal shape** of the shift — how the metrics evolved across the N turns of the event (a matrix of metric values over time)
- **Importance weights** — which metrics are most characteristic of this particular shift type
- **Tolerance bounds** — how much variation is allowed when matching new events against this signature
- A **compressed feature vector** for fast approximate matching
- Metadata: which shift type it represents, how many times it has been observed, confidence score

Signatures are extracted from confirmed regime shift events — events that have been verified by an analyst or validated through repeated observation.

---

## Signature Matching

For every new analysis turn, SIGTRACK runs a fast matching step against the signature library. This operates in two phases:

### Phase 1: Fast Path (compressed vectors)
A nearest-neighbor search against compressed 4-dimensional feature vectors. This runs in constant time regardless of library size and filters the library down to a small set of candidate signatures.

### Phase 2: Verification (full comparison)
The candidate signatures are compared against the recent turn history using the full metric representation and tolerance bounds. If the match score is within the signature's tolerance, a match is reported.

A positive match means: *this behavioral pattern matches a known regime shift signature*. It tells the analyst not just that something is happening, but that it matches a previously observed and classified event.

---

## The Learning Cycle

SIGTRACK improves over time through a learning cycle:

1. ACT detects a RED alert
2. The event is stored in the ledger and flagged for analyst review
3. The analyst labels it: `confirmed` or `false_positive`
4. Confirmed events are fed to the signature extractor
5. New or updated signatures are added to the library
6. Future matching benefits from the improved library

This creates a feedback loop: the more the platform is used, the better its pattern recognition becomes. Signatures extracted from one deployment are reusable across all deployments facing the same model or the same class of behavioral patterns.

---

## SIGTRACK and Forensic Investigations

The SIGTRACK ledger is designed to support post-incident forensic analysis:

- **Reproducibility** — The exact metric vector for any past turn can be retrieved and re-analyzed
- **Timeline reconstruction** — The sequence of turns leading up to an incident can be visualized as a behavioral trajectory
- **Signature lookup** — The incident pattern can be matched against the library to determine if this is a known attack vector or a novel event
- **Chain of custody** — The cryptographic chain provides verifiable evidence that the record has not been tampered with
