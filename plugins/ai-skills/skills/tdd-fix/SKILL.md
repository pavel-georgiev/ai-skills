---
name: tdd-fix
description: Test-driven bug fixing - reproduce issue with a failing test, then fix without modifying the test
---


# TDD Bug Fix

A test-driven approach to fixing bugs: first prove the bug exists with a failing test, then diagnose and fix it.

## Step 1: Understand the Problem

1. Read the bug description from `$ARGUMENTS`
2. If no argument provided, use AskUserQuestion:
   - "What issue are you experiencing?"
   - Ask for error messages, unexpected behavior, or reproduction steps
3. Clarify the **expected behavior** vs **actual behavior** - this is what the test will assert

## Step 2: Locate Test Target (Minimal Investigation)

Do just enough investigation to know where to write the test:

1. Use Grep/Glob to find the affected module/component
2. Find existing test files for that module
3. Identify which interfaces are affected (CLI, API, library)

Do NOT diagnose root cause yet. Output:
- **Affected Area**: {module/component}
- **Test File(s)**: {where to add the test}
- **Interface(s)**: {CLI / API / library}

## Step 3: Create Failing Test(s)

Write tests for the **affected functionality**, not the specific failure condition.

**Key principle**: Test that the feature works, not that the bug is handled. A test for "delete prompt works end-to-end" is better than "handle empty API response" because:
- It validates the feature actually works
- It catches the same bug plus other potential issues
- It remains valuable documentation after the fix

### Example

Bad (tests the specific failure):
```python
def test_handle_empty_api_response():
    # Simulates the bug condition
    mock_response = Response(status=204, body="")
    result = parse_response(mock_response)
    assert result is None
```

Good (tests the functionality):
```python
def test_delete_prompt_removes_from_list():
    # Create a prompt, delete it, verify it's gone
    prompt = create_prompt(name="test")
    delete_prompt(prompt.id)
    prompts = list_prompts()
    assert prompt.id not in [p.id for p in prompts]
```

The good test catches the JSON parsing bug naturally because it exercises the real delete flow.

### Guidelines

1. **Determine the appropriate test file(s)**:
   - Look for existing tests for the affected module
   - Follow project test conventions

2. **Test the affected functionality, not the bug**:
   - Ask: "What feature is broken?" not "What error occurred?"
   - Write a test that proves the feature works when passing
   - The test should fail now (because the feature is broken) and pass after the fix

3. **Consider all affected interfaces**:
   - If the bug affects CLI: write an e2e test that runs the CLI via subprocess
   - If the bug affects API: write an integration test that calls the API endpoints
   - If the bug is in shared code: write tests for BOTH CLI and API to ensure both are fixed
   - Unit tests alone are not sufficient for bugs that manifest in CLI or API usage

4. **Prefer real execution over mocking**:
   - Use no mocking or minimal mocking - test against real behavior
   - Create temporary files/directories with real content rather than mocking file systems
   - Call real endpoints rather than mocking HTTP responses
   - Only mock external services that are unavailable in test environments

5. **Watch for process boundary issues**:
   - If the feature uses singletons or global state, mocking in the same process won't catch initialization bugs
   - CLI commands often run in subprocesses where parent process state doesn't exist
   - API requests may spawn worker processes that lack server-initialized state
   - When existing unit tests pass but the bug persists, suspect a process boundary issue
   - **Key question**: "Does the test run in the same process as the bug, or does production cross a process boundary?"

6. **Name tests descriptively**: `test_{feature}_{expected_behavior}` or similar

After writing the test(s), run them using the project's test runner.

## Step 4: Verify Test Fails

The test MUST fail. This proves:
- The bug exists and is reproducible
- The test correctly captures the expected behavior

If the test passes:
1. The bug may not exist, or the reproduction steps are incomplete
2. The test may not be testing the right thing
3. Use AskUserQuestion to clarify with the user before proceeding

Output: "Test fails as expected: {failure message summary}"

## Step 5: Diagnose Root Cause

NOW investigate why the test fails:

1. Read the code paths exercised by the test
2. Trace execution to understand current behavior
3. Identify the root cause of the discrepancy
4. **If existing unit tests pass but e2e fails**: Check for process boundary issues:
   - Does the feature depend on singletons/global state?
   - Is that state initialized in the entry point the e2e test uses?
   - Search for `subprocess`, `Popen`, `multiprocessing` to find process boundaries
   - Trace from CLI/API entry point to where state is consumed

Document your findings:
- **Expected**: {what the test asserts should happen}
- **Actual**: {what the code currently does}
- **Root Cause**: {why the code behaves this way}
- **Proposed Fix Location**: {file:line}

Use AskUserQuestion to confirm diagnosis before fixing:

"The test fails because:

**Root Cause**: {explanation}
**Proposed Fix**: {description}

Proceed with this fix?"

Options:
- Yes, implement the fix
- No, let me clarify

## Step 6: Implement the Fix

1. Make the minimal change to fix the bug
2. Do NOT:
   - Refactor unrelated code
   - Add extra features
   - Change the test
3. Keep the fix focused and small

## Step 7: Verify Fix

Run the same test(s) again.

### If Test Passes

1. Run the full test suite to check for regressions
2. Output: "Bug fixed! Test now passes."
3. Summarize the fix

### If Test Still Fails

DO NOT modify the test. Instead:

1. Analyze why the fix didn't work
2. Use AskUserQuestion:
   "The test still fails after my fix attempt.

   **What I tried**: {description}
   **Why it failed**: {analysis}

   Options:
   - Try alternative approach: {describe approach}
   - Discuss the issue further
   - Revisit the diagnosis"

3. If user wants to try alternative, go back to Step 6
4. If multiple attempts fail, discuss whether:
   - The diagnosis was incorrect
   - The test needs adjustment (only with user approval)
   - The issue is more complex than initially thought

## Summary Template

When complete, output:

```
## Bug Fix Summary

**Issue**: {original problem description}
**Root Cause**: {what was wrong}
**Tests Added**:
  - Unit: {test_name} (if applicable)
  - CLI e2e: {test_name} (if CLI affected)
  - API e2e: {test_name} (if API affected)
**Fix Applied**: {file(s) modified}
**Changes**: {brief description of fix}
```
