# 2026-10-06: Epistemic Action Gates & Typed State Substrates

**Papers Reviewed**:
1. *Before Agents Decide: Epistemic Action in LLM-Based Systems* (`arXiv:2610.00511`, NeurIPS 2026 FAST)
2. *The Agentic Company OS: Substrate Inversion* (`arXiv:2609.13334`)

---

## 1. Core Architectural Mechanisms

### A. Epistemic vs. Pragmatic Action Partitioning (`arXiv:2610.00511`)
- **Epistemic Actions**: Read-only diagnostic probes executed strictly to reduce state uncertainty (entropy) across competing hypotheses (e.g., querying `INFORMATION_SCHEMA.JOBS`, inspecting log slices).
- **Pragmatic Actions**: State-mutating operations that alter external production systems (e.g., rolling back a deployment, shifting traffic, modifying quota overrides).

### B. Substrate Inversion (`arXiv:2609.13334`)
- Empirical telemetry across enterprise agent deployments shows **>70% failure within 30 days** when long-horizon state is passed via unstructured Markdown summaries or raw conversation history.
- **Substrate Inversion** replaces free-text memory buffers with strongly-typed, schema-enforced relational state stores as the single source of truth across multi-agent handoffs.

---

## 2. Production Stress-Test & Engineering Critique

| Paper Claim | Production Failure Mode | Systems Engineering Verdict |
| :--- | :--- | :--- |
| LLMs can autonomously balance epistemic probing vs. pragmatic action (`arXiv:2610.00511`) | In ambiguous production incidents, unconstrained LLMs fall into **infinite epistemic probing loops** (repeatedly calling read tools without converging) or prematurely execute destructive mutations. | **Adopt with Hard State-Machine Gate**: Enforce a deterministic transition boundary (`Investigate ➔ Falsify/Verify ➔ Synthesize`) with a strict probing budget and `DisprovenDeadEnd` anti-anchoring lock. |
| Replace all enterprise SaaS layers with a unified agentic substrate (`arXiv:2609.13334`) | Ripping out existing enterprise storage layers is operationally unrealistic; however, **banning schema-less free text for inter-agent state** is essential. | **Adopt Typed Contracts (`Pydantic V2` + `SQLite-JSON1`)**: Bind every diagnostic turn and tool output to an immutable ID (`payload_ref` / `turn_id`) rather(than) positional array indices. |

---

## 3. Concrete Failure Pattern: Positional Index Shift vs. Immutable `turn_id`

A classic anti-pattern of weak state substrates occurs in streaming evaluation runners:
- When a streaming client sends a short input and half-closes before Voice Activity Detection (VAD) registers trailing silence, the runtime drops Turn $k$.
- If the evaluation harness binds execution traces by **array position (`Trace[0]`, `Trace[1]`)** rather than an immutable **`turn_id`**, Turn $k+1$'s tool execution is rendered on Card $k$, while the expectation matcher grades Turn $k$ as `MISSED`.
- **Rule**: Never join asynchronous agent telemetry by positional offset; enforce primary-key correlation (`turn_id` / `incident_id`) at the schema boundary.
