# ACTIVE — Attractor Conflict Testing via Induced Elicitation

## Purpose

ACTIVE is the active probing layer. Where ACT passively observes the model's behavior in normal operation, ACTIVE deliberately probes it — sending controlled stimuli designed to reveal behavioral boundaries that would never appear in ordinary use.

The distinction is the same as in medicine: a passive monitor tracks vital signs; a stress test pushes the system to reveal how it responds under controlled load. ACTIVE is the stress test.

---

## Why Active Probing Is Necessary

Passive monitoring has a fundamental limitation: it can only detect regime shifts that actually occur during observed sessions. Several categories of vulnerability are invisible to passive monitoring:

- **Sub-threshold migration** — silent drift that never triggers per-turn alerts
- **Boundary instability** — inconsistent responses to a specific input class, invisible if that class is rare in normal operation
- **Latent vulnerabilities** — input patterns that would cause acute collapse, but haven't been tried yet
- **Adversarial edge cases** — inputs specifically crafted to exploit boundary conditions

ACTIVE addresses all of these by directly probing the model's boundaries under controlled conditions.

---

## How ACTIVE Works

ACTIVE operates through **probe sequences** — structured sets of inputs designed to test a specific behavioral hypothesis. Each probe is designed to elicit a specific class of response and measure how the model reacts.

### Probe Design Principles

- **Controlled variation** — probes vary one dimension at a time (e.g., escalating pressure on a single topic) to isolate the cause of any behavioral shift
- **Boundary mapping** — probes approach the model's boundaries incrementally, measuring at each step
- **Repeatability** — the same probe set can be re-run against a different model version to detect behavioral regressions

### Probe Types

| Probe Type | What It Tests |
|------------|---------------|
| **Pressure escalation** | How does the model respond as instruction pressure increases? At what point does the boundary shift? |
| **Register shift** | Does the model maintain consistent behavior across different communication styles (formal, casual, technical)? |
| **Boundary stress** | Direct probes at the edges of known behavioral rules |
| **Decay testing** | Does the model's adherence to a behavioral constraint weaken over a long context? |
| **Cross-domain transfer** | Does a vulnerability in one topic domain transfer to related domains? |

---

## ACTIVE and ACT Integration

ACTIVE does not replace ACT — it uses ACT. Every probe response is analyzed by the ACT metric engine. The output is not just "did the model comply?" but a full behavioral profile of how the model responded, using the same metrics and alert system as passive monitoring.

This means ACTIVE results are:
- Stored in the SIGTRACK ledger alongside passive monitoring data
- Comparable across model versions using the same metric space
- Usable for signature extraction when a probe reliably elicits a specific behavioral pattern

---

## Regime Shift Classification via ACTIVE

When ACTIVE detects a behavioral shift, it feeds the pattern to the regime shift classifier. For each shift type:

- **Acute collapse** — ACTIVE identifies the specific input class that triggered it and characterizes the trigger
- **Boundary instability** — ACTIVE measures the variance across repeated probes of the same class
- **Sub-threshold migration** — ACTIVE provides ground truth probes to detect baseline drift even when passive metrics haven't moved

The result of an ACTIVE session is a **behavioral boundary map** for the tested model: where it is stable, where it is vulnerable, and what types of inputs produce which behavioral outcomes.
