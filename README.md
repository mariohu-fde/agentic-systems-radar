# 📡 Agentic Systems Engineering Radar & ADRs

> **Curated by Mario Hu (`@mariohu-fde`)**  
> Distilling frontier `arXiv` agentic systems research into **production-grade engineering mechanisms, trade-off matrices, and Architecture Decision Records (ADRs)**.

---

## 🎯 Purpose: Compilation Over Retrieval

Academic agent papers frequently optimize for benchmark leaderboards while ignoring production constraints such as **context-window inflation**, **KV-cache invalidation**, **confirmation bias in multi-step debugging**, and **skill library rot**.

Every digest in this repository follows a strict **Systems Engineering Lens**:
1. **What production failure mode does this paper solve?**
2. **What is the concrete algorithmic mechanism?**
3. **How does it translate into runnable Python / LangGraph / Pydantic code in [`cloudops-autonomous-agent`](https://github.com/mariohu-fde/cloudops-autonomous-agent)?**

---

## 📚 Curated Research Synthesis Catalog

### 1. Self-Evolving Agent Harnesses & Counterfactual Memory
- 📄 **[Self-Evolving Harnesses, Counterfactual Replay & Constitutional Caps](digests/2026-09_self-evolving-agents-and-counterfactual-replay.md)**
  - **Key Papers**: *Dream-RSI* (`arXiv:2609.14858`), *RRSI / Total Cost of Agency* (`arXiv:2609.23790`), *RetireOPD* (`arXiv:2609.20784`), *SWE-Router* (`arXiv:2607.00053`).
  - **Core Engineering Takeaways**:
    - **Negative-Knowledge Contracts (`DISPROVEN_DEAD_END`)**: Why recording falsified hypotheses prevents multi-agent confirmation loops.
    - **Counterfactual Replay Gate (`CF_Value`)**: Evaluating synthesized playbooks via offline trajectory replay before merging into the production skill index.
    - **The `<= 120` Line Constitutional Cap**: Preventing instruction bloat ("Total Cost of Agency") through overflow consolidation.

### 2. Context Compaction, Gated Memory & Tool Hygiene
- 📄 **[5-Tier Context Compaction & Schemaless SQLite-JSON1 Tool Reduction](digests/2026-09_context-compaction-and-gated-memory.md)**
  - **Key Papers & Patterns**: *Gated-Memory Routing* (`LongMemEval-V2`), *Refining Over Resampling* (`arXiv:2608.05643`), *Structured Output Quality Tax*.
  - **Core Engineering Takeaways**:
    - **Prefix KV Cache Preservation**: Why mutating system prompts mid-trajectory destroys TTFT latency and adds a 5–10x token cost multiplier.
    - **Zero-Bloat `AfterToolCallback` Compaction**: Intercepting >50KB JSON telemetry payloads, storing raw artifacts in an out-of-band `SQLite-JSON1` store, and injecting only a compact Schema Skeleton into the LLM context.
    - **Decoupled Reasoning vs. Formatting**: Separating free-form root-cause reasoning from strict JSON schema formatting to eliminate the "Structured Output Quality Tax."

### 3. Multi-Agent Falsification & Graph-Augmented Retrieval
- 📄 **[Falsification-First Multi-Agent DAGs & Graph-Augmented RAG](digests/2026-09_falsification-dags-and-graph-rag.md)**
  - **Key Papers**: *RepoMAS* (`arXiv:2609.11790`), *GraMRAG* (`arXiv:2609.14066`), *SAGE* (`arXiv:2609.35412`).
  - **Core Engineering Takeaways**:
    - **Anti-Anchoring Prompting (`FAILED_ATTEMPT`)**: Framing prior agent steps as unverified attempts that must be falsified against hard telemetry.
    - **Readiness Gate (`should_decompose`)**: Dynamic complexity gating to bypass heavy multi-agent orchestration on deterministic single-hop incidents.

---

## 🏛️ Architecture Decision Records (ADRs)

- **[ADR-001: Falsification-First StateGraph over Open-Ended ReAct Loops](adrs/ADR-001-falsification-first-stategraph.md)**
- **[ADR-002: Negative-Knowledge (`DISPROVEN_DEAD_END`) & `<=120` Line Cap for Skill Distillation](adrs/ADR-002-negative-knowledge-dead-end-contracts.md)**
