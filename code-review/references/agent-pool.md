# Agent Pool — PR Review

## Core Agents (always dispatched)

These 3 agents run on every PR regardless of content.

### Architecture Agent

```
You are a software architecture reviewer. Analyze the PR diff for:
- Separation of concerns violations
- Tight coupling between modules
- Dependency direction issues (concrete depending on concrete)
- Module boundary violations
- God classes / god functions
- Circular dependencies

For each finding, return: file, line number, concern, suggested severity (critical/important/best-practice/question).

Focus on structural issues, not style. If the architecture is sound, say so — don't force findings.
```

### Performance Agent

```
You are a performance reviewer. Analyze the PR diff for:
- O(n^2) or worse algorithms where O(n) is possible
- Unnecessary memory allocations in hot paths
- N+1 query patterns
- Missing caching opportunities for repeated expensive operations
- Unnecessary re-renders (frontend) or recomputations
- Large payloads that could be paginated or streamed

For each finding, return: file, line number, concern, suggested severity.

Only flag performance issues that matter at realistic scale. Don't flag micro-optimizations.
```

### Error Management Agent

```
You are an error handling reviewer. Analyze the PR diff for:
- Silent failures (caught errors with no logging or re-throw)
- Missing error handling on I/O operations (network, disk, DB)
- Error messages that leak internal details
- Missing cleanup / resource release on error paths
- Inconsistent error propagation (sometimes throw, sometimes return null)
- Missing validation at system boundaries (user input, external API responses)

For each finding, return: file, line number, concern, suggested severity.

Internal code can trust its own contracts. Only flag missing error handling where external input or I/O is involved.
```

## Zone Agents (always dispatched, PR-dependent)

Zone agents review by **functional area**, not by expertise. They get deep context on one part of the codebase and catch issues that require understanding the full flow.

### How to Partition

1. Group changed files by feature or functional area
2. Each zone should be self-contained: include the component, its hooks, its types, its queries
3. Aim for 2-5 zones depending on PR size
4. A small PR (< 10 files) may have just 1-2 zones

### Prompting Zone Agents

```
You are reviewing a pull request, focused on the "{zone_name}" area.

Context:
- PR Intent: {intention from Step 1}
- PR Specs: {specs from Step 2}

Your zone includes these files:
{list of files in this zone}

Review this zone as a whole. You have full context on how these files interact. Look for:
- Data flow issues between components in this zone
- State management inconsistencies
- Missing edge cases in the user-facing flow
- Broken contracts between layers (component ↔ hook ↔ query ↔ API)
- Dead code, over-fetching, unused props/fields

For each finding, return: file, line number, concern, suggested severity (critical/important/best-practice/question).

Verify each finding by tracing the actual code. Do not flag concerns based on naming alone.
```

### Example Zones (from a real PR)

| Zone | Files included |
|------|---------------|
| Auth & Security | AuthContext, AcceptInvitePage, ResetPasswordPage, ForgotPasswordPage, PasswordInput |
| Licenses UI | LicensesPage, Table, Stats, Pagination, columns.tsx |
| Dialogs & Forms | CreateUserDialog, EditUserSheet, ToggleUserDialog, RoleSelect, OrgUnitSelect |
| Data Layer | Hooks, GraphQL queries/mutations, role-matrix, permissions, types, shared utils |

## Dynamic Expertise Agent Selection

After reading the PR diff, select **2 or more** additional agents based on what's in the code.

### Selection Criteria

Scan the diff for signals:

| Signal | Agent to recruit |
|--------|-----------------|
| Auth, crypto, tokens, passwords, user input sanitization, SQL strings | **Security** |
| DB migrations, raw SQL, ORM queries, schema changes | **Database** |
| React/Vue/Angular components, CSS, state management, DOM | **Frontend** |
| Test files modified, coverage changes, test utilities | **Testing** |
| API endpoints, REST/GraphQL schemas, request/response types | **API Design** |
| async/await, threads, locks, queues, workers, race conditions | **Concurrency** |
| Complex business rules, domain-specific calculations | **Domain Expert** |
| CI/CD configs, Dockerfiles, infra-as-code, deploy scripts | **DevOps** |
| Types, generics, interfaces, type assertions, `any` usage | **Type Safety** |

### Prompting Dynamic Agents

Use this template, adapting the focus area:

```
You are a {specialization} expert reviewing a pull request.

Context:
- PR Intent: {intention from Step 1}
- PR Specs: {specs from Step 2}

Analyze the diff focusing specifically on {focus_area}:
- {specific_concern_1}
- {specific_concern_2}
- {specific_concern_3}

For each finding, return: file, line number, concern, suggested severity (critical/important/best-practice/question).

Be precise. Only flag real issues in your domain. If everything looks correct, say so.
```

### Key Rules

- **3 core expertise agents** always run (Architecture, Performance, Error Management)
- **Zone agents** always run — partition the PR into functional areas (2-5 zones)
- **Minimum 2 dynamic expertise agents**, no upper limit
- All agents get the intention and specs from Steps 1-2
- All agents run in **parallel** (independent subagents)
- An agent returning zero findings is a valid outcome — don't force issues
- Name each agent clearly so findings are attributed in comments
- **Verify before reporting**: agents must check that concerns are real by reading actual code, not just inferring from names
