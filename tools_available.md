# Tools Available

This document lists all tools available in this session, grouped by namespace, with purpose, parameters, and usage notes.

## Namespace: functions

- Tool: semantic-code-search
  - Purpose: Semantic search over a GitHub repository to find code relevant to a concept or idea.
  - Parameters:
    - query (string): A full sentence containing the user's original question.
    - repoName (string): Repository name.
    - repoOwner (string): Repository owner.
  - Usage notes:
    - MUST use when the prompt asks about a concept/idea in a specific repo.
    - MUST pass a complete sentence in `query`.

- Tool: lexical-code-search
  - Purpose: Lexical (exact match/pattern) code search for precise terms, paths, symbols, or regex.
  - Parameters:
    - query (string): Optimized query (may include qualifiers: content:, symbol:, path:, org:, repo:, language:, boolean operators, and regex in /slashes/).
    - scopingQuery (string, optional): Additional scope qualifiers.
  - Usage notes:
    - ONLY use when an exact word/phrase/path/symbol match is appropriate.
    - For regex path queries, surround with slashes and escape `/` in paths.

- Tool: support-search
  - Purpose: Answer GitHub product/support questions (Actions, auth, PR practices, Pages, Packages, Discussions, Copilot Spaces, etc.).
  - Parameters:
    - rawUserQuery (string): The latest raw, unedited user message.
  - Usage notes:
    - Not for repository-specific code questions.

- Tool: bing-search
  - Purpose: Web search (Bing) for recent or frequently updated information, or when explicitly requested.
  - Parameters:
    - freshness (string, optional): "", "month", "week", "day", or a date range.
    - query (string): Optimized web query (use site: operators when applicable).
    - user_prompt (string): A concise, standalone question for the answering model.
  - Usage notes:
    - Preserve citations exactly as returned when presenting results.

- Tool: githubwrite
  - Purpose: Perform write operations on GitHub (create/update files, branches; push changes; merge PRs; etc.).
  - Parameters:
    - query (string): A complete instruction including owner/repo, branch, target resource, and content when applicable.
  - Usage notes:
    - Include exact contents and blob SHA when updating existing files (if available).
    - Use this instead of non-existent helpers like create_or_update_file or push_files.

- Tool: github-coding-agent
  - Purpose: Create a new pull request to implement code changes based on a problem statement.
  - Parameters:
    - base_ref (string, optional): Base branch.
    - problem_statement (string, required): Detailed, actionable problem/task (Markdown supported).
    - problem_title (string, optional): Concise PR title.
    - repository (string, required): "owner/repo".
  - Usage notes:
    - Do NOT use unless explicitly asked to open a PR.

- Tool: githubread
  - Purpose: Read/query GitHub data (repos, files, PRs, issues, alerts, discussions, diffs, etc.).
  - Parameters:
    - query (string): The user's request as a complete sentence.
  - Usage notes:
    - Use whenever asked to retrieve any GitHub data.

- Tool: github-draft-issue
  - Purpose: Draft GitHub issues (single or multiple), including template support and metadata.
  - Parameters:
    - repository (string, optional): "owner/repo" if specified.
  - Usage notes:
    - MUST be used whenever the user asks to draft/create/generate issues or modify existing drafts.

- Tool: get_me
  - Purpose: Get details of the authenticated GitHub user.
  - Parameters: none
  - Usage notes:
    - Use for requests about the user's own profile or when info is needed to build other calls.

- Tool: search_users
  - Purpose: Find GitHub users by username, name, or profile info.
  - Parameters:
    - order (string, optional)
    - page (number, optional)
    - perPage (number, optional)
    - query (string): Search query (scoped to type:user).
    - sort (string, optional)

## Namespace: multi_tool_use

- Tool: parallel
  - Purpose: Wrapper to execute multiple functions tools in parallel when they can operate independently.
  - Parameters:
    - tool_uses (array): List of { recipient_name: "functions.<tool_name>", parameters: <object> }.
  - Usage notes:
    - Only include tools from the functions namespace.

## Notes on non-existent tools
- The following names were referenced previously but are NOT available in this environment: create_or_update_file, create_branch, push_files.
- Use githubwrite for write operations instead.