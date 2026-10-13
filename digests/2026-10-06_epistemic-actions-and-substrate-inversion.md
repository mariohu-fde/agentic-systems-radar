# 2026-10-06: Epistemic Action Gates & Typed State Substrates

**Papers Reviewed**:
1. *Before Agents Decide: Epistemic Action in LLM-Based Systems* (`arXiv:2610.00511`, NeurIPS 2026 FAST)
2. *The Agentic Company OS: Substrate Inversion* (`arXiv:2609.13334`)

---

## 1. Core Paper Mechanisms

### A. Epistemic vs. Pragmatic Action Partitioning (`arXiv:2610.00511`)
- **Epistemic Actions**: Read-only information-gathering probes executed strictly to reduce state uncertainty (entropy) across competing hypotheses before committing to a decision.
- **Pragmatic Actions**: State-mutating operations that alter the external environment.
- The authors formalize pre-decision epistemic exploration in LLM agents, analyzing when agents over-probe (analysis paralysis) versus under-probe (premature action under high uncertainty).

### B. Substrate Inversion (`arXiv:2609.13334`)
- Empirical evaluations of long-horizon multi-agent workflows show **>70% degradation** when inter-agent state is passed via unstructured Markdown summaries or raw conversation buffers.
- **Substrate Inversion** replaces free-text scratchpads with strongly-typed, schema-enforced relational state stores as the single source of truth across multi-agent handoffs.

---

## 2. Paper Stress-Test & Systems Critique

| Paper Claim | Architectural Failure Mode | Open-Source Implementation Verdict |
| :--- | :--- | :--- |
| LLMs can autonomously balance epistemic probing vs. pragmatic action via prompting (`arXiv:2610.00511`) | In high-entropy environments, unconstrained LLMs fall into **infinite epistemic probing loops** (repeatedly calling read tools without converging) or prematurely execute state mutations. | **Enforce a Hard `LangGraph` State-Machine Gate**: Separate epistemic nodes from pragmatic nodes via a deterministic conditional edge (`Investigate ➔ Falsify/Verify ➔ Synthesize`) with a strict `max_epistemic_rounds` counter and `DisprovenDeadEnd` lock (`ADR-001`, `ADR-002`). |
| Replace unstructured chat buffers with a typed relational substrate (`arXiv:2609.13334`) | Passing raw JSON arrays across agent turns causes context bloat and positional indexing errors if intermediate steps are filtered or compacted. | **Bind Every Artifact to an Immutable Primary Key (`Pydantic V2` + `SQLite-JSON1`)**: Reference tool outputs and reasoning steps strictly by UUID handles (`payload_ref`) rather than array indices (`ADR-003`). |

---

## 3. Open-Source `LangGraph` / `Pydantic` Blueprint

To operationalize `arXiv:2610.00511` and `arXiv:2609.13334` in [`cloudops-autonomous-agent`](https://github.com/mariohu-fde/cloudops-autonomous-agent):
1. **Tool Taxonomy Tagging**: Decorate every tool contract with `action_type: Literal["EPISTEMIC", "PRAGMATIC"]`.
2. **State-Level Probing Budget**: Track `epistemic_steps_remaining: int` inside the `Pydantic` graph state; once exhausted or once all competing hypotheses except one are recorded in `disproven_dead_ends`, transition deterministically to synthesis.
