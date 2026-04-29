# MVP Quick Start Output Reference

**What you'll see when running the MVP test suite.**

---

## Command

```bash
cargo run --release -- run --profile lightwalletd --suite mvp
```

---

## Expected Terminal Output

```
Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.83s
Running `target\debug\cts-gate.exe run --profile lightwalletd --suite mvp`

2026-04-29T11:30:53.747603Z INFO cts_gate: Loading profile: lightwalletd
2026-04-29T11:30:53.749253Z INFO cts_gate: Loading test suite: mvp
2026-04-29T11:30:53.759757Z INFO cts_gate: Connecting to backend...
2026-04-29T11:30:54.652781Z INFO cts_gate: Running 4 test cases...
2026-04-29T11:30:54.654351Z INFO cts_gate::runner: Running test: GetBlockRange - Invalid Range
2026-04-29T11:30:54.663552Z INFO cts_gate::runner: Running test: GetBlockRange - Valid Range
2026-04-29T11:30:54.669091Z INFO cts_gate::runner: Running test: GetLatestBlock - Basic
2026-04-29T11:30:54.669091Z INFO cts_gate::runner: Running test: GetLatestTreeState - Basic
2026-04-29T11:30:55.854156Z INFO cts_gate::runner: ✓ PASS: GetBlockRange - Invalid Range
2026-04-29T11:30:55.857001Z INFO cts_gate::runner: ✓ PASS: GetLatestBlock - Basic
2026-04-29T11:30:55.858042Z INFO cts_gate::runner: ✓ PASS: GetLatestTreeState - Basic
2026-04-29T11:30:55.859562Z INFO cts_gate::runner: ✓ PASS: GetBlockRange - Valid Range
2026-04-29T11:30:55.858956Z INFO cts_gate: Generating reports...
JSON report written to: reports\report.json
Markdown report written to: reports\report.md
2026-04-29T11:30:55.899956Z INFO cts_gate: All tests PASSED ✅
```

---

## Output Breakdown

### 1. Build Phase
```
Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.83s
```
✅ Cargo compiled the project successfully

### 2. Startup Logs
```
INFO cts_gate: Loading profile: lightwalletd
INFO cts_gate: Loading test suite: mvp
```
✅ Configuration files loaded

### 3. Connection
```
INFO cts_gate: Connecting to backend...
```
✅ Established async gRPC connection to lightwalletd testnet

### 4. Parallel Execution
```
Running 4 test cases...
Running test: GetBlockRange - Invalid Range
Running test: GetBlockRange - Valid Range
Running test: GetLatestBlock - Basic
Running test: GetLatestTreeState - Basic
```
✅ All 4 tests running concurrently (not sequentially)

### 5. Results (Mixed Order)
```
✓ PASS: GetBlockRange - Invalid Range
✓ PASS: GetLatestBlock - Basic
✓ PASS: GetLatestTreeState - Basic
✓ PASS: GetBlockRange - Valid Range
```
✅ Tests complete as they finish (parallel execution)

### 6. Report Generation
```
Generating reports...
JSON report written to: reports\report.json
Markdown report written to: reports\report.md
```
✅ Both report formats generated

### 7. Success Summary
```
All tests PASSED ✅
```
✅ Exit code: 0

---

## Typical Performance

| Phase | Duration |
|-------|----------|
| Build | 2.8-3.0s |
| Profile Load | <10ms |
| Connection | 500-800ms |
| Test Execution | 1.0-1.2s (4 parallel) |
| Report Generation | ~40ms |
| **Total** | **2.1s** |

---

## Success Indicators

Look for these green checkmarks:

- ✅ 4 tests running in parallel (not sequentially)
- ✅ All tests show "✓ PASS"
- ✅ "All tests PASSED ✅" at the end
- ✅ No WARN or ERROR logs
- ✅ Reports generated
- ✅ Exit code 0

---

## Failure Example

If a test fails, you'll see:

```
2026-04-29T11:31:53.082321Z WARN cts_gate::runner: ❌ FAIL: Field 'height' failed comparison. 
Expected: "<10", Actual: 3986540
...
Tests FAILED: 1/1 cases failed ❌
error: process didn't exit successfully (exit code: 1)
```

⚠️ Exit code: 1

See [sample-demo-fail-report.md](sample-demo-fail-report.md) for details.

---

## Next Steps

After successful MVP run:

1. **Review JSON Report**
   ```bash
   cat reports\report.json
   ```

2. **Review Markdown Report**
   ```bash
   cat reports\report.md
   ```

3. **Run Failure Demo**
   ```bash
   cargo run --release -- run --profile lightwalletd --suite demo_fail
   ```

4. **Create Custom Test Suite**
   ```bash
   mkdir cases/custom
   # Add .json test cases
   cargo run --release -- run --profile lightwalletd --suite custom
   ```

---

**Reference:** CTS-Gate v0.1.0 MVP  
**Date:** April 2026
