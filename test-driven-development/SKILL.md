---
name: test-driven-development
description: description: TDD, test driven development. Use before changing any production code, writing code, implementing features, adding methods, modifying classes, injecting dependencies, fixing bugs or refactoring.
---

# Test Driven Development

Test-Driven Development is a software development method where you write a failing automated test before writing the production code, then write just enough code to pass the test, and finally refactor the code for quality.

## When to use this skill

**ALWAYS:**

- Any modification to production code.
- New features.
- Dependency injections.
- Modifying classes.
- Changing implementations.
- Bug fixes.
- Behavior changes.
- Refactoring.

**If you think you're facing an exception, always ask your human teammate first.** Exceptions may include:

- Throwaway prototypes.
- Generated code.
- Configuration files.

## Fundamental principles

- If you haven't seen the test fail, you don't know if it tests what it should.
- A test that has never been red is not a reliable test.
- If you wrote production code before the test, delete it and start over. No exceptions.

## The three laws

1. Do not write production code without a failing test first.
2. Do not write more test than necessary to fail (one assert, one behavior).
3. Do not write more production code than necessary to pass the current test.

## The cycle: Red-Green-Refactor

The TDD cycle consists of three phases repeated continuously:

1. **Red**: Write a test that fails.
2. **Green**: Write minimum code to make it pass.
3. **Refactor**: Clean the code while keeping tests green.
4. **Repeat from step 1**.

**NEVER SKIP THESE STEPS.**

## Why the order matters

- **Red first**: If you don't see it fail, you don't know if it tests what it should. A test that passes immediately is a test you can't trust.
- **Green before refactor**: Don't clean what doesn't work. First make it work, then make it clean.
- **Minimum code first**: Don't guess the future, write minimum code to make the test pass. Code without tests is code without verification (YAGNI).

**Guideline:**

Skipping steps means losing TDD's guarantees. The order is not a suggestion, it's a safety mechanism.

### 1. RED - Write a failing test

- Write a minimal test for a single behavior.
- Use a clear, descriptive name.
- Use real code, no mocks unless unavoidable.
- Run the test.
- Verify it fails on an assert (not due to syntax errors).
- If it passes immediately it means that the test is wrong, fix the test not the code.

**Correct test:**

_Tests real behavior and one thing only._

```python
def test_returns_200(self, client):
    response = client.post(
        "/api/v1_0/example/",
        data=json.dumps({"example": 7}),
        content_type="application/json"
    )

    assert response.status_code == 200
    assert response.json() == {"type": "ok", "payload": {}}
```

**Incorrect test:**

_Tests the mock, not the real behavior._

```python
def test_returns_200(self, client, mock_action_to_execute):
    response = client.post(
        "/api/v1_0/example/",
        data=json.dumps({"example": 7}),
        content_type="application/json"
    )

    mock_action_to_execute.assert_called_once_with(7)
    assert response.status_code == 200
```

**Guidelines:**

- Test passes? You're testing existing behavior. Fix the test.
- Test fails with errors (not asserts)? Fix the error and run again until it fails correctly.

**Expected output for a valid red test:**

```
FAILED test_example_view.py::TestExampleView::test_returns_200 - assert 404 == 200
```

### 2. GREEN - Write minimum code to pass

- Write only what's needed for this test.
- Do not add extra functionality.
- Do not refactor yet.
- Run the test.
- If it fails, fix the code, not the test.

**Correct implementation:**

_Minimum code to pass the test._

```python
class ExampleView(APIView):
    def post(self, request: Request, *args: Any, **kwargs: Any) -> Response:
         return {"type": "ok", "payload": {}}
```

**Incorrect implementation:**

_Over-engineering, adds functionality not required by the test._

```python
class ExampleView(APIView):
    def post(self, request: Request, *args: Any, **kwargs: Any) -> Response:
        # Validate input (even though the test doesn't require it yet)
        if not request.data.get("example"):
            return {"type": "error", "message": "Example required"}

        # Prepare for future cases (YAGNI)
        options = {"validate": True, "async": False, "retry_count": 3}
        return {"type": "ok", "payload": {}}
```

**Guidelines:**

- Verify ALL tests pass, not just the new one.
- Other tests failing? Review why before continuing.

**Expected output:**

```
1 passed in 0.50s
```

### 3. REFACTOR - Clean the code

- Only when all tests are green.
- Restructure code to improve readability without changing functionality.
- Remove duplication, improve names, extract functions.
- Do not add new behavior.
- Run all tests.
- If they fail, undo the refactor.

**Example before refactor:**

```python
class ExampleView(APIView):
    def post(self, request: Request, *args: Any, **kwargs: Any) -> Response:
         return {"type": "ok", "payload": {}}
```

**Example after refactor:**

_Extracted repeated response structure to a factory. Functionality unchanged._

```python
class ExampleView(APIView):
    def post(self, request: Request, *args: Any, **kwargs: Any) -> Response:
         return ResponseFactory.create_ok()
```

**Guidelines:**

- Keep ALL tests green throughout the refactor.
- Small steps: refactor, run tests and repeat.

### 4. REPEAT from step 1

For each new behavior or functionality, start again from step 1 with a new failing test.

## Application Strategy

1. Start with the simplest test case.
2. Follow the cycle strictly for each behavior.
3. One test, one behavior, one cycle.
4. For bugs: write a test that reproduces it first, verify it fails, then fix it.

_Never fix a bug without a failing test._

**Common mistakes to avoid:**

- Writing code before the test.
- Test that passes without implementation (badly written test).
- Writing test with multiple behaviors at once.
- Adding functionality "for the future".
- Refactoring with tests in red.
- Mocking unnecessarily.
- Fixing the test when it fails on green (fix the code instead).

**Troubleshooting:**

- Don't know how to test? Write the assert first. Ask your human teammate.
- Test too complex? The design is too complex. Simplify the interface.
- Mocking everything? Coupled code. Use dependency injection.
- Test setup too complex? Use builders, mothers, or scenarios.
- Test passes but code is wrong? Test is not covering the right behavior. Review the test.

## Expected Outcomes

- Every feature has a test covering its behavior.
- Every test failed before implementing it.
- Every test failed for the expected reason.
- Code is the minimum necessary.
- All tests pass.
- No errors or warnings.
- Corner cases and error scenarios are covered.
