---
name: pr-review
description: Review a pull request or git diff for correctness, security, performance, testing, and maintainability issues. Focus on changed code and provide specific, prioritized feedback.
---

# PR Review

## When to use this skill

Use this skill when:
- Reviewing a pull request
- Reviewing a git diff before merge
- Checking changed code for bugs, regressions, or bad design
- Auditing a change for security or performance issues
- Reviewing unstaged, staged, or branch diff changes

Do not use this skill for:
- Full codebase audits unrelated to current changes
- Pure style-only review when formatters/linters should handle it
- Rewriting code without first identifying concrete review findings

## Review goal

Find the highest-value issues in the changed code first:
1. Correctness and regressions
2. Security risks
3. Data loss or broken behavior
4. Performance problems
5. Missing or weak tests
6. Maintainability issues

Prioritize signal over volume. Do not flood the author with low-value comments.

## Inputs to gather first

Before reviewing, gather as much of the following as possible:
- PR title and description
- Linked issue or task, if any
- Changed files
- Diff hunks
- Related tests
- Any screenshots, API examples, migration notes, or rollout notes

If there is no PR description, infer intent from the diff and say that you inferred it.

## Review process

### Step 1: Understand the change

Determine:
- What problem this change is solving
- Whether it is a feature, bugfix, refactor, migration, or cleanup
- Whether the scope matches the stated goal
- Whether tests were added or updated
- Whether the change is bigger than it needs to be

Questions to ask:
- What could break because of this?
- What assumptions does this code make?
- Does the implementation match the intent?

### Step 2: High-level design review

Check:
- Is the approach sensible?
- Is it consistent with existing patterns in the codebase?
- Is the code in the right module or layer?
- Is the abstraction level appropriate?
- Is this introducing unnecessary complexity?

Look for:
- Wrong ownership of logic
- Leaky abstractions
- Business logic in controllers/views/handlers
- Tight coupling
- Duplicate patterns that should be shared

### Step 3: Detailed code review

Inspect the changed lines closely.

#### Correctness
Check for:
- Broken logic
- Wrong conditionals
- Off-by-one errors
- Incorrect async handling
- Race conditions
- Nil/null/undefined cases
- Bad default values
- Bad error propagation
- Partial failure handling
- Backward compatibility issues

#### Code quality
Check for:
- Clear names
- Small focused functions
- Minimal side effects
- No dead code
- No commented-out junk
- No magic numbers
- No copy-paste duplication
- Consistent style with existing code

#### Error handling
Check for:
- Silent failures
- Swallowed exceptions
- Missing logging where operationally important
- Poor user-facing errors
- Missing retries, timeouts, or cleanup where needed

### Step 4: Security review

Check all changed surfaces for:
- Missing input validation
- Authorization gaps
- Authentication mistakes
- SQL injection
- XSS
- CSRF exposure where relevant
- Hardcoded secrets
- Unsafe deserialization
- Path traversal
- SSRF
- Command injection
- Insecure file handling
- Sensitive data leaks in logs
- Token/session/cookie mishandling

Do not call something a security issue unless you can explain the actual risk clearly.

### Step 5: Performance review

Check for:
- N+1 queries
- Unbounded loops
- Repeated expensive work
- Bad query patterns
- Missing pagination
- Large payload processing
- Excessive renders/recomputations
- Memory leaks
- Connections/files/resources not released
- Missing batching, caching, or indexing where clearly needed

Do not nitpick theoretical micro-optimizations unless they matter in the changed path.

### Step 6: Testing review

Check whether:
- New behavior is tested
- Regressions would be caught by existing tests
- Edge cases are covered
- Failure cases are covered
- Tests are deterministic
- Test names explain behavior
- Test setup is maintainable
- The change should have tests but does not

If tests are missing, say exactly what should be tested.

### Step 7: Documentation and rollout review

Check whether the change requires updates to:
- README
- API docs
- Config docs
- Migration notes
- Environment variable docs
- Feature flags
- Operational runbooks
- Changelog

