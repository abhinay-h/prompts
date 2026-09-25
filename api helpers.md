Here is the **reusable feature-wise prompt template** tailored to the repository's rules and the **Standard Implementation Protocol**. 

Whenever you are ready to migrate a feature, copy the template below, replace `[FEATURE_NAME]` (and optional file hints), and run it.

---

### Reusable Feature-Wise Prompt Template

```markdown
Implement Stage 5 (Extract shared API client helper logic) for the following feature:

TARGET FEATURE: [FEATURE_NAME]

Follow the "Standard Implementation Protocol" and rules in .codex/agents.md, .codex/repo-knowledge.md, and .codex/shared-common-refactor.md:

1. ARCHITECTURAL BOUNDARIES (Zero Breakages):
   - DO NOT merge web/mobile goFetch implementations. They intentionally remain separate clients.
   - Allowed in shared/common:
     * endpoint URL builders
     * query parameter builders (arrays, booleans, undefined filtering)
     * request payload builders & DTO transformers
     * response normalizers (array fallback, null coalescing, envelope unwrap)
     * error mappers & status classifiers
     * upload path/policy builders
   - Strictly FORBIDDEN in shared/common:
     * goFetch implementations
     * cookies, headers mutation, SecureStore, AsyncStorage
     * token refresh, login/session lifecycle
     * navigation (next/navigation, expo-router)
     * toasts, alerts, UI components
     * React hooks & TanStack Query hooks
     * Any change in API behavior or payload shapes

2. EXECUTION STEPS:
   a. Audit web API/query files and mobile API files for [FEATURE_NAME].
   b. Identify duplicate endpoint strings, query params, payload builders, normalizers, and error mappers.
   c. Create/update `shared/common/[FEATURE_NAME]/api.ts` (pure platform-neutral code only).
   d. Update web callers to import from `@prepdha/shared/common/[FEATURE_NAME]/api` (or re-export through `shared/web-mobile/[FEATURE_NAME]/`).
   e. Update mobile callers to import from `@shared-common/common/[FEATURE_NAME]/api`.
   f. Preserve exact runtime behavior and call signatures.

3. VALIDATION:
   - Web TypeScript: `npm run type-check`
   - Mobile TypeScript: `npm --prefix mobile run type-check`
   - AST Knowledge Graph: `python -m graphify update .`

4. COMMIT:
   - Git commit message: `refactor([FEATURE_NAME]): centralize shared api helpers`

5. SUMMARY:
   - Files created in shared/common/
   - Web files updated
   - Mobile files updated
   - Validations run and results
   - Risks/platform-specific logic safely preserved
```

---

### Example: Running Feature 1 (`assignments`)

To execute the first feature, you can submit:

```markdown
Implement Stage 5 (Extract shared API client helper logic) for the following feature:

TARGET FEATURE: assignments

Relevant files:
- Web: lib/query/teacher-assignments-queries.ts, components/student/assignments/*
- Mobile: mobile/features/assignments/api/assignments.api.ts, mobile/features/assignments/api/student-upload.api.ts, mobile/features/teacher-dashboard/api/teacher-dashboard.api.ts

Follow the "Standard Implementation Protocol" and rules in .codex/agents.md, .codex/repo-knowledge.md, and .codex/shared-common-refactor.md:

1. ARCHITECTURAL BOUNDARIES (Zero Breakages):
   - DO NOT merge web/mobile goFetch implementations.
   - Only platform-neutral endpoint builders, query param builders, payload mappers, normalizers, and error mappers in shared/common/assignments/api.ts.
   - Do not move cookies, AsyncStorage, tokens, or TanStack query hooks.

2. EXECUTION STEPS:
   a. Audit web and mobile assignments files.
   b. Create shared/common/assignments/api.ts with pure helpers.
   c. Update web and mobile imports.
   d. Preserve exact behavior.

3. VALIDATION:
   - `npm run type-check`
   - `npm --prefix mobile run type-check`
   - `python -m graphify update .`

4. COMMIT:
   - `refactor(assignments): centralize shared api helpers`

5. Provide the final summary.
```

---

### Quick Reference: Feature Target Names

You can plug any of these into `TARGET FEATURE: [FEATURE_NAME]`:

| # | Core Sequence (1–10) | # | Extended Features (11–26) |
|---|---|---|---|
| 1 | `assignments` | 11 | `auth` |
| 2 | `teacher-dashboard` | 12 | `calendar` |
| 3 | `parent-dashboard` | 13 | `cognitive-exercises` |
| 4 | `teacher-notes` | 14 | `student-dashboard` |
| 5 | `student-learning` | 15 | `dharani` |
| 6 | `flashcard` | 16 | `erp` |
| 7 | `practice` | 17 | `fees` |
| 8 | `storage` | 18 | `guardian` |
| 9 | `notifications` | 19 | `issue-report` |
| 10 | `shop` | 20 | `labs` |
| | | 21 | `revision-notes` |
| | | 22 | `search` |
| | | 23 | `staff-attendance` |
| | | 24 | `teacher-tests` |
| | | 25 | `timetable` |
| | | 26 | `todo` |

The detailed file breakdown and extracted function specifications for all 26 features are documented in [stage5_shared_api_helpers_plan.md](file:///C:/Users/rabhi/.gemini/antigravity-ide/brain/2f52216d-a4ff-4018-9dcf-b83d3a0a93b4/stage5_shared_api_helpers_plan.md).
