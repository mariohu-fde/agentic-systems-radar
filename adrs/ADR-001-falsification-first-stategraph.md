# ADR-001: Falsification-First StateGraph over Open-Ended ReAct Loops

- **Status**: Accepted
- **Date**: 2026-09-28
- **Author**: Mario Hu (`@mariohu-fde`)

## Context
Open-ended single-agent ReAct (`Reason + Act`) loops struggle in complex cloud troubleshooting scenarios because:
1. They exhibit strong **anchoring bias** toward the first error message found in logs.
2. They lack structural checkpoints to prune falsified hypotheses, leading to cyclic tool calling when an initial fix fails.

## Decision
Adopt an explicit **5-Node Falsification-First `StateGraph`** (`Ingest & Triage ➔ Hypothesize ➔ Investigate ➔ Falsify & Verify ➔ Synthesize RCA`) governed by strongly-typed **Pydantic V2** state contracts. Every hypothesis must declare explicit falsification criteria before tool dispatch.

## Consequences
- **Positive**: Deterministic loop bounds, auditable state transitions, and zero repeat probing of falsified dead ends.
- **Negative**: Slightly higher upfront schema definition overhead compared to untyped prompt chains.
