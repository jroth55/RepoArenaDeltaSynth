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

- KB: Scoring Engine Implementation
How RepoArena actually works under the hood - the algorithm that turns tool results into scored evidence.

## Signal Detection Pipeline
1. **Query Construction**: Convert rulepack signals into tool queries
   - content_any → lexical-code-search with OR terms
   - language_structural → lexical-code-search with language: filters
   - structural_hint → semantic-code-search → confirm with lexical
   - path_regex → lexical-code-search with path: qualifier
   - github_api → githubread for workflows/settings

2. **Evidence Collection**: Execute searches, verify by opening files
   - Each match becomes a candidate citation
   - File opens verify context (not just keyword presence)
   - Extract precise line ranges for citations

3. **Threshold Evaluation**: Apply pass_threshold rules
   - Count citations by detector type for min_distinct_detectors
   - Count unique files for min_distinct_files  
   - Apply min_citations across all evidence

4. **Confidence Assignment**: Based on signal type and verification
   - High: Direct code evidence in primary paths
   - Medium: Config/workflow evidence, secondary paths
   - Low: Documentation mentions, generated files

## Scoring Logic
```
For each criterion:
  citations = collect_evidence(signals, repo, sha)
  if meets_threshold(citations, pass_threshold):
    result = PASS, score = 1.0
  elif has_partial_evidence(citations):
    result = PARTIAL, score = 0.5
  else:
    result = FAIL, score = 0.0

Total = sum(criterion_score * weight) for all criteria
```

## Tool Integration Details

### Signal Type Mapping
- **content_any**: `lexical-code-search` with OR'd terms
- **language_structural**: Multiple `lexical-code-search` calls per language
- **structural_hint**: `semantic-code-search` then lexical confirmation
- **path_regex**: `lexical-code-search` with path: qualifier
- **file_exists**: `githubread` to check file presence
- **github_api**: `githubread` for workflows, branch protection, etc.

### Evidence Verification Process
1. Get initial matches from appropriate tool
2. Use `githubread` to open each matched file
3. Verify context around matches (not just keyword presence)
4. Extract precise line ranges containing relevant code
5. Generate SHA-pinned URLs with line anchors
6. Assign confidence based on file type and context

### Threshold Logic
```python
def evaluate_criterion(citations, threshold):
    if not citations:
        return "FAIL"
    
    citation_count = len(citations)
    distinct_files = len(set(c.file for c in citations))
    distinct_detectors = len(set(c.detector for c in citations))
    
    meets_min_citations = citation_count >= threshold.min_citations
    meets_min_files = distinct_files >= threshold.get('min_distinct_files', 1)
    meets_min_detectors = distinct_detectors >= threshold.get('min_distinct_detectors', 1)
    
    if meets_min_citations and meets_min_files and meets_min_detectors:
        return "PASS"
    elif citation_count >= 1:  # Has some evidence
        return "PARTIAL"
    else:
        return "FAIL"
```

## Error Handling
- Tool timeouts/failures → NV for affected criteria
- Private/inaccessible repos → NV for entire repo
- Rate limits → backoff, partial results with explicit gaps noted
- No matches found → FAIL (distinguishable from NV)

## Confidence Assignment Rules
- **High**: Direct code in src/, lib/, core/ with verified context
- **Medium**: Config files, workflows, secondary paths
- **Low**: Comments, docs, generated files, test fixtures