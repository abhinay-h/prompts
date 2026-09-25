# Stage 5: Shared API Client Helper Logic — Master Implementation Plan

> **Architectural Objective**: Centralize platform-neutral API helpers (endpoint path builders, query parameter builders, request payload formatters, response normalizers, error mappers, pagination builders, and DTO transformers) under `shared/common/<feature>/api.ts` across Web and Mobile, without merging `goFetch` implementations or introducing platform leaks.

---

## 1. Active Role Modes & Guidelines

In accordance with [.codex/agents.md](file:///d:/PrepdhaSchool2/.codex/agents.md) and [.codex/shared-common-refactor.md](file:///d:/PrepdhaSchool2/.codex/shared-common-refactor.md):
- **Senior Frontend Engineer**: Guard Next.js query configurations, web API contracts, and browser component boundaries.
- **Senior Mobile Engineer**: Guard Expo/React Native API files, Metro resolution, and bundle footprint.
- **Principal Code Reviewer**: Audit invariant retention, verify zero runtime behavioral drifts, and enforce platform neutrality.
- **QA/Test Engineer**: Validate through dual-platform type checks (`npm run type-check` & `npm --prefix mobile run type-check`) and unit tests.
- **Security/Privacy Reviewer**: Enforce strict boundary rules on auth, tokens, student PII, and file upload policies.

---

## 2. Hard Architectural Invariants

### Strictly Allowed in `shared/common/<feature>/api.ts`
- **Endpoint URL builders**: Pure functions generating route strings (e.g., `teacherTasksApiPath()`, `studentAssignmentStatusApiPath(id)`).
- **Query param builders**: Pure functions that clean, cast, and stringify search params / query dicts (handling comma-joined IDs, pagination `page`/`limit`, filter IDs, dates).
- **Request payload builders & DTO transformers**: Clean shape mappers (e.g., mapping camelCase UI forms to backend snake_case JSON schemas like `student_ids`, `attachment_url`).
- **Response normalizers**: Pure safety wrappers (e.g., fallback empty array `Array.isArray(d) ? d : []`, unwrapping Go `{ ok, data }` envelopes, casting numbers).
- **Error mappers**: Pure functions extracting user-friendly error messages or classifying HTTP status codes (e.g., 401/403 access check).
- **Upload path & policy helpers**: Storage key generators and validation rules.

### Strictly Forbidden in `shared/common/`
- **NO `goFetch` merging**: Web (`shared/web-cms/lib/api/go-fetch.ts`) and Mobile (`mobile/core/api/go-fetch.ts`) must remain separate implementations.
- **NO storage or state**: No cookies, headers mutation, `AsyncStorage`, or `SecureStore`.
- **NO auth lifecycle**: No token refresh, session rotation, or login side-effects.
- **NO UI / Framework hooks**: No React components, React hooks (`useState`, `useEffect`), or TanStack Query hooks (`useQuery`, `useMutation`).
- **NO navigation or side-effects**: No `next/navigation`, `expo-router`, toasts, alerts, or logging libraries.
- **NO behavioral alterations**: Request URLs, HTTP methods, headers, and payloads must match backend expectations byte-for-byte.

---

## 3. Path & Module Resolution Matrix

| Consumer | Source Alias | Maps To Physical Location |
|---|---|---|
| **Web** (`tsconfig.json`) | `@prepdha/shared/common/*` or `@/shared/common/*` | `d:/PrepdhaSchool2/shared/common/*` |
| **Mobile** (`mobile/tsconfig.json`) | `@shared-common/common/*` | `d:/PrepdhaSchool2/shared/common/*` |
| **Mobile Metro** (`metro.config.js`) | `'@shared-common': path.resolve(__dirname, '../shared')` | `d:/PrepdhaSchool2/shared/*` |
| **Web-Mobile Re-export** (`shared/web-mobile/<feature>/api.ts`) | Optional proxy re-exporting `shared/common/<feature>/api.ts` | Eliminates broken legacy imports |

---

## 4. Part 1: Core Feature Sequence (1 - 10)

### 1. `assignments`
- **Web Files**:
  - [teacher-assignments-queries.ts](file:///d:/PrepdhaSchool2/lib/query/teacher-assignments-queries.ts)
  - [StudentAssignmentsList.tsx](file:///d:/PrepdhaSchool2/components/student/assignments/StudentAssignmentsList.tsx)
  - [AssignmentStatusButton.tsx](file:///d:/PrepdhaSchool2/components/student/assignments/AssignmentStatusButton.tsx)
- **Mobile Files**:
  - [assignments.api.ts](file:///d:/PrepdhaSchool2/mobile/features/assignments/api/assignments.api.ts)
  - [student-upload.api.ts](file:///d:/PrepdhaSchool2/mobile/features/assignments/api/student-upload.api.ts)
  - [teacher-dashboard.api.ts](file:///d:/PrepdhaSchool2/mobile/features/teacher-dashboard/api/teacher-dashboard.api.ts) (assignments section)
- **Extracted Shared Helpers** (`shared/common/assignments/api.ts`):
  - `teacherTasksApiPath()` $\rightarrow$ `"/teacher/tasks"`
  - `teacherTaskApiPath(taskId: number)` $\rightarrow$ `"/teacher/tasks/${taskId}"`
  - `teacherTaskAssignmentsApiPath(taskId: number)` $\rightarrow$ `"/teacher/tasks/${taskId}/assignments"`
  - `teacherClassSubjectsApiPath()` $\rightarrow$ `"/teacher/class-subjects"`
  - `teacherEligibleStudentsApiPath()` $\rightarrow$ `"/teacher/tasks/eligible-students"`
  - `teacherAssignmentCommentApiPath(assignmentId: number)` $\rightarrow$ `"/teacher/assignments/${assignmentId}/comment"`
  - `teacherTaskUploadContextApiPath()` $\rightarrow$ `"/teacher/tasks/upload-context"`
  - `studentAssignmentsApiPath()` $\rightarrow$ `"/student/assignments"`
  - `studentAssignmentsNewCountApiPath(since?: string | null)` $\rightarrow$ `since ? "/student/assignments/new-count?since=" + encodeURIComponent(since) : "/student/assignments/new-count"`
  - `studentAssignmentStatusApiPath(assignmentId: number)` $\rightarrow$ `"/student/assignments/${assignmentId}/status"`
  - `buildEligibleStudentsQueryParams(classSubjectId: number)` $\rightarrow$ `{ classSubjectId }`
  - `buildTeacherTaskPayload(draft: TeacherTaskDraft)` $\rightarrow$ `{ title, description, due_date, attachment_url, attachment_name, needs_attachment, class_subject_id, student_ids }`
  - `buildAssignmentFeedbackPayload(feedback: string)` $\rightarrow$ `{ feedback }`
  - `buildStudentAssignmentStatusPayload(complete: boolean, attachmentUrl?: string | null, attachmentName?: string | null)`
  - `normalizeTeacherTasks(data: unknown)` $\rightarrow$ `Array.isArray(data) ? data : []`
  - `normalizeTeacherTaskAssignments(data: unknown)` $\rightarrow$ `Array.isArray(data) ? data : []`
- **Commit**: `refactor(assignments): centralize shared api helpers`

---

### 2. `teacher-dashboard`
- **Web Files**:
  - [teacher-dashboard-queries.ts](file:///d:/PrepdhaSchool2/lib/query/teacher-dashboard-queries.ts)
  - [shared/web-mobile/teacher-dashboard/queries.ts](file:///d:/PrepdhaSchool2/shared/web-mobile/teacher-dashboard/queries.ts)
- **Mobile Files**:
  - [teacher-dashboard.api.ts](file:///d:/PrepdhaSchool2/mobile/features/teacher-dashboard/api/teacher-dashboard.api.ts)
  - [teacher-dashboard.queries.ts](file:///d:/PrepdhaSchool2/mobile/features/teacher-dashboard/api/teacher-dashboard.queries.ts)
- **Extracted Shared Helpers** (`shared/common/teacher-dashboard/api.ts`):
  - `teacherDashboardApiPath()` $\rightarrow$ `"/teacher/dashboard"`
  - `teacherDashboardCoreApiPath()` $\rightarrow$ `"/teacher/dashboard/core"`
  - `teacherDashboardInsightsApiPath()` $\rightarrow$ `"/teacher/dashboard/insights"`
  - `teacherStudentsApiPath()` $\rightarrow$ `"/teacher/students"`
  - `teacherStudentDetailApiPath(studentId: number)` $\rightarrow$ `"/teacher/students/${studentId}"`
  - `teacherReportsApiPath()` $\rightarrow$ `"/teacher/reports"`
  - `teacherProfileApiPath()` $\rightarrow$ `"/teacher/profile"`
  - `principalTeachersApiPath()` $\rightarrow$ `"/principal/teachers"`
  - `principalTeacherDetailApiPath(teacherId: number)` $\rightarrow$ `"/principal/teachers/${teacherId}"`
  - `principalTeacherDetailCoreApiPath(teacherId: number)` $\rightarrow$ `"/principal/teachers/${teacherId}/core"`
  - `principalTeacherDetailInsightsApiPath(teacherId: number)` $\rightarrow$ `"/principal/teachers/${teacherId}/insights"`
  - `teacherBooksApiPath()` $\rightarrow$ `"/teacher/books"`
  - `teacherBookTopicApiPath(topicId: number)` $\rightarrow$ `"/teacher/books/topics/${topicId}"`
  - `buildTeacherDashboardFilterParams(filter?: { classId?: number; subjectId?: number })`
  - `buildTeacherStudentsQueryParams(params: TeacherStudentsQueryInput)`
  - `buildPrincipalTeachersQueryParams(params: { q?: string; sort?: string })`
- **Commit**: `refactor(teacher-dashboard): centralize shared api helpers`

---

### 3. `parent-dashboard`
- **Web Files**:
  - [parent-dashboard-queries.ts](file:///d:/PrepdhaSchool2/lib/query/parent-dashboard-queries.ts)
  - [parent-dashboard-api.ts](file:///d:/PrepdhaSchool2/features/guardian/api/parent-dashboard-api.ts)
  - [shared/web-mobile/parent-dashboard/queries.ts](file:///d:/PrepdhaSchool2/shared/web-mobile/parent-dashboard/queries.ts)
- **Mobile Files**:
  - [parentDashboard.api.ts](file:///d:/PrepdhaSchool2/mobile/features/parent-dashboard/api/parentDashboard.api.ts)
  - [parentDashboard.queries.ts](file:///d:/PrepdhaSchool2/mobile/features/parent-dashboard/api/parentDashboard.queries.ts)
- **Extracted Shared Helpers** (`shared/common/parent-dashboard/api.ts`):
  - `parentDashboardAccessApiPath()` $\rightarrow$ `"/guardian/parent-dashboard/access"`
  - `parentDashboardOtpApiPath()` $\rightarrow$ `"/guardian/parent-dashboard/otp"`
  - `parentDashboardReportPath(childId: number, range: ParentDashboardTimeRange)` $\rightarrow$ `"/guardian/parent-dashboard/${childId}/academic-report?range=${encodeURIComponent(range)}"`
  - `buildParentDashboardOtpPayload(action: 'send' | 'verify', otp?: string)`
  - `buildParentDashboardQueryParams(screen: string, query?: Record<string, string | number | boolean>)`
  - `isParentDashboardAccessError(error: unknown)`
  - `shouldRetryParentDashboardQuery(failureCount: number, error: unknown)`
- **Commit**: `refactor(parent-dashboard): centralize shared api helpers`

---

### 4. `teacher-notes`
- **Web Files**:
  - [teacher-notes-queries.ts](file:///d:/PrepdhaSchool2/lib/query/teacher-notes-queries.ts)
  - [TeacherNotesManager.tsx](file:///d:/PrepdhaSchool2/features/teacher-notes/TeacherNotesManager.tsx)
- **Mobile Files**:
  - [teacher-notes.api.ts](file:///d:/PrepdhaSchool2/mobile/features/teacher-notes/api/teacher-notes.api.ts)
- **Extracted Shared Helpers** (`shared/common/teacher-notes/api.ts`):
  - `teacherNotesApiPath()` $\rightarrow$ `"/teacher/notes"`
  - `teacherNoteApiPath(noteId: number)` $\rightarrow$ `"/teacher/notes/${noteId}"`
  - `teacherNoteSectionsApiPath()` $\rightarrow$ `"/teacher/notes/sections"`
  - `teacherNoteAttachmentsApiPath(noteId: number)` $\rightarrow$ `"/teacher/notes/${noteId}/attachments"`
  - `teacherNoteAttachmentApiPath(attachmentId: number)` $\rightarrow$ `"/teacher/notes/attachments/${attachmentId}"`
  - `buildTeacherNotesQueryParams(topicId: number)` $\rightarrow$ `{ topicId }`
  - `buildCreateTeacherNotePayload(input: CreateTeacherNoteInput)`
  - `buildUpdateTeacherNotePayload(input: UpdateTeacherNoteInput)`
  - `buildTeacherNoteAttachmentPayload(input: TeacherNoteAttachmentPayload)`
  - `normalizeTeacherNotes(data: unknown)` $\rightarrow$ `Array.isArray(data) ? data : []`
- **Commit**: `refactor(teacher-notes): centralize shared api helpers`

---

### 5. `student-learning/topics`
- **Web Files**:
  - [annotations.client.ts](file:///d:/PrepdhaSchool2/features/annotations/api/annotations.client.ts)
  - [schedule-revision-api.ts](file:///d:/PrepdhaSchool2/components/student/learn/learning/schedule-revision-api.ts)
  - [book-reader-queries.ts](file:///d:/PrepdhaSchool2/lib/query/book-reader-queries.ts)
- **Mobile Files**:
  - [topics.api.ts](file:///d:/PrepdhaSchool2/mobile/features/topics/api/topics.api.ts)
  - [learn.api.ts](file:///d:/PrepdhaSchool2/mobile/features/learn/api/learn.api.ts)
  - [notesApi.ts](file:///d:/PrepdhaSchool2/mobile/features/topics/api/notesApi.ts)
  - [annotations.api.ts](file:///d:/PrepdhaSchool2/mobile/features/topics/annotations/annotations.api.ts)
- **Extracted Shared Helpers** (`shared/common/student-learning/api.ts`):
  - `highlightsApiPath()` $\rightarrow$ `"/annotations/highlights"`
  - `highlightApiPath(highlightId: string)` $\rightarrow$ `"/annotations/highlights/${highlightId}"`
  - `commentsApiPath()` $\rightarrow$ `"/annotations/comments"`
  - `commentApiPath(commentId: string)` $\rightarrow$ `"/annotations/comments/${commentId}"`
  - `topicFeedbackApiPath()` $\rightarrow$ `"/feedback"`
  - `topicFeedbackCountsApiPath()` $\rightarrow$ `"/feedback/counts"`
  - `revisionNoteApiPath()` $\rightarrow$ `"/revision-note"`
  - `scheduleRevisionApiPath()` $\rightarrow$ `"/revision/schedule"`
  - `buildHighlightsQueryParams(params: { pageId?: number; topicId?: number })`
  - `buildSubmitTopicFeedbackPayload(params: { topicId: number; reaction: string; comment: string })`
  - `buildSaveRevisionNotePayload(params: SaveRevisionNoteInput)`
- **Commit**: `refactor(student-learning): centralize shared api helpers`

---

### 6. `flashcard`
- **Web Files**:
  - [flashcard-queries.ts](file:///d:/PrepdhaSchool2/lib/query/flashcard-queries.ts)
  - [ReviewFlashcards.tsx](file:///d:/PrepdhaSchool2/components/flashcard/ReviewFlashcards.tsx)
  - [FlashcardSide.tsx](file:///d:/PrepdhaSchool2/components/flashcard/card/FlashcardSide.tsx)
  - [FlashcardEdit.tsx](file:///d:/PrepdhaSchool2/components/flashcard/card/FlashcardEdit.tsx)
- **Mobile Files**:
  - [flashcard.api.ts](file:///d:/PrepdhaSchool2/mobile/features/flashcard/api/flashcard.api.ts)
- **Extracted Shared Helpers** (`shared/common/flashcard/api.ts`):
  - `flashcardSetupApiPath()` $\rightarrow$ `"/flashcard/setup"`
  - `flashcardRefillApiPath()` $\rightarrow$ `"/flashcard/refill"`
  - `flashcardSessionApiPath()` $\rightarrow$ `"/flashcard/session"`
  - `topicFlashcardsApiPath(topicId: number)` $\rightarrow$ `"/flashcard/topics/${topicId}/cards"`
  - `flashcardCardApiPath(cardId: number)` $\rightarrow$ `"/flashcard/cards/${cardId}"`
  - `generateAIFlashcardsApiPath()` $\rightarrow$ `"/flashcard/generate-ai-flashcards"`
  - `buildFlashcardRefillQueryParams(topicIds?: number[])` $\rightarrow$ `topicIds?.length ? { topicIds: topicIds.join(",") } : undefined`
  - `buildFlashcardSessionPayload(learningQueue: unknown[])` $\rightarrow$ `{ learningQueue }`
  - `buildCreateTopicFlashcardsPayload(cards: unknown[])` $\rightarrow$ `{ cards }`
- **Commit**: `refactor(flashcard): centralize shared api helpers`

---

### 7. `practice/exam-results`
- **Web Files**:
  - [PracticeSetup.tsx](file:///d:/PrepdhaSchool2/components/student/practice/PracticeSetup.tsx)
  - [LiveExamsList.tsx](file:///d:/PrepdhaSchool2/components/student/practice/LiveExamsList.tsx)
  - [LiveTestRunner.tsx](file:///d:/PrepdhaSchool2/components/student/live-test/LiveTestRunner.tsx)
  - [ExamResultsView.tsx](file:///d:/PrepdhaSchool2/components/exam-results/ExamResultsView.tsx)
- **Mobile Files**:
  - [practice.api.ts](file:///d:/PrepdhaSchool2/mobile/features/practice/api/practice.api.ts)
  - [live-test.api.ts](file:///d:/PrepdhaSchool2/mobile/features/practice/api/live-test.api.ts)
  - [examResults.api.ts](file:///d:/PrepdhaSchool2/mobile/features/exam-results/api/examResults.api.ts)
- **Extracted Shared Helpers** (`shared/common/practice/api.ts`):
  - `practiceApiPath()` $\rightarrow$ `"/practice"`
  - `studentLiveTestsApiPath()` $\rightarrow$ `"/student/tests"`
  - `studentLiveTestStartApiPath(testId: number)` $\rightarrow$ `"/student/tests/${testId}/start"`
  - `studentLiveTestSubmitApiPath(testId: number)` $\rightarrow$ `"/student/tests/${testId}/submit"`
  - `studentLiveTestHeartbeatApiPath(testId: number)` $\rightarrow$ `"/student/tests/${testId}/heartbeat"`
  - `buildPracticeSetupQueryParams()` $\rightarrow$ `{ screen: "setup" }`
  - `buildPracticeQuizActionPayload(params: LoadQuizInput)` $\rightarrow$ `{ action: "loadQuiz", ... }`
  - `buildSavePracticeSessionPayload(params: SavePracticeSessionInput)` $\rightarrow$ `{ action: "savePracticeSession", ... }`
  - `buildSaveExamSessionPayload(params: SaveExamSessionInput)` $\rightarrow$ `{ action: "saveExamSession", ... }`
  - `buildQuestionStatsQueryParams(params: QuestionStatsQueryInput)`
  - `buildLiveTestSubmitPayload(answers: Record<string, unknown>, interrupted?: boolean)`
- **Commit**: `refactor(practice): centralize shared api helpers`

---

### 8. `storage/uploads`
- **Web Files**:
  - [shared/web-cms/lib/storage/client-upload.ts](file:///d:/PrepdhaSchool2/shared/web-cms/lib/storage/client-upload.ts)
  - [shared/web-cms/lib/storage/upload-config.ts](file:///d:/PrepdhaSchool2/shared/web-cms/lib/storage/upload-config.ts)
- **Mobile Files**:
  - [mobile/core/services/storage/client-upload.ts](file:///d:/PrepdhaSchool2/mobile/core/services/storage/client-upload.ts)
  - [mobile/core/services/storage/upload-config.ts](file:///d:/PrepdhaSchool2/mobile/core/services/storage/upload-config.ts)
  - [assignment-upload.api.ts](file:///d:/PrepdhaSchool2/mobile/features/teacher-dashboard/api/assignment-upload.api.ts)
- **Extracted Shared Helpers** (`shared/common/storage/upload-api.ts`):
  - `storagePresignedUrlApiPath()` $\rightarrow$ `"/api/storage/presigned-url"`
  - `storageDirectUploadApiPath()` $\rightarrow$ `"/storage/direct-upload"`
  - `buildInitiateUploadPayload(params: InitiateUploadInput)`: validates policy and outputs pure `{ purpose, scope, key, fileName, contentType, sizeBytes, schoolId }`
  - `extractStorageApiErrorMessage(status: number, statusText: string, body?: { error?: string; message?: string } | null)`
  - Direct upload purpose registry and validation rules (consolidating the duplicate `upload-config.ts` files).
- **Commit**: `refactor(storage): centralize shared api helpers`

---

### 9. `notifications`
- **Web Files**:
  - [NotificationBell.tsx](file:///d:/PrepdhaSchool2/components/notifications/NotificationBell.tsx)
  - [SubscriptionExpiryBanner.tsx](file:///d:/PrepdhaSchool2/components/guardian/SubscriptionExpiryBanner.tsx)
- **Mobile Files**:
  - [notifications.api.ts](file:///d:/PrepdhaSchool2/mobile/features/notifications/api/notifications.api.ts)
- **Extracted Shared Helpers** (`shared/common/notifications/api.ts`):
  - `notificationsBasePath(scope?: "default" | "guardian")` $\rightarrow$ `scope === "guardian" ? "/guardian/notifications" : "/notifications"`
  - `notificationUnreadCountApiPath(scope?: "default" | "guardian")` $\rightarrow$ `"${notificationsBasePath(scope)}/unread-count"`
  - `notificationsListApiPath(scope?: "default" | "guardian")` $\rightarrow$ `notificationsBasePath(scope)`
  - `notificationMarkReadApiPath(recipientId: number, scope?: "default" | "guardian")` $\rightarrow$ `"${notificationsBasePath(scope)}/${recipientId}/read"`
  - `notificationsMarkAllReadApiPath(scope?: "default" | "guardian")` $\rightarrow$ `"${notificationsBasePath(scope)}/read-all"`
  - `guardianSubscriptionExpiryAlertApiPath()` $\rightarrow$ `"/guardian/subscription-expiry-alert"`
  - `deviceTokensRegisterApiPath()` $\rightarrow$ `"/device-tokens/register"`
  - `deviceTokensUnregisterApiPath()` $\rightarrow$ `"/device-tokens/unregister"`
  - `normalizeNotificationsUnreadCount(payload: unknown): number`
  - `normalizeNotificationsList(payload: unknown): NotificationItem[]`
- **Commit**: `refactor(notifications): centralize shared api helpers`

---

### 10. `shop/gamification`
- **Web Files**:
  - [shop.client.ts](file:///d:/PrepdhaSchool2/features/shop/api/shop.client.ts)
  - [AvatarCard.tsx](file:///d:/PrepdhaSchool2/components/shop/AvatarCard.tsx)
  - [StreakFreezeShopDialog.tsx](file:///d:/PrepdhaSchool2/components/shop/StreakFreezeShopDialog.tsx)
- **Mobile Files**:
  - [shop.api.ts](file:///d:/PrepdhaSchool2/mobile/features/shop/api/shop.api.ts)
- **Extracted Shared Helpers** (`shared/common/shop/api.ts`):
  - `shopAvatarsApiPath()` $\rightarrow$ `"/shop/avatars"`
  - `shopUserPurchasesApiPath()` $\rightarrow$ `"/shop/user-purchases"`
  - `shopBuyAvatarApiPath()` $\rightarrow$ `"/shop/buy"`
  - `shopSelectAvatarApiPath()` $\rightarrow$ `"/shop/select"`
  - `shopStudentXpApiPath()` $\rightarrow$ `"/shop/student-xp"`
  - `shopStreakFreezeApiPath()` $\rightarrow$ `"/shop/streak-freeze"`
  - `shopSelectedAvatarApiPath()` $\rightarrow$ `"/shop/selected-avatar"`
  - `shopListApiPath()` $\rightarrow$ `"/shop/list"`
  - `studentAccuracyProgressApiPath()` $\rightarrow$ `"/student/accuracy-progress"`
  - `buildBuyAvatarPayload(avatarId: string)` $\rightarrow$ `{ avatarId }`
  - `buildSelectAvatarPayload(avatarId: string | null)` $\rightarrow$ `{ avatarId }`
  - `buildBuyStreakFreezePayload(quantity: number)` $\rightarrow$ `{ quantity }`
  - `mapShopApiError(payload: unknown, fallbackMessage?: string): string`
- **Commit**: `refactor(shop): centralize shared api helpers`

---

## 5. Part 2: Comprehensive Plan for Remaining Features ("Even Out From Features Order")

Beyond the initial 10 features, 16 additional features have API calls between Web and Mobile:

| # | Feature | Web API / Query Sources | Mobile API Sources | Shared Helper Target | Key Shared Functions Extracted |
|---|---|---|---|---|---|
| **11** | **`auth`** | `shared/web-cms/features/auth/`, `lib/auth/login-api-path.ts` | `mobile/features/auth/api/auth.api.ts` | `shared/common/auth/api.ts` | Endpoint builders (`authLoginApiPath()`, `authOtpApiPath()`, `authSessionApiPath()`), `buildLoginPayload()`, `buildVerifyOtpPayload()`, error mappers. |
| **12** | **`calendar`** | `app/student/calendar/page.tsx` | `mobile/features/calendar/api/calendar.api.ts` | `shared/common/calendar/api.ts` | `calendarEventsApiPath()`, `calendarAttendanceApiPath()`, query param builders for month/year range, event normalizers. |
| **13** | **`cognitive-exercises`** | `features/cognitive-exercises/api/cognitive-exercises.client.ts` | `mobile/features/cognitive-exercises/api/cognitive-exercises.api.ts` | `shared/common/cognitive-exercises/api.ts` | `cognitiveSessionsApiPath()`, `cognitiveResultsApiPath()`, payload builder for session answers, response normalizers. |
| **14** | **`student-dashboard`** | `components/student/dashboard/*` | `mobile/features/dashboard/api/dashboard.api.ts`, `profile.api.ts` | `shared/common/student-dashboard/api.ts` | `studentDashboardSummaryApiPath()`, `dailyTouchApiPath()`, `examAccuracyInsightsApiPath()`, payload & query builders. |
| **15** | **`dharani`** | `components/student/learn/learning/panels/chat-panel.tsx` | `mobile/features/dharani/api/dharani.api.ts` | `shared/common/dharani/api.ts` | `dharaniChatApiPath()`, `dharaniQuotaApiPath()`, `dharaniFeedbackApiPath()`, stream payload builders, response unwrappers. |
| **16** | **`erp`** | `shared/web-cms/lib/core/erp/erp-api.ts` | `mobile/features/erp/api/erpAccess.api.ts` | `shared/common/erp/api.ts` | `erpAccessApiPath()`, `erpModulesApiPath()`, `erpMeApiPath()`, query filter builders, permission normalizers. |
| **17** | **`fees`** | `components/fees/FamilyFeesView.tsx` | `mobile/features/fees/api/fees.api.ts` | `shared/common/fees/api.ts` | `familyFeesApiPath(familyId)`, `feeReceiptApiPath(receiptId)`, fee summary normalizers, fee status DTO transformers. |
| **18** | **`guardian`** | `app/guardian/profiles/profiles-client.tsx`, `components/guardian/*` | `mobile/features/guardian/api/guardian.api.ts` | `shared/common/guardian/api.ts` | `guardianChildrenApiPath()`, `guardianProfileApiPath()`, `guardianSubscriptionsApiPath()`, child profile DTO normalizers. |
| **19** | **`issue-report`** | `components/student/dashboard/ReportIssueDialog.tsx`, `shared/web-cms/components/support/ReportIssueDialog.tsx` | `mobile/features/issue-report/api/issueReport.api.ts` | `shared/common/issue-report/api.ts` | `reportIssueApiPath()`, `supportIssuesApiPath()`, `buildReportIssuePayload(title, desc, attachments, screenContext)`. |
| **20** | **`labs`** | `features/labs/components/LabsListPage.tsx` | `mobile/features/labs/api/labs.api.ts`, `teacher-dashboard.api.ts` | `shared/common/labs/api.ts` | `labsListApiPath()`, `labDetailApiPath(id)`, `buildLabsQueryParams(classSubjectId)`, lab item normalizers. |
| **21** | **`revision-notes`** | `features/notes/RevisionNoteEditor.tsx` | `mobile/features/notes/api/revision-notes.api.ts` | `shared/common/revision-notes/api.ts` | `revisionNoteApiPath()`, `revisionScheduleApiPath()`, `revisionCompleteApiPath()`, query & payload builders. |
| **22** | **`search`** | `components/student/dashboard/SearchModal.tsx` | `mobile/features/search/api/search.api.ts` | `shared/common/search/api.ts` | `searchTopicsApiPath()`, `searchBooksApiPath()`, `buildSearchQueryParams(q, limit, classId, subjectId)`. |
| **23** | **`staff-attendance`**| `shared/web-cms/features/staff-attendance/staff-attendance-api.ts` | `mobile/features/staff-attendance/checkin.api.ts` | `shared/common/staff-attendance/api.ts` | `staffAttendanceCheckinApiPath()`, `staffAttendanceStatusApiPath()`, `buildCheckinPayload(lat, lng, qrCode, remarks)`. |
| **24** | **`teacher-tests`** | `lib/query/teacher-tests-queries.ts`, `features/teacher-tests/*` | `mobile/features/teacher-tests/api/teacher-tests.api.ts` | `shared/common/teacher-tests/api.ts` | `teacherTestsApiPath()`, `teacherTestApiPath(id)`, `teacherTestAnalyticsApiPath(id)`, test draft payload builders. |
| **25** | **`timetable`** | `shared/web-cms/features/timetable/api.ts` | `mobile/features/timetable/api.ts` | `shared/common/timetable/api.ts` | `timetableWeeklyApiPath()`, `timetableTodayApiPath()`, `buildTimetableFilterParams(classId, sectionId)`. |
| **26** | **`todo`** | `components/student/dashboard/Backlogs.tsx` | `mobile/features/todo/api/todo.api.ts` | `shared/common/todo/api.ts` | `studentTodoApiPath()`, `studentTodoItemApiPath(id)`, `studentTodoToggleApiPath(id)`, todo payload builders. |

---

## 6. Standard Implementation Protocol (Per Feature)

For every feature $F$:
1. **Audit & Scope Check**:
   - Inspect web API/query files and mobile API files.
   - Extract string literals (`"/teacher/tasks"`), query shapes, body builders, and error wrappers.
2. **Draft Platform-Neutral Helper**:
   - Create or update `shared/common/<feature>/api.ts`.
   - Ensure zero imports from `react`, `react-native`, `next/*`, `expo-*`, or storage libraries.
3. **Update Web Callers**:
   - Replace literal strings and inline query/payload mappings with the shared helpers.
   - Preserve `goFetch` call and TanStack Query hook structure exactly as-is.
4. **Update Mobile Callers**:
   - Replace literal strings and inline query/payload mappings with the shared helpers.
   - Preserve mobile `goFetch` and token/AsyncStorage management unchanged.
5. **Run Verification Commands**:
   - Web TypeScript: `npm run type-check`
   - Mobile TypeScript: `npm --prefix mobile run type-check`
   - Ast Index: `python -m graphify update .`
6. **Isolated Feature Commit**:
   - Standard commit message: `refactor(<feature>): centralize shared api helpers`

---

## 7. Risk Analysis & Skipped Risky Areas

| Domain / Area | Risk Classification | Handling Strategy |
|---|---|---|
| **Client Transport (`goFetch`)** | **High / Critical** | **DO NOT MERGE.** Web has Next.js server/browser environment branching and CMS cookie transport; Mobile has Expo client-key headers and custom JWT bearer injection. Both remain strictly isolated. |
| **Auth Tokens & Storage** | **High / Critical** | **DO NOT MERGE.** Mobile relies on React Native `SecureStore` / `AsyncStorage`; Web relies on HTTP-only cookies and Next.js session handlers. |
| **Quiz Encryption / Decryption** | **Medium** | Kept in platform utilities (`@core/utils/quizObfuscation` on Mobile, `lib/crypto` on Web) until crypto primitives are reviewed. |
| **File System / Media Downloads** | **Medium** | `FileSystem.downloadAsync` and `Sharing.shareAsync` stay in mobile; web anchor downloads stay in web. |
| **Parent Dashboard OTP Tokens** | **Medium** | Token storage in `storage.service.ts` remains mobile-only; only the URL paths and action payloads (`{ action: 'send' }`) are shared. |
