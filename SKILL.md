---
name: test-coverage-rubric
description: "Apply a deterministic rubric when assessing test coverage in any code review. Use this skill whenever you are evaluating whether a PR, diff, or changeset has adequate tests — even if the review request does not explicitly mention tests. Required for QA Engineer, Test Coverage Reviewer, Correctness Reviewer, and any agent running a code review that includes a test assessment dimension. Invoke before writing any test coverage verdict."
---

# Test Coverage Rubric

This skill gives you a deterministic, five-check rubric for assessing test coverage in a PR.
The goal is consistent, predictable verdicts — ICs should be able to read this rubric and know
exactly what test level will satisfy QA before they open the PR.

## Why this skill exists

Retrospective analysis of QA reviews (TEC-1870, TEC-1773, TEC-1534, TEC-1375, TEC-1163) found
that test adequacy assessment was the most variable dimension: some PRs passed with tests that
missed critical error paths; others were held up demanding E2E coverage for trivial backend
changes. This rubric replaces judgment calls with a table-driven gate.

---

## Step 1 — Scope the PR

Before applying any check, identify what changed:

- **New or modified public functions/methods** → Unit test check applies
- **New or modified modules/components** → Error-path check applies
- **New REST endpoints** → API integration check applies
- **New React pages or client-side routes** → UI flow check applies
- **Any deleted test files or removed test cases** → Regression guard check applies

Mark checks N/A if the change genuinely does not touch that category. Be conservative with N/A —
"no new endpoints" requires you to actually confirm the diff adds no route registrations.

---

## Step 2 — Apply the five checks

### Check 1 — Unit tests (BLOCKING if failed)

Every new or modified **public** function or method must have at least one unit test that
exercises it. Private/internal helpers are out of scope unless they are directly exported.

**FAIL condition:** One or more public functions with zero test coverage → BLOCKING.

---

### Check 2 — Error paths (BLOCKING if failed)

Each tested component or module must have at least one test for a failure or error case:
invalid input, authentication failure, service error, empty/null response, or similar.
"At least one" means one per module, not one per function.

**FAIL condition:** A module has unit tests but none of them test a failure scenario → BLOCKING.

---

### Check 3 — API endpoints (BLOCKING if failed, N/A if no new routes)

New REST endpoints need at least one integration or route-level test that checks:
1. The HTTP response status code
2. The shape of the response body (not just that it is non-empty)

Existing endpoints modified only in business logic (not contract) do not require new route tests,
but any new route registration triggers this check.

**FAIL condition:** A new route is registered with no route/integration test → BLOCKING.

**N/A condition:** The diff adds no new route registrations.

---

### Check 4 — UI flows (WAIVABLE, not BLOCKING)

New React pages or new client-side route entries need at least one E2E smoke test covering
the happy path (page renders, primary action succeeds).

Unlike the other checks, this one is **waivable** when a test stub or follow-up issue is
created at the time of review. A waiver is not a free pass — it requires explicit documentation.

**FAIL condition:** A new page/route exists, no E2E test, and no waiver documented → FAIL.

**WAIVED condition:** QA approves the PR with a note and creates a child issue assigned to the
owning team for the missing E2E test. The child issue ID must appear in the review comment.

**N/A condition:** The diff adds no new React pages or route entries.

---

### Check 5 — Regression guard (BLOCKING if failed)

Existing test files or test cases must not be deleted without replacement coverage.
If a test is removed because the code it covered was removed, that is acceptable — but
note it explicitly. If a test is removed for any other reason (refactor, "it was flaky",
"it was outdated"), a replacement test is required.

**FAIL condition:** Test file or test case deleted without corresponding code deletion or
replacement → BLOCKING.

---

## Step 3 — Write the verdict comment

Use this exact table format in your review comment. Fill every row; never omit a check.

```
### Test Coverage Verdict: PASS / PASS_WITH_NOTES / FAIL

| Check | Result | Notes |
|-------|--------|-------|
| Unit tests | PASS/FAIL | <which functions, or "all covered"> |
| Error paths | PASS/FAIL | <which modules, or "all covered"> |
| API endpoints | PASS/FAIL/N/A | <which routes, or "no new routes"> |
| UI flows | PASS/WAIVED/FAIL/N/A | <waiver issue ID if waived, or "no new pages"> |
| Regression guard | PASS/FAIL | <deleted tests if any, or "none deleted"> |
```

**Overall verdict rules:**

- **FAIL** if any BLOCKING check is FAIL
- **PASS_WITH_NOTES** if all BLOCKING checks pass but a UI flow is WAIVED
- **PASS** if all five checks are PASS or N/A

When the overall verdict is FAIL:
- List each BLOCKING failure as a bullet under the table
- Link any child issues created for test gaps

When the overall verdict is PASS_WITH_NOTES:
- Include the child issue ID for the waived E2E test

---

## Edge cases

**"The tests exist in a separate PR."** Not acceptable — test coverage must be present in the
same changeset being reviewed. If the author says tests are coming in a follow-up, treat the
missing tests as FAIL and require the follow-up PR to be opened before approval.

**"We use a shared test suite / integration harness."** If the shared suite already covers the
new code and you can cite the specific test cases, those count. Document which test cases cover it
in the Notes column.

**"This is a configuration-only or docs-only change."** All five checks are N/A. State that
explicitly in the verdict comment rather than omitting the table.

**"The function is internal but it is complex."** Scope creep — this rubric only mandates coverage
for public/exported symbols. Flag complex internals as a recommendation, not a BLOCKING item.

---

*TEC Custom Skill — maintained by the Deltek Technical Services Engineering team.*
