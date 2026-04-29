# Sample Demo Fail Report

**This is a reference report showing intentional test failure.**

Generated: April 29, 2026 11:31:53 UTC  
Backend: lightwalletd (Testnet)  
Suite: demo_fail

---

## Test Summary

| Metric | Value |
|--------|-------|
| **Total Tests** | 1 |
| **Passed** | 0 |
| **Failed** | 1 |
| **Success Rate** | 0% |
| **Duration** | 1.05 seconds |

⚠️ **Tests FAILED** — See details below

---

## Detailed Results

### ❌ GetLatestBlock - Intentional Fail Demo

**Status:** FAIL  
**Method:** `GetLatestBlock`  
**Duration:** 245ms

**Request:**
```json
{}
```

**Response:**
```json
{
  "height": 3986540,
  "hash": "a1b2c3d4e5f6..."
}
```

**Expected:**
```json
{
  "height": "<10"
}
```

**Validation Result:**

| Check | Expected | Actual | Status |
|-------|----------|--------|--------|
| Field `height` present | ✅ Yes | ✅ Yes | PASS |
| Field type (number) | ✅ Yes | ✅ 3986540 | PASS |
| Comparison: height < 10 | ✅ true | ❌ false | **FAIL** |

**Failure Message:**
```
❌ FAIL: Field 'height' failed comparison.
Expected: "<10"
Actual: 3986540

The backend returned a height of 3986540, which does not satisfy
the constraint height < 10. This is the intended failure scenario.
```

---

## Execution Log

```
2026-04-29T11:31:52.282401Z INFO cts_gate: Loading profile: lightwalletd
2026-04-29T11:31:52.283541Z INFO cts_gate: Loading test suite: demo_fail
2026-04-29T11:31:52.285213Z INFO cts_gate: Connecting to backend...
2026-04-29T11:31:52.898312Z INFO cts_gate: Running 1 test cases...
2026-04-29T11:31:52.899285Z INFO cts_gate::runner: Running test: GetLatestBlock - Intentional Fail Demo
2026-04-29T11:31:53.082321Z WARN cts_gate::runner: ❌ FAIL: Field 'height' failed comparison. Expected: "<10", Actual: 3986540
2026-04-29T11:31:53.082328Z INFO cts_gate: Generating reports...
JSON report written to: reports\report.json
Markdown report written to: reports\report.md
Tests FAILED: 1/1 cases failed ❌
error: process didn't exit successfully: `target\debug\cts-gate.exe run --profile lightwalletd --suite demo_fail` (exit code: 1)
```

---

## Failure Analysis

### Why This Test Failed

The `demo_fail` test suite is **intentionally designed to fail** to demonstrate:

1. ✅ Test framework's failure detection
2. ✅ Error reporting and logging
3. ✅ Report generation on failure
4. ✅ Exit code behavior (exit code 1)

### What Happened

- **Expected Condition:** Blockchain height < 10 blocks
- **Actual Condition:** Blockchain height = 3,986,540 blocks
- **Result:** Assertion failed → test marked as FAIL

This is **expected behavior** and validates that the testing framework correctly identifies when backend responses don't match expectations.

---

## Backend Information

| Property | Value |
|----------|-------|
| **Endpoint** | https://lightwalletd.testnet.cipherscan.app |
| **Backend Type** | lightwalletd |
| **Protocol** | gRPC over HTTPS |
| **Network** | Zcash Testnet |
| **Latest Height** | 3,986,540 |
| **Connection Status** | ✅ Successful |

**Note:** Backend connectivity is working correctly. The test failure is due to the deliberate unrealistic expectation (height < 10), not a backend issue.

---

## Exit Code Behavior

```
$ cargo run -- run --profile lightwalletd --suite demo_fail
...
Tests FAILED: 1/1 cases failed ❌
error: process didn't exit successfully (exit code: 1)
```

**Exit Code:** `1` (Tests failed)

This exit code enables CI/CD integration:
```bash
if [ $? -eq 0 ]; then
  echo "All tests passed"
  exit 0
else
  echo "Some tests failed"
  exit 1
fi
```

---

## Use Case: Failure Testing

This demo is useful for:

- **CI/CD Testing** — Verify test framework detects failures correctly
- **Onboarding** — Show developers what failure looks like
- **Regression Detection** — Prove test suite catches regressions
- **Documentation** — Demonstrate report generation on failure

---

## How to Recover

To return to a passing state:

1. **Option 1:** Run MVP suite (which passes)
   ```bash
   cargo run -- run --profile lightwalletd --suite mvp
   ```

2. **Option 2:** Fix the expectation in the test case
   - Edit `cases/demo_fail/999_intentional_fail.json`
   - Change `"height": "<10"` to `"height": ">0"`
   - Re-run: `cargo run -- run --profile lightwalletd --suite demo_fail`

---

**Report Generated:** 2026-04-29 11:31:53 UTC  
**Tool:** CTS-Gate v0.1.0

**Status:** ⚠️ INTENTIONAL FAILURE — Framework working correctly
