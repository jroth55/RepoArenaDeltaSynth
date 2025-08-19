# KB: Evidence JSON Schema
Purpose: Define the structure of evidence.json produced by RepoArena so you can audit, post-process, or integrate with other systems.

Top-level
- run_id: string — Unique identifier for this Arena run.
- generated_at: ISO datetime
- rulepack:
  - name: string
  - version: string
- cohort: array of objects
  - repo: "owner/name"
  - sha: string (40-char)
  - branch: string (optional if from default branch)
- results: array of RepoResult

RepoResult
- repo: "owner/name"
- sha: string
- totals:
  - must_score: number
  - should_score: number
  - overall: number
- criteria: array of CriterionResult

CriterionResult
- criterion_id: string
- title: string
- result: "PASS" | "PARTIAL" | "FAIL" | "NV"
- rationale: string
- citations: array of Citation
- metrics (optional): object of detector-specific counts

Citation
- citation_id: string (stable within this run)
- file: string (repo-relative path)
- lines: string (e.g., "12-39")
- url: string (SHA-pinned blob URL with #Lx-Ly)
- detector: string (e.g., "content_any:registry", "language_structural:go:select")
- confidence: "high" | "medium" | "low"

Tie-breakers (optional)
- pr_activity_90d: number (used only for ranking tie-breakers)
- distinct_citations: number
- must_score: number (redundant with totals.must_score)

Notes
- NV indicates Not Verifiable given current environment (e.g., runtime-only behavior, inaccessible repo).
- Only code/workflows/CI evidence should be used for PASS; docs/PRs may support PARTIAL rationales.
- Citation IDs are the linkage used by DeltaSynth; do not alter them if you post-process.

## Scoring Engine Implementation
This section has moved to KB.ScoringEngine.md.