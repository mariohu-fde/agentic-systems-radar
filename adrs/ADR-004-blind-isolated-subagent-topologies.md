# ADR-004: Blind-Isolated Subagent Probing over All-to-All Broadcast

- **Status**: Accepted
- **Date**: 2026-10-08

## Context
In multi-agent diagnostic workflows (e.g., concurrently probing compute cluster quota, storage latency, and hierarchical pipeline configs), a common orchestration anti-pattern is **All-to-All Broadcast**—sharing a global scratchpad where parallel subagents read each other's intermediate hypotheses in real time.

Empirical findings on collective LLM inference (`arXiv:2610.05041`) and production troubleshooting traces show that shared intermediate scratchpads induce three failure modes:
1. **Premature False Consensus (Groupthink)**: An early, plausible-sounding but unverified guess from one subagent anchors the entire swarm, suppressing independent falsification.
2. **Top-Level Config Illusion**: When one subagent reports that a top-level configuration override looks healthy, peer subagents prematurely halt deeper inspection of leaf-level execution descriptors (missing hierarchical config shadowing).
3. **Cross-Contaminated Context Windows**: Broadcasting raw peer reasoning inflates every subagent's token footprint by $O(N^2)$.

## Decision
We enforce **Blind-Isolated Subagent Probing (`Topology = Star-to-Falsification-Gate`)**:
1. **Information-Isolated Dispatch**: Each domain prober subagent receives strictly scoped read-only parameters (`Telemetry-Slice-Only` or `Leaf-Config-Path-Only`) with zero visibility into sibling subagents' active hypotheses.
2. **Leaf-Node Effective State Verification**: Subagents are contractually required to inspect effective leaf-node runtime state (e.g., child task `footprints` and admission control counters) rather than top-level config intent.
3. **Typed Convergence at the Gate**: Subagents return strictly typed `EvidenceBlock` or `DisprovenDeadEnd` records to the central Falsification Gate (`ADR-001`), which cross-examines competing hypotheses only after independent evidence collection completes.

## Consequences
- **Positive**: Eliminates inter-agent confirmation cascades; preserves diagnostic diversity across orthogonal failure domains; bounds per-subagent token cost to $O(1)$.
- **Negative / Boundary**: Subagents cannot dynamically short-circuit each other mid-flight if one agent immediately finds a smoking gun; mitigated by imposing a strict timeout and `max_epistemic_rounds` budget per prober.
