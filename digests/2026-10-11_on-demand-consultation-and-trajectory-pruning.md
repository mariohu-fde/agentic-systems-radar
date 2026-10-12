# 2026-10-11: On-Demand Heterogeneous Consultation & Selective Trajectory Pruning

**Papers Reviewed**:
1. *MemPilot: Orchestrating On-Demand Multimodal Memory Curation for LLM Agents* (`arXiv:2610.06830`, Oct 2026)
2. *Selecting Long-Horizon Trajectories for Reliable and Efficient Terminal-Agent Training* (`arXiv:2610.05831`, Oct 2026)

---

## 1. Core Architectural Mechanisms

### A. Query-Aware On-Demand Memory & Execution Escalation (`arXiv:2610.06830`)
- Conventional agent memory pipelines perform **query-agnostic static summarization** at write time—burning compute on every turn while discarding fine-grained telemetry details needed by future debugging steps.
- *MemPilot* trains a multi-step routing policy that dynamically trades off accuracy, cost, and latency at runtime: serving cheap static summaries for routine turns, and invoking high-capacity models for **on-demand raw-history curation** only when complex cross-turn dependencies arise.
- This mirrors the execution-layer pattern of **Bounded On-Demand Model Consultation** (`ModelConsultTool`): running 90% of deterministic tool orchestration on a fast lightweight model (`Flash-Lite`) and invoking a frontier advisor model mid-turn under strict `max_uses` / `session_max_uses` caps (reducing per-task cost by **67%** and latency by **40%** while lifting pass rates from 75% to 90%).

### B. Supervision Horizon Pruning Against Over-Persistence (`arXiv:2610.05831`)
- Benchmarking terminal agents across varying supervision horizons reveals an inverse scaling trap: **12K-token supervision windows outperform 16K-token windows** (`29±0.7` vs. `26±0.8` resolution rate) while cutting training compute by 30%.
- Late-stage tail segments in long trajectories contain high-variance, noisy error-recovery loops. Training or prompting on unpruned long-tail retries teaches agents **Over-Persistence**—repeatedly re-executing failed commands inside dead-end states rather than pivoting hypotheses.

---

## 2. Production Stress-Test & Engineering Critique

| Paper Claim | Production Failure Mode | Systems Engineering Verdict |
| :--- | :--- | :--- |
| Replace static write-time summarization with RL-routed on-demand memory curation (`arXiv:2610.06830`) | Training a custom RL router adds operational complexity, while re-reading raw transcripts on every turn blows up TTFT latency. | **Combine Out-of-Band `SQLite-JSON1` Storage (`ADR-003`) with Bounded `ModelConsultTool` Escalation**: Store lossless raw tool payloads out-of-band at zero LLM write cost; let the lightweight orchestrator query specific JSON slices or escalate to a frontier advisor model strictly on high-ambiguity decision nodes (`max_uses=1`). |
| Truncate noisy long-horizon retry tails to prevent over-persistence (`arXiv:2610.05831`) | Feeding verbose failed-retry transcripts into few-shot prompts or fine-tuning datasets anchors the agent on repeating broken commands. | **Enforce `DisprovenDeadEnd` Compaction (`ADR-002`)**: Collapse multi-turn failed probing sequences into a single typed `DisprovenDeadEnd` record (`hypothesis_id + falsifying_evidence`), preserving the pivot signal while stripping the retry noise. |

---

## 3. Concrete Distributed Systems Patterns: Context-Stripped Redaction & Terminal-Event Parser Traps

Two production failure modes illustrate what happens when security and evaluation layers assume agent state is a flat, single-turn text string:

1. **Context-Stripped PII Redaction Blind Spots (Isolated Columns & Supervisor Payloads)**:
   - **Bypass 1 (Unredacted Supervisor Detections)**: When a runtime guardrail (`Agent Supervisor`) flags raw user input (e.g., DTMF keypad sequences) as `INVALID_TEXT`, writing the raw snippet into `supervisor.detections[].details` bypasses standard transcript DLP unless the supervisor payload path is explicitly wired into the redaction pipeline.
   - **Bypass 2 (Context-Stripped Variable Columns)**: When a redaction processor serializes updated session variables (`updatedVariables`) as isolated DLP table columns stripped of full-conversation context, DLP templates that rely on specific schema field names (`FieldId == 'pin'`) or conversational hotword proximity silently fail to redact sensitive values stored in generic state variables.
   - **Engineering Rule**: Redaction rules for stateful agents must enforce **field-agnostic pattern matchers** across both primary transcripts, state-variable diffs, and supervisor exception payloads.
2. **Multi-Agent Trajectory Terminal-Event Parser Trap (`Trace[-1]` Assumption)**:
   - Evaluation harnesses that hardcode `resp_item[-1]["content"]["parts"][0]["text"]` assume every agent run terminates with a plain-text assistant message.
   - In multi-agent handoffs, trajectories routinely terminate with a `tool_call`, `agent_transfer`, or `thought` part—triggering a `KeyError: 'text'` that silently replaces the agent's output with a parser error string before scoring. Multi-agent evaluators must traverse backwards to the last user-facing synthesis event rather than indexing `Trace[-1]` blindly.
