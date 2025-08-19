# KB: Architecture Primer v1
Purpose: Provide shared definitions and mental models for the core concerns used in RepoArena rulepacks. This KB is for explanations and rationales; it is not used as scoring evidence.

1) Pluggable architecture
- Definition: A system that discovers, registers, and invokes extensions at runtime without changing core code.
- Cues in code:
  - Central registry or plugin manager; factories/strategies; interface- or contract-based loading.
  - Dynamic discovery via entrypoints, import hooks, or file/module scanning.
- Anti-patterns:
  - Hard-coded switch/case wiring; no extension points; plugin mentions only in docs.
- Typical evidence:
  - Registry/loader modules; factory methods; extension interfaces; dynamic imports/entrypoints.

2) Session/context management
- Definition: Capturing request-scoped state and propagating it across async boundaries safely.
- Cues in code:
  - TS/JS: AsyncLocalStorage, middleware attaching req.session/res.locals.
  - Python: contextvars, Flask/Django request context, FastAPI dependencies.
  - Go: context.Context passed through call chains.
  - Rust: extractors/state in Axum/Tower, Send + Sync correctness notes.
- Anti-patterns:
  - Global mutable state used for request data; context dropped or recreated ad hoc.

3) Concurrency orchestration
- Definition: Coordinating concurrent tasks with safe cancellation, backpressure, and composition.
- Cues in code:
  - TS/JS: Promise.all/any with AbortController; worker_threads if applicable.
  - Python: asyncio.gather with timeouts/cancellation; structured concurrency libraries.
  - Go: goroutines + channels + select with contexts for cancellation.
  - Rust: tokio::spawn/join!, structured cancellation patterns.
- Anti-patterns:
  - Fire-and-forget without cancellation; shared mutable state without guards.

4) SDLC integration (CI/CD + repo hygiene)
- Definition: Automated workflows, protections, and quality gates integrated into development.
- Cues in repo:
  - .github/workflows with matrix testing, CodeQL, releases; branch protection with required checks; Dependabot.
- Anti-patterns:
  - No workflows; flaky or non-required checks; lack of code scanning/security gates.

Notes
- Use these definitions for explanatory sections and rationales in reports.
- Scoring still requires code/workflow evidence; this KB does not count as evidence.

5) Performance and reliability
- Parallelization: I fan out searches across repos/criteria when possible.
- Scoping: Restrict languages/paths to reduce noise and run time.
- Rate limits: I back off automatically; if persistent, I return partials and propose scope reductions.
- Large repos/monorepos: Prefer path scoping (src/, packages/*) and specific criteria.