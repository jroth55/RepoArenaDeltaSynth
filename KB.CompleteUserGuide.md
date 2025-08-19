# KB: Complete User Guide
A comprehensive guide to using the RepoArena + DeltaSynth Space. Covers concepts, workflows, evidence model, CI, and troubleshooting.

1) Overview and scope
- Purpose: Evidence-based repo comparison and synthesis (unique traits + merge plans).
- Outputs: Markdown report and evidence JSON with SHA-pinned file:line citations.
- Modes: Natural-language compare, template-driven runs, or custom rulepacks.

2) Core concepts
- Cohort: The list of repos (optionally pinned at SHAs) to compare.
- Rulepack: YAML criteria defining what "good" looks like (must/should, signals, thresholds).
- Criterion: A single scored check (PASS/PARTIAL/FAIL/NV) with citations.
- Signals: Detectors (path regex, content keywords, language-structural, github_api, etc.).
- Thresholds: min_citations, min_distinct_files, min_distinct_detectors to earn PASS.
- Weights: "must" vs "should" weighting for overall rank.
- Tie-breakers: Rank rules when totals tie (e.g., must-score → distinct citations → PR activity).
- Evidence: File:line citations pinned to SHAs; each with detector and confidence.
- Run ID: Unique identifier to reference a specific Arena run.

3) Standard workflow
- Plan: You describe goals; I propose a short execution plan for approval.
- Inputs: Repos + SHAs (or "resolve defaults"), rulepack (or I draft one), scope filters.
- Arena (analysis): Collect and verify evidence, score criteria, produce ranked results.
- DeltaSynth (synthesis): Unique Traits per repo and a Merge Blueprint with citations.
- Export: Commit artifacts via PR or generate a CI workflow to automate future runs.

4) Inputs, defaults, and fallbacks
- SHAs: If omitted, I resolve default branches and display the resolved SHAs.
- Rulepack: If none provided, I draft one from your intent and ask for approval.
- Exclusions: tests/examples/fixtures excluded by default (can be overridden).
- Scope: You can limit by languages and paths (e.g., TypeScript under src/).
- Reuse: "Use last run" or "use run_id:<uuid>" to reference previous evidence.

5) How I search and verify (tool mapping)
- Concepts (e.g., "plugin registry", "agent router"): semantic search → confirm with lexical search → open files.
- Exact APIs/files (Promise.all, tokio::spawn): lexical with language/path filters → open files.
- CI/branch rules: GitHub reads for .github/workflows and branch protection settings.
- Always verify by opening matched files to cite precise line ranges.

6) Evidence model and scoring
- Results per criterion: PASS | PARTIAL | FAIL | NV (Not Verifiable).
- PASS requires meeting thresholds with code/workflow evidence; docs/PRs support rationale, not PASS alone.
- Citations carry: file path, line span, SHA-pinned URL, detector, confidence.
- Scoring defaults: pass=1.0, partial=0.5, fail=0.0; weights must=3, should=1 (tweakable).
- NV cases: runtime-only behavior, inaccessible repos, or insufficient evidence.

7) Knowledge Base (RAG) usage
- Reference sources by title: "Use KB: Architecture Primer v1".
- KB informs explanations and rationale; it does not count as scoring evidence.
- Reports include a Sources section listing KB items when used.

8) Re-runs and partial runs
- Re-run last: "Re-run Arena on the last cohort and rulepack; same weights/thresholds."
- Partial re-run: "Re-run only criterion <id> for owner/repoX (same SHA)."
- Pinning: Use run_id for deterministic follow-ups (e.g., DeltaSynth, exports).
- Diffing: I merge fresh citations into prior evidence and update totals.

9) Performance and reliability
- Parallelization: I fan out searches across repos/criteria when possible.
- Scoping: Restrict languages/paths to reduce noise and run time.
- Rate limits: I back off automatically; if persistent, I return partials and propose scope reductions.
- Large repos/monorepos: Prefer path scoping (src/, packages/*) and specific criteria.

10) Privacy and permissions
- Read-only by default; I only write when you ask (commits/PRs/workflows).
- Access is limited to your GitHub permissions; SSO/private repo blocks are reported.
- No source leaves this session unless you export artifacts or ask me to push.

11) CI/CD integration
- Publish artifacts by PR:
  - "Commit Markdown + JSON to branch arena-report/<date> in owner/repo and open a PR."
- Schedule via GitHub Actions (see KB: CI Workflow):
  - Resolve SHAs, run Arena on a matrix of repos, upload artifacts, optionally publish to Pages.
  - Recommended settings: pinned action versions, caching, minimal permissions (contents: write for PR, pages: write if publishing), cron schedule, concurrency group to avoid overlap.

12) Troubleshooting guide
- Too many false positives:
  - Tighten signals; add negatives for tests/examples/fixtures.
  - Increase min_citations and require min_distinct_files/detectors.
  - Narrow languages/paths; prefer language_structural over broad keywords.
- Not enough signal:
  - Add synonyms across languages (e.g., registry, plugin, extension, entrypoint).
  - Lower thresholds one step temporarily to inspect candidates; then restore.
  - Include structural hints (factory/strategy/interface registry).
- Long runtimes or rate limits:
  - Reduce cohort size or criteria; run in batches and aggregate.
  - Limit search scope to key directories (src/, pkg/, core/).
  - I will back off and return partials with suggestions automatically.
- Private/SSO repo access:
  - Ensure your account has access and SSO authorization; I'll proceed with accessible repos and mark blocked ones NV.
- YAML validation errors (custom rulepacks):
  - Ask: "Validate this rulepack and show which signals matched before running."
  - Fix schema (ids unique, must/should specified, thresholds numeric).
- Evidence audit:
  - Ask: "Return the raw evidence JSON for the last run, including citation_ids."
  - Request "spot-open" of any citation to view exact code lines.
- Plan rejection workflow:
  - If you reject my proposed plan, I'll ask for specific adjustments
  - Common modifications: scope reduction, different tools, alternative criteria
  - I'll re-propose until you approve or provide direct instructions

13) Best practices
- Start natural language; accept the plan; iterate quickly.
- Keep criteria small and precise; two to four signals per criterion.
- Require distinct files/detectors for must-have criteria.
- Always pin SHAs for reproducibility.
- Use DeltaSynth only after Arena to guarantee citations.

14) FAQ
- Do I need SHAs? Recommended; if omitted, I resolve defaults and display the resolved SHAs.
- Can I compare across orgs? Yes, subject to your access and rate limits.
- Can I bring my own rulepack? Yes; I can validate and run it.
- Can I re-use previous results? Yes; say "use last run" or "use run_id:<uuid>".
- What languages are supported? Any text-based repo; language_structural hints are available for TS/JS, Python, Go, Rust (extendable via keywords/regex).

15) Related KBs
- KB: Quick Start Templates — copy/paste prompts
- KB: Custom Rulepacks Guide — YAML anatomy, examples, best practices
- KB: User Journey Examples — end-to-end transcripts
- KB: Tool Integration Mapping — signal type to tool call translation
- KB: Scoring Engine — algorithm implementation details
- KB: DeltaSynth Algorithm — unique traits and merge blueprint generation
- KB: CI Workflow — ready-to-commit Action for scheduled Arena runs
- KB: Architecture Primer v1 — definitions and rationale
- Rulepack: Best-Architecture v2.1 — canonical reference implementation

Need a jumpstart? See KB: Quick Start Templates, paste a prompt, and I'll propose a plan for approval before running.