Also check for rollout risk:
- Breaking changes
- Data migration concerns
- Required backfills
- Compatibility issues
- Deployment ordering issues

### Step 8: Write the review

Be direct, specific, and useful.
Do not give vague complaints.
Do not attack the author.
Do not pad the review with generic praise.

When raising an issue:
- Point to the exact code or behavior
- Explain why it is a problem
- State impact
- Suggest a fix or safer direction

## Severity levels

Use these severities:

### Critical
Use for:
- Security vulnerabilities
- Data corruption or data loss
- Major correctness bugs
- Broken auth or permissions
- Crash paths in common flow
- Highly likely production regressions

### High
Use for:
- Serious logic flaws
- Significant performance issues
- Missing handling for realistic failure cases
- Important missing tests for risky code

### Medium
Use for:
- Maintainability problems that make bugs likely
- Inconsistent patterns
- Weak error handling
- Incomplete edge case handling

### Low
Use for:
- Minor cleanup
- Readability improvements
- Non-blocking refactors
- Small documentation issues

## Output format

Always use this structure:

### Summary
- 1 to 4 bullets
- State what the change does
- State overall review outcome
- Mention biggest risks

### Findings

For each finding, use this format:

#### [Severity] Short title
- **Where:** file/path.ext[:line or function]
- **What:** clear description of the issue
- **Why it matters:** impact, failure mode, or exploitability
- **Suggested fix:** concrete guidance
- **Blocking:** yes/no

If there are no meaningful issues, say so plainly.

### Testing assessment
- What tests exist
- What is missing
- Whether current coverage is enough for the risk level

### Final verdict
Choose one:
- Approve
- Approve with non-blocking comments
- Request changes

Then give a short reason.

## Review rules

- Review the diff, not the whole universe
- Focus on changed code and directly affected areas
- Prefer correctness over style
- Prefer concrete findings over generic suggestions
- Do not invent hidden context
- If context is missing, state the assumption
- Do not flag hypothetical issues without explaining a realistic path
- Do not suggest major rewrites unless the current approach is genuinely risky
- Avoid duplicate comments for the same root cause

## What good feedback looks like

Good:
- "This authorization check happens in the UI, but the server handler still accepts the action without verifying ownership. A user could call the endpoint directly."

Bad:
- "Security issue here."

Good:
- "This loop performs one query per item, which will degrade badly for larger orders. Fetch the related rows in one query or batch by IDs."

Bad:
- "Performance could be better."

Good:
- "This branch returns success even if the write fails because the exception is logged but not rethrown. That can leave the client with a false success response."

Bad:
- "Error handling seems off."

## Common things to catch

### Logic
- Inverted conditions
- Wrong fallback values
- Missing null checks
- Async work not awaited
- State update ordering bugs
- Broken retry logic
- Missing transactions

### Security
- Trusting client input
- Missing server-side auth checks
- Raw SQL/string interpolation
- Rendering unsanitized HTML
- Secrets in code
- Logging tokens or PII

### Performance
- N+1 DB access
- Large in-memory processing
- Repeated parsing or serialization
- Unbounded scans
- Missing limits/pagination
- Wasteful network calls

### Maintainability
- Huge handlers
- Mixed responsibilities
- Deep nesting
- Copy-paste logic
- Hidden side effects
- Hardcoded behavior that should be config

## Optional tooling

Use automated tools where relevant, but do not replace reasoning with tool output.

Examples:
- JavaScript/TypeScript: eslint, prettier, tsc
- Python: ruff, pylint, pytest
- Go: gofmt, golangci-lint, go test
- Rust: clippy, rustfmt, cargo test
- Security: npm audit, Bandit, dependency scanning

Automated tools catch surface issues. This skill should focus on review judgment.

## Default review stance

Default to:
- concise
- high-signal
- exact
- blocking only when justified

Do not over-review. Catch what matters.
