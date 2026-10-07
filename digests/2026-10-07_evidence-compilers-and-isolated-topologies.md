# 2026-10-07: Deterministic Evidence Compilers & Blind-Isolated Subagent Topologies

**Papers Reviewed**:
1. *FinNextAssist: Towards Professional Financial Deep Research Assistant* (`arXiv:2610.03174`, Oct 2026)
2. *Communication Shapes Collective Inference in Self-Adapting LLM Societies* (`arXiv:2610.05041`, Oct 2026)

---

## 1. Core Architectural Mechanisms

### A. Intermediate Evidence Compilation (`arXiv:2610.03174`)
- Multi-agent retrieval workflows typically pipe raw heterogeneous tool outputs (tabular SQL results, log slices, unstructured runbooks) directly into the orchestrator's reasoning window.
- *FinNextAssist* interposes a dedicated **Evidence Compiler** between domain worker agents (`TabAgent`, `HeteroAgent`) and the central reasoning engine, normalizing raw observations into compact, provenance-linked evidence blocks before synthesis.

### B. Topological Isolation vs. All-to-All Broadcast (`arXiv:2610.05041`)
- Game-theoretic multi-agent experiments demonstrate that **All-to-All Broadcast** communication topologies drive LLM populations to rapidly converge on **high-confidence false consensus** (groupthink).
- Enforcing **sparse, information-isolated topologies**—where subagents gather evidence independently before structured aggregation—preserves population diversity and self-correction capacity.

---

## 2. Production Stress-Test & Engineering Critique

| Paper Claim | Production Failure Mode | Systems Engineering Verdict |
| :--- | :--- | :--- |
| Insert an LLM-based Evidence Compiler between subagents and the primary reasoner (`arXiv:2610.03174`) | Adding an extra LLM inference pass per tool call inflates end-to-end P99 latency and compounds token cost. | **Replace LLM Compiler with Deterministic `SQLite-JSON1` Preprocessor**: Execute evidence compilation via deterministic schema extraction (`<400-token` digest + immutable `payload_ref` pointer), cutting context noise by ~70% at zero LLM latency overhead. |
| Restrict inter-agent communication topology to prevent premature consensus (`arXiv:2610.05041`) | When parallel diagnostic subagents (e.g., Database Prober vs. Compute Scheduler Prober) read each other's intermediate scratchpads, an early unverified guess anchors the entire swarm. | **Enforce Blind-Isolated Subagent Probing**: Dispatch subagents with isolated context scopes (`File-Path-Only` / `Telemetry-Slice-Only`). Subagents return strictly typed `EvidenceBlock` or `DisprovenDeadEnd` records to the falsification gate without lateral peer broadcast. |

---

## 3. Concrete Distributed Systems Pattern: Hierarchical Config Shadowing & Quota Starvation

Two recurring distributed control-plane failure modes reinforce why deterministic evidence compilers must inspect **effective leaf-node state** rather than top-level config intent:
1. **Priority Without Admission Reservation**: Elevating a workload's scheduling priority has zero effect if lower-priority batch jobs are permitted to consume 100% of a shared resource pool without a hard charging cap (e.g., `0.5` batch ceiling). High-priority jobs starve at Admission Control unless capacity is explicitly partitioned.
2. **Leaf-Node Override Shadowing**: In hierarchical pipeline runners, setting a top-level cluster migration override (`scheduling_override.allowed_cells`) is silently ignored whenever child sub-tasks (`footprints`) declare their own local `scheduling` block. Diagnostic agents must verify effective leaf-level execution descriptors rather than trusting top-level configuration diffs.
