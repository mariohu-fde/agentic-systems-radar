# 2026-10-11: On-Demand Multimodal Memory Curation & Supervision Horizon Pruning

**Papers Reviewed**:
1. *MemPilot: Orchestrating On-Demand Multimodal Memory Curation for LLM Agents* (`arXiv:2610.06830`, Oct 2026)
2. *Selecting Long-Horizon Trajectories for Reliable and Efficient Terminal-Agent Training* (`arXiv:2610.05831`, Oct 2026)

---

## 1. Core Paper Mechanisms

### A. Query-Aware On-Demand Memory Curation (`arXiv:2610.06830`)
- Conventional agent memory pipelines perform **query-agnostic static summarization** at write time—burning compute on every turn while discarding fine-grained details needed by future reasoning steps.
- *MemPilot* trains a multi-step routing policy via reinforcement learning that dynamically trades off accuracy, cost, and latency at runtime: serving cheap static summaries for routine turns, and invoking high-capacity models for **on-demand raw-history curation** only when complex cross-turn dependencies arise.

### B. Supervision Horizon Pruning Against Over-Persistence (`arXiv:2610.05831`)
- Benchmarking terminal agents on Terminal-Bench across varying supervision horizons reveals an inverse scaling trap: **12K-token supervision windows outperform 16K-token windows** (`29±0.7` vs. `26±0.8` resolution rate) while cutting training compute by **30%**.
- Late-stage tail segments in long trajectories contain high-variance, noisy error-recovery loops. Training or prompting on unpruned long-tail retries teaches agents **Over-Persistence**—repeatedly re-executing failed commands inside dead-end states rather than pivoting hypotheses.
- The authors propose **Selective Long-Horizon Refinement** (warm-starting on short prefixes, then selectively refining only high-likelihood continuation tails), lifting consistent pass rates (solving at least 6/8 trials) from `9` to `14`.

---

## 2. Paper Stress-Test & Systems Critique

| Paper Claim | Architectural Failure Mode | Open-Source Implementation Verdict |
| :--- | :--- | :--- |
| Replace static write-time summarization with RL-routed on-demand memory curation (`arXiv:2610.06830`) | Training a custom RL memory router adds infrastructure overhead, while re-reading raw transcripts on every turn blows up TTFT latency. | **Out-of-Band `SQLite-JSON1` Storage + On-Demand SQL Slice Retrieval (`ADR-003`)**: Store lossless raw tool payloads out-of-band at zero LLM write cost; let the orchestrator read lightweight schema skeletons by default and query raw JSON slices on demand only when deeper evidence is required. |
| Truncate noisy long-horizon retry tails to prevent over-persistence (`arXiv:2610.05831`) | Feeding verbose failed-retry transcripts into few-shot prompts or replay buffers anchors the agent on repeating broken commands. | **Enforce `DisprovenDeadEnd` Trajectory Compaction (`ADR-002`)**: Collapse multi-turn failed probing sequences into a single typed `DisprovenDeadEnd` record (`hypothesis_id + falsifying_evidence`), preserving the pivot signal while stripping the retry noise. |

---

## 3. Open-Source `LangGraph` / `Pydantic` Blueprint

1. **Lazy Read-Time Evidence Expansion**: Never summarize raw tool outputs with an LLM at write time. Keep the message history compact (`<400 tokens` per tool call) and expose `query_cached_tool_output(payload_ref, json_path)` for on-demand inspection.
2. **Dead-End Tail Truncation in Replay Buffers**: Before distilling a resolved trajectory into a few-shot skill (`replay_scorer.py`), strip all intermediate retry loops that preceded a falsified hypothesis, retaining only the single falsifying observation and the subsequent pivot.
