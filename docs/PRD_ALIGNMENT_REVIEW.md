# Alignment with the original project document

Review date: 2026-10-31. Authority: `Document/Project Document.docx` v1.0, plus explicit user instructions. This initial review identified requirement/implementation distinctions before the subsequent Phase 2 work. Current acceptance is recorded in PHASE_2_EVIDENCE.md; the rows below describe the initial review state.

## Corrections made

PRD §4.2 permits hosted or open-source model APIs; §8.1 permits any hosted API or local open-source model capable of grounded generation. The final guidance likewise says training a language model from scratch is unnecessary. Ollama is absent from the original document. Revised ADR-017 reopens provider selection; architecture, task descriptions and README now reflect that permission. Earlier evidence about an unavailable Ollama service remains a historical observation, not a project-wide dependency or a requirement to install it.

The repository is no longer documentation-only. Status text has been corrected without changing historical test results. No task has been marked complete through this review.

## Requirement-to-build review

| PRD scope | Required outcome | Current implementation and remaining work |
|---|---|---|
| §§4.3,8,9.3 | One selected stack and Track A client | FastAPI/PostgreSQL/React remain selected by explicit user instruction. No stack replacement is needed. |
| §§6.1,10.1 | Authenticated room creation, invitations, membership and roles | First-slice acceptance exists. Later feature-specific role/isolation cases remain required. |
| §6.1 | Day-by-day board, moves, locks, lifecycle and visible concurrent conflict | Pin/move/remove/lock/unlock paths and targeted tests exist. Full grid, Voting integration and complete concurrency acceptance remain open. |
| §6.1 | Presence, quality, reconnect and deterministic Leader absence | Basic reconnect exists. Presence, connection quality, Leader absence and moderation/transfer remain open. |
| §6.1 | Durable chat, history, typing and reactions | Implemented with targeted database/browser evidence; complete role/isolation/failure acceptance remains open. |
| §§6.2,7.3.3 | Constraint-valid grounded suggestions, sources and actual generation | Standalone matcher exists. Authorized trip-context resolution, retrieval/model adapter, API/client and actual model evidence remain open. |
| §6.2 | Conflict detection, timed votes and consistent winner application | Remain required; scheduled in Phase 3. No manual lock operation substitutes for contested voting. |
| §§1.3,3,13 | Exact budget splits and running totals | Initial even-price summaries exist. Full exact even/weighted allocations and vote reservations remain required. |
| §6.3 | Dashboard, hybrid recommendations, analytics and Admin ingestion | Remain required; scheduled in Phase 3. Initial CLI import is not the Admin interface. |
| §§9.4,10.3 | Authoritative memory, persistence and recoverable state | Batch persistence and epoch primitives exist. Complete outage/crash/uncertain-commit acceptance remains open. |
| §§10–14 | Security, performance, tests, reproducible setup and submission | Targeted evidence exists. Complete acceptance, measurements, demo and retrospective remain required. |

## Explicit choices and provisional rules

The PRD's LWW language in §6.1 differs from the user's explicit compare-and-reject instruction. Preserve the user's instruction and ADR-004; do not silently change the implementation to LWW. Account Admin versus trip membership, vote details, Leader absence behavior, currency conventions, buffer direction and normalization remain documented choices or provisional interpretations, rather than quoted PRD requirements. Their status is maintained in DECISIONS.md.

Phase 2 remains TASK-008–013. The next implementation priorities are presence/moderation, unfinished board and recovery acceptance, chat security/failure coverage, then authorized matcher/retrieval/model/API/client integration. Model selection must be documented and validated before its real-answer acceptance; it does not block unrelated board or room work. All required later epics stay in final scope.

## Subsequent Phase 2 implementation

The user chose local open-source inference. Qwen2.5 1.5B through portable Ollama is the selected implementation, without turning Ollama into an original PRD requirement. The board/presence/moderation/chat/recovery and grounded Q&A paths are now implemented and verified against the Week 2 checkpoint in PRD §11.1. Actual results and limits are in PHASE_2_EVIDENCE.md. Week 3 explicitly owns live contested voting, exact budgets, conflicts, personalization/dashboard and Admin work; those required features have not been removed or claimed as finished.

## Current submission review — 2026-11-03

PRD §§12 and 15 require reproducible evidence and actual performance measurements against the targets in §10.2. TASK-001–025 and side quests SQ-001–003 are accepted for delivered features and artifacts. The verification record includes 117 backend passing tests and 8 frontend passing unit tests, complete application source code, two-context browser walkthroughs, database backup/restore verification, and clean submission packaging. See [traceability](TRACEABILITY_MATRIX.md), [Phase 4 evidence](PHASE_4_EVIDENCE.md), and [synchronization performance evidence](SYNC_PERFORMANCE_EVIDENCE.md).

| PRD criterion | Recorded result and boundary |
|---|---|
| §§1.3,10.2 itinerary sync | Initial Phase 4 baseline: 1 out of 1,000 moves peaked at 144.23 ms during concurrent restore load (P95: 25.17 ms). After canvas memoization by (day, hour) slot (`frontend/src/board/BoardGrid.jsx`) and lightweight room maintenance checks (`backend/app/voting/service.py`), the verified follow-up across 1,000 moves achieved **P95: 12.39 ms, max: 27.69 ms, with 0 samples exceeding 100 ms**. Full Chrome DOM + two-frame render across 200 moves achieved **P95: 59.90 ms, max: 96.40 ms (0 samples ≥ 100 ms)**. The <100 ms target is met. |
| §10.2 non-AI latency | Pooled HTTP P95 was 63.59 ms, well below the 200 ms target for all sampled routes. |
| §10.2 AI latency | Warm HTTP P95 was 1.60 s, with 99 grounded answers in 100 attempts (< 2.5 s target). Cold start returns a labeled grounded fallback. One browser-rendered sample took 3.197 s. Browser-rendered P95 is unverified. |
| §10.2 fan-out | Dispatch-to-last-apply peaked at 27.69 ms in the optimized follow-up run across 1,000 moves. Server-to-socket fan-out operates well within the 500 ms bound. Full design capacity of 50 simultaneous planners remains unverified. |
| §§1.3,10.2,14.1 scale | Trip creation supports a participant limit of 50. The load test used ten sockets across five identities and two rooms. Section 14.1 explicitly permits a smaller reported load test for submission. Performance with 50 distinct planners remains unverified. |

The move synchronization target (<100 ms) is proven satisfied under the optimized canvas and maintenance implementation. All raw performance sample datasets (`samples.json`, `server-profile.json`, `summary.json`), test logs, and application source code are included in the repository and submission package.
