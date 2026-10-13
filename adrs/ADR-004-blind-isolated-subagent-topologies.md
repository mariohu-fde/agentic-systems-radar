# ADR-004: Blind-Isolated Subagent Probing over All-to-All Broadcast

- **Status**: Accepted
- **Date**: 2026-10-08

## Context
In multi-agent reasoning and diagnostic workflows, a common orchestration anti-pattern is **All-to-All Broadcast**—sharing a global scratchpad where parallel subagents read each other's intermediate hypotheses in real time.

Empirical findings on collective LLM inference (*Communication Shapes Collective Inference in Self-Adapting LLM Societies*, `arXiv:2610.05041`) demonstrate that dense, unrestrained broadcast communication across LLM societies induces three structural failure modes:
1. **Premature False Consensus (Groupthink)**: An early, plausible-sounding but unverified guess from one agent anchors the entire swarm, collapsing hypothesis diversity and suppressing independent error correction.
2. **Correlated Hallucination Cascades**: When peer agents observe a confident intermediate claim in a shared scratchpad, they prematurely halt deeper verification of orthogonal evidence paths.
3. **Quadratic Context Inflation**: Broadcasting raw peer reasoning across $N$ agents inflates per-round token consumption by $O(N^2)$ without improving final ensemble accuracy.

## Decision
We enforce **Blind-Isolated Subagent Probing (`Topology = Star-to-Falsification-Gate`)** via `LangGraph` `Send` primitives:
1. **Information-Isolated Dispatch**: Each parallel subagent receives a strictly scoped input slice with zero visibility into sibling subagents' active hypotheses or intermediate scratchpads.
2. **Independent Evidence Verification**: Subagents must ground their local verdict exclusively in primary tool observations rather than peer assertions.
3. **Typed Convergence at the Gate**: Subagents return strictly typed `EvidenceBlock` or `DisprovenDeadEnd` (`ADR-002`) records to the central Falsification Gate (`ADR-001`), which cross-examines competing hypotheses only after independent evidence collection completes.

## Consequences
- **Positive**: Eliminates inter-agent confirmation cascades (`arXiv:2610.05041`); preserves hypothesis diversity across orthogonal search paths; bounds per-subagent token cost to $O(1)$.
- **Negative / Boundary**: Subagents cannot dynamically short-circuit each other mid-flight if one branch immediately resolves the task; mitigated by imposing a strict per-branch token and step budget (`max_epistemic_rounds`).
