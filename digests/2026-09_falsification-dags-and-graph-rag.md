# 🔬 Falsification-First Multi-Agent DAGs & Graph-Augmented RAG

> **Synthesis Date**: September 2026  
> **Primary Literature**: *RepoMAS* (`arXiv:2609.11790`), *GraMRAG* (`arXiv:2609.14066`), *SAGE* (`arXiv:2609.35412`)  
> **Reference Implementation**: [`cloudops-autonomous-agent/src/cloudops_agent/schemas/incident.py`](https://github.com/mariohu-fde/cloudops-autonomous-agent)

---

## 1. Executive Summary

When multiple LLM agents collaborate on root-cause analysis, standard "consensus voting" often amplifies sycophancy: downstream verifier agents rubber-stamp the lead planner's initial hypothesis even when log evidence is circumstantial.

To break confirmation bias in production incident response, the orchestration topology must shift from **Verification-First** to **Falsification-First**.

---

## 2. Core Architectural Patterns

### 2.1 Anti-Anchoring Prompting (`FAILED_ATTEMPT` Framing)
When passing an initial diagnostic hypothesis from the Planner node to the Investigator/Verifier node, never frame it as a "Proposed Root Cause." Instead, serialize it under an explicit **`[UNVERIFIED_HYPOTHESIS / POTENTIAL_FAILED_ATTEMPT]`** block alongside all previously recorded `DISPROVEN_DEAD_END` items. The Investigator's primary reward signal is finding telemetry that *refutes* weak hypotheses early.

### 2.2 Dynamic Readiness Gate (`should_decompose`)
Not every incident warrants a 5-node multi-agent DAG. A lightweight readiness gate evaluates incoming symptom entropy:
- **Single-hop deterministic signatures** (e.g., explicit quota exhaustion or known error code with 1:1 runbook match) route via a fast-path single-step resolver.
- **Multi-service cascading anomalies** (e.g., tail-latency spikes across RPC boundaries with zero explicit 5xx errors) trigger full multi-hypothesis decomposition.
