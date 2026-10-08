# 2026-10-08: Harness Dominance over Model Scale & Precondition State-Machine Gates

**Papers Reviewed**:
1. *Agents Are Systems, Not Models: Rethinking Agentic Evaluation* (`arXiv:2610.01618`, Oct 2026)
2. *SchemaFill: Efficient LLM Tool Calling via Slot-Parallel Speculative Decoding* (`arXiv:2610.07086`, Oct 2026)

---

## 1. Core Architectural Mechanisms

### A. System Harness Dominance over Parameter Scale (`arXiv:2610.01618`)
- Analyzing **18,000+ complex agentic task trajectories** reveals that **input context structure, execution time budgets, and harness constraint configurations** account for significantly larger variance in task success rate and cost than underlying model parameter differences.
- Upgrading the base LLM without fixing context noise or state-transition preconditions yields negligible reliability gains while inflating inference cost.

### B. Slot-Parallel Speculative Decoding vs. Schema Flattening (`arXiv:2610.07086`)
- Deeply nested JSON tool-calling schemas create autoregressive decoding bottlenecks.
- *SchemaFill* uses grammar-tree dependency analysis to decode independent JSON slots in parallel via speculative decoding, reducing time-to-first-tool-call and total completion latency.

---

## 2. Production Stress-Test & Engineering Critique

| Paper Claim | Production Failure Mode | Systems Engineering Verdict |
| :--- | :--- | :--- |
| Model upgrades alone fail to rescue poorly constrained agent harnesses (`arXiv:2610.01618`) | When stateful workflows stall (e.g., `OUTPUT_STATE_PENDING` or blocked mutation queues), waiting for a newer foundation model release leaves production reliability unchanged. | **Enforce Deterministic Precondition Gates & Context Hygiene First**: Bound tool-output context noise (`<400-token` digests via `ADR-003`) and codify explicit state-machine precondition checks in the harness before blaming model reasoning. |
| Accelerate structured tool calls via server-side slot-parallel speculative decoding (`arXiv:2610.07086`) | Managed cloud model endpoints do not expose custom syntax-tree speculative decoding masks to client applications. | **Flatten Client-Side Pydantic Tool Contracts**: Achieve equivalent latency reduction on managed endpoints by eliminating deeply nested sub-objects, stripping optional telemetry metadata fields from tool input schemas, and separating reasoning from schema serialization. |

---

## 3. Concrete Distributed Systems Pattern: Precondition State-Machine Gates & Asymmetric Orphan Cleanup

Production control planes reinforce the core thesis of `arXiv:2610.01618`—autonomous remediation agents fail when they invoke migration or cleanup tools without respecting **state-machine preconditions** and **asymmetric resource teardown rules**:

1. **Strict 3-Step Precondition Sequence for Failed Pipelines**:
   - When a distributed training pipeline enters `OperationalState == FAILURE`, the execution guard (`ShouldSkipPipelineForExecutingMutation`) rejects direct queue flushes (`CleanUpMutations`) or cell migrations (`MigratePipeline`).
   - An autonomous remediation harness must enforce the exact precondition transition graph:
     1. Clear the terminal error state (`CleanUpFailedMutation` with `remove_failed_mutation_from_queue: true`);
     2. Flush stale queued triggers (`CleanUpMutations`);
     3. Execute cross-cell migration (`MigratePipeline`).
2. **Asymmetric Orphan Worker Pool Teardown**:
   - Control-plane cancellation hooks (`CancelPipelineJobs`) frequently auto-terminate old worker pools only for decoupled asynchronous data-generation pipelines (`is_async_pipeline: true`), while leaving coupled multi-task worker pools (`featurejoinv2.*`) running as orphans in the old cluster after migration.
   - Remediation harnesses must include a post-migration **Orphan Resource Verification Gate** rather than assuming `MigratePipeline == Clean Teardown`.
3. **Long-Tail Fleet Verification Gate**:
   - In multi-region/multi-country incidents (e.g., 23 impacted pipelines across 37 country projects), resolving the top 6 tier-1 markets often triggers premature incident closure while 17 long-tail pipelines remain stalled. Automated verification scripts must audit the 100% fleet tail before marking an incident resolved.
