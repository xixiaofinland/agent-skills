# Apex Testing Reference

## Basic Test Execution

```bash
# Run a single test method
sf apex run test -t MyClass.myTestMethod -o <org> -w 180 -r human

# Run all tests in a class
sf apex run test -n MyClass -o <org> -w 180 -r human

# Run multiple classes
sf apex run test -n MyClass -n OtherClass -o <org> -w 180 -r human
```

### Key Flags

| Flag                      | Purpose                | Notes                               |
| ------------------------- | ---------------------- | ----------------------------------- |
| `-t ClassName.MethodName` | Single test method     | Dot-separated class and method      |
| `-n ClassName`            | All tests in class     | Can repeat for multiple classes     |
| `-o <org>`                | Target org             | Always required                     |
| `-w 180`                  | Wait up to 180 seconds | Prevents async polling loop         |
| `-r human`                | Human-readable output  | Use `json` for programmatic parsing |
| `-c`                      | Include code coverage  | Adds line-level coverage data       |
| `-l <level>`              | Test level             | See levels below                    |

### Test Levels

| Level               | Runs                                         |
| ------------------- | -------------------------------------------- |
| `RunSpecifiedTests` | Only the `-t`/`-n` specified tests (default) |
| `RunLocalTests`     | All local (non-managed package) tests        |
| `RunAllTestsInOrg`  | Every test in the org                        |

```bash
sf apex run test -l RunLocalTests -o <org> -w 300 -r human
```

## Code Coverage Workflow

Coverage requires two steps:

**Step 1 — Run tests with JSON output to capture the run ID:**

```bash
sf apex run test -n MyClass -o <org> -w 180 -r json -c
```

The JSON output contains `testRunId`. Extract it:

```json
{
  "result": {
    "testRunId": "707xx0000000001"
  }
}
```

**Step 2 — Fetch the coverage report:**

```bash
sf apex get test -i 707xx0000000001 -c --json
```

The response includes per-class coverage percentages and uncovered line numbers.

## Reading Test Results (human format)

Human-readable output shows:

```
Test Summary
────────────────────────────────────────────────────────────────────────────────
Outcome              Passed
Tests Ran            5
Pass Rate            100%
Fail Rate            0%
...

Test Results
────────────────────────────────────────────────────────────────────────────────
PASS  MyClass.testCreateRecord
PASS  MyClass.testUpdateRecord
...
```

Failed tests show the assertion message and stack trace inline.

## Reading Test Results (JSON format)

```json
{
  "result": {
    "summary": {
      "outcome": "Failed",
      "testsRan": 5,
      "passing": 4,
      "failing": 1,
      "testRunId": "707xx0000000001"
    },
    "tests": [
      {
        "fullName": "MyClass.testMethod",
        "outcome": "Fail",
        "message": "System.AssertException: Assertion Failed",
        "stackTrace": "Class.MyClass.testMethod: line 42, column 1"
      }
    ]
  }
}
```

## Code Coverage Requirements

Salesforce requires **75% overall code coverage** to deploy to production. Check coverage per class with `sf apex get test -i <runId> -c --json`.

Coverage output per class:

```json
{
  "name": "MyClass",
  "coveredLines": 45,
  "uncoveredLines": 5,
  "totalLines": 50,
  "coveredPercent": 90
}
```

## Anonymous Apex Execution

Run ad-hoc Apex without creating a class:

```bash
# From a file
sf apex run -f scripts/apex/myScript.apex -o <org>

# Inline (short snippets)
echo "System.debug('hello');" | sf apex run -o <org>
```

Results appear in the terminal. Use `System.debug()` to inspect values; output appears in the response.

## Debug Logs

```bash
# List recent logs
sf apex list log --json -o <org>

# Download a specific log by ID
sf apex get log -i <logId> -o <org>

# Download all recent logs to a directory
sf apex get log -d logs/ -o <org>
```
