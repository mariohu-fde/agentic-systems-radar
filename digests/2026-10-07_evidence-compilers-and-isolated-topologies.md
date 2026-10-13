# 2026-10-07: Deterministic Evidence Compilers & Blind-Isolated Subagent Topologies

**Papers Reviewed**:
1. *FinNextAssist: Towards Professional Financial Deep Research Assistant* (`arXiv:2610.03174`, Oct 2026)
2. *Communication Shapes Collective Inference in Self-Adapting LLM Societies* (`arXiv:2610.05041`, Oct 2026)

---

## 1. Core Paper Mechanisms

### A. Intermediate Evidence Compilation (`arXiv:2610.03174`)
- Multi-agent deep-research workflows typically pipe raw heterogeneous tool outputs (tabular SQL results, document chunks, API responses) directly into the central orchestrator's context window.
- *FinNextAssist* interposes a dedicated **Evidence Compiler** between domain worker agents (`TabAgent`, `HeteroAgent`) and the central reasoning engine, normalizing raw observations into compact, provenance-linked evidence blocks before synthesis.

### B. Topological Isolation vs. All-to-All Broadcast (`arXiv:2610.05041`)
- Game-theoretic multi-agent experiments demonstrate that **All-to-All Broadcast** communication topologies drive LLM populations to rapidly converge on **high-confidence false consensus** (groupthink).
- Enforcing **sparse, information-isolated topologies**—where subagents gather evidence independently before structured aggregation—preserves population diversity and collective error-correction capacity.

---

## 2. Paper Stress-Test & Systems Critique

| Paper Claim | Architectural Failure Mode | Open-Source Implementation Verdict |
| :--- | :--- | :--- |
| Insert an LLM-based Evidence Compiler between subagents and the primary reasoner (`arXiv:2610.03174`) | Adding an extra LLM inference pass per tool call inflates end-to-end P99 latency and compounds token cost. | **Replace LLM Compiler with Deterministic `SQLite-JSON1` Preprocessor (`ADR-003`)**: Execute evidence compilation via deterministic schema extraction (`<400-token` digest + immutable `payload_ref` pointer), cutting context noise by ~70% at zero LLM latency overhead. |
| Restrict inter-agent communication topology to prevent premature consensus (`arXiv:2610.05041`) | When parallel subagents read each other's intermediate scratchpads, an early unverified guess anchors the entire swarm. | **Enforce Blind-Isolated Subagent Probing (`ADR-004`)**: Dispatch parallel `LangGraph` subagents (`Send` API) with isolated state slices. Subagents return strictly typed `EvidenceBlock` or `DisprovenDeadEnd` records to the central gate without lateral peer broadcast. |

---

## 3. Open-Source `LangGraph` / `Pydantic` Blueprint

1. **Star-to-Gate Fan-Out (`LangGraph Send` API)**: Map each candidate hypothesis to an isolated worker node that receives only its hypothesis description and read-only tool handles—never the shared conversation history of sibling workers.
2. **Provenance-Linked `EvidenceBlock`**: Require every worker to return a `Pydantic` `EvidenceBlock(hypothesis_id, payload_ref, key_findings, falsified)` validated against deterministic `SQLite-JSON1` slices.
