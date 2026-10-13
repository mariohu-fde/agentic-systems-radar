# 2026-10-08: Harness Dominance over Model Scale & Slot-Parallel Tool Decoding

**Papers Reviewed**:
1. *Agents Are Systems, Not Models: Rethinking Agentic Evaluation* (`arXiv:2610.01618`, Oct 2026)
2. *SchemaFill: Efficient LLM Tool Calling via Slot-Parallel Speculative Decoding* (`arXiv:2610.07086`, Oct 2026)

---

## 1. Core Paper Mechanisms

### A. System Harness Dominance over Parameter Scale (`arXiv:2610.01618`)
- Analyzing **18,000+ complex agentic task trajectories** across multiple benchmarks reveals that **input context structure, execution time budgets, and harness constraint configurations** account for significantly larger variance in task success rate and cost than underlying model parameter differences.
- Swapping in a larger foundation model without fixing context noise, tool-output bloat, or state-transition constraints yields negligible reliability gains while inflating inference cost.

### B. Slot-Parallel Speculative Decoding (`arXiv:2610.07086`)
- Complex, deeply nested JSON tool-calling schemas create sequential autoregressive decoding bottlenecks.
- *SchemaFill* parses the JSON Schema grammar tree to identify mutually independent fields (slots) and decodes them in parallel via speculative decoding masks, significantly reducing time-to-first-tool-call and total structured generation latency.

---

## 2. Paper Stress-Test & Systems Critique

| Paper Claim | Architectural Failure Mode | Open-Source Implementation Verdict |
| :--- | :--- | :--- |
| Harness design dominates base model scale across 18,000+ trajectories (`arXiv:2610.01618`) | Teams routinely wait for next-generation model releases to fix multi-step agent loops that are actually caused by unconstrained ReAct scratchpads and uncompacted tool outputs. | **Invest Engineering Effort in Deterministic Harness Gates**: Bound tool-output token residency via `SQLite-JSON1` compaction (`ADR-003`) and enforce falsification-first state transitions (`ADR-001`) rather than relying on larger models to parse noisy contexts. |
| Accelerate structured tool calls via server-side slot-parallel speculative decoding (`arXiv:2610.07086`) | Standard commercial and hosted LLM APIs do not expose custom syntax-tree speculative decoding masks to client applications. | **Flatten Client-Side `Pydantic V2` Tool Schemas**: Achieve equivalent decoding latency reductions on standard APIs by eliminating deeply nested sub-models, stripping optional boilerplate fields from tool input schemas, and decoupling free-form reasoning from final JSON serialization. |

---

## 3. Open-Source `LangGraph` / `Pydantic` Blueprint

1. **Flat Tool Argument Contracts**: Keep all `Pydantic` `BaseModel` tool schemas at depth `<= 2` with primitive types (`str`, `int`, `Literal` enums) to minimize constrained-decoding overhead.
2. **Harness-First Ablation Testing**: In evaluation suites (`replay_scorer.py`), benchmark harness interventions (compaction thresholds, max probing budgets, anti-anchoring prompts) holding the base model constant.
