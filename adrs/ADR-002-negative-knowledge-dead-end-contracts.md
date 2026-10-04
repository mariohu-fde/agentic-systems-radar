# ADR-002: Negative-Knowledge (`DISPROVEN_DEAD_END`) & `<=120` Line Constitutional Cap

- **Status**: Accepted
- **Date**: 2026-09-29
- **Author**: Mario Hu (`@mariohu-fde`)

## Context
Automated skill/playbook distillation from resolved incidents typically extracts only positive resolution steps. Over time, this creates two severe production issues:
1. Future agent runs repeat the same dead-end investigations that human engineers already ruled out.
2. Unbounded appending of new rules bloats the active playbook beyond the model's effective instruction-following horizon.

## Decision
1. Mandate a **`DisprovenDeadEnd`** schema field in every incident state and distilled playbook, recording falsified claims and the telemetry proof that refuted them.
2. Enforce a hard **`<= 120` line constitutional cap** on all active playbooks via a Stage 2.5 Counterfactual Replay gate (`CF_Value >= 0.20`).

## Consequences
- **Positive**: Eliminates recurring diagnostic dead ends and bounds system-prompt token overhead.
- **Negative**: Requires an offline replay evaluation pass before promoting newly synthesized playbooks.
