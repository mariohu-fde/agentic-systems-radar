# ADR-003: Zero-Bloat Tool Output Compaction via Out-of-Band SQLite-JSON1

- **Status**: Accepted
- **Date**: 2026-09-23

## Context
Enterprise diagnostic tools (Cloud Logging queries, `INFORMATION_SCHEMA` job metadata dumps, Kubernetes pod specs) routinely return 20KB–200KB JSON payloads. Injecting raw tool outputs directly into the LLM conversation history causes three production failures:
1. **Attention Dilution ("Lost in the Middle")**: Crucial error signatures are buried inside repetitive JSON boilerplate.
2. **Prefix KV Cache Busting & Cost Inflation**: Every subsequent turn re-processes tens of thousands of raw log tokens.
3. **Hallucinated Summaries**: Using a secondary LLM call to summarize tool outputs introduces latency and drops exact field values (`job_id`, `turn_id`, `quota_limit`) required for downstream verification.

## Decision
We implement a deterministic **`AfterToolCallback` Compaction Middleware (`SQLiteToolCompactor`)**:
1. Any JSON tool output exceeding `1,500` characters is intercepted before entering the model transcript.
2. The complete lossless JSON payload is written to an out-of-band in-memory **SQLite `json_each` store** keyed by an immutable `payload_ref`.
3. The LLM receives a bounded `<400-token` **Schema Skeleton & Sample Digest** plus the `payload_ref` handle, allowing on-demand SQL/JSONPath slice retrieval without context bloat.

## Consequences
- **Positive**: 95%+ reduction in tool-output token residency; zero loss of exact evidence fields; zero LLM summarization latency.
- **Negative / Boundary**: Adds one extra tool-call round-trip when the initial 3-row sample does not contain the target anomaly. Short outputs (`<=1,500` chars) bypass interception to avoid unnecessary indirection.
