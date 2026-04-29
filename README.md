# CTS-Gate: Zcash Conformance Testing Service

A robust Rust-based tool for conformance testing of Zcash backend implementations against the gRPC service specification.

## Project Summary

CTS-Gate is a Conformance Testing Service tool that validates Zcash backend compliance with the `CompactTxStreamer` gRPC API specification. It:

- Connects to Zcash backends (e.g., lightwalletd) via gRPC
- Executes predefined test cases against the backend
- Validates responses against expected outputs
- Generates JSON and Markdown reports
- Runs tests in parallel for efficiency
- Supports multiple backend profiles and test suites

**Technology Stack:**
- Language: Rust 2021 edition
- gRPC Framework: Tonic 0.11
- Protobuf: Prost 0.12
- CLI: Clap 4
- Async Runtime: Tokio 1

---

## MVP Scope

The MVP includes 5 core test cases validating the `CompactTxStreamer` service methods:

| # | Test Case | Method | Purpose |
|---|-----------|--------|---------|
| 001 | GetLatestBlock - Basic | `GetLatestBlock` | Verify the backend returns the latest block |
| 002 | BlockRange - Multiple Blocks | `GetBlockRange` | Fetch a range of blocks |
| 003 | LatestTreeState - Tree State | `GetLatestTreeState` | Retrieve tree state for the latest block |
| 004 | Invalid Request Handling | `GetLatestBlock` | Ensure proper error handling |
| 999 | Intentional Fail Demo | `GetLatestBlock` | Demonstrate test failure reporting |

**Key Features:**
- Async gRPC connections (zero-blocking)
- Parallel test execution (4 tests concurrent)
- Efficient report generation with pre-allocated buffers
- Zero-cost comparator for result validation

---

## How to Run the MVP

### Prerequisites

- Rust 1.70+ (with `cargo`)
- Network access to a Zcash backend (e.g., lightwalletd)

### Build the Project

```bash
cargo build --release
```

### Run MVP Test Suite

```bash
cargo run --release -- run \
  --profile lightwalletd \
  --suite mvp \
  --output reports
```

**Parameters:**
- `--profile lightwalletd` — Backend profile to use (located in `profiles/`)
- `--suite mvp` — Test suite name (located in `cases/`)
- `--output reports` — Directory for report files (default: `reports`)

### Output

On success:
```
    Finished release [optimized] target(s) in 2.34s
    Running `target/release/cts-gate run --profile lightwalletd --suite mvp --output reports`
All tests PASSED ✅
```

Reports are generated in the `reports/` directory:
- `report.json` — Detailed test results
- `report.md` — Human-readable summary

---

## How to Run the Intentional Fail Demo

The `demo_fail` suite contains a test case deliberately designed to fail, showcasing error reporting:

```bash
cargo run --release -- run \
  --profile lightwalletd \
  --suite demo_fail \
  --output reports
```

**What it tests:**
- The `GetLatestBlock` test expects height `< 10`, which will fail on live networks
- Generates failure reports showing:
  - Assertion details
  - Expected vs. actual values
  - Timestamp of failure

**Expected Output:**
```
Tests FAILED: 1/1 cases failed ❌
```

Reports show the failure details in `report.json` and `report.md`.

---

## Case Format

Test cases are JSON files located in `cases/<suite-name>/`. Each file defines a single test.

### Schema

```json
{
  "name": "Test Display Name",
  "method": "GrpcMethodName",
  "request": {
    // request parameters (empty if none required)
  },
  "expected": {
    "fieldName": "comparison_value"
  },
  "validation": {
    "required_fields": ["fieldName1", "fieldName2"],
    "field_types": {
      "fieldName1": "number",
      "fieldName2": "string"
    },
    "exact_match": false,
    "allow_error": false
  }
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Human-readable test case name |
| `method` | string | gRPC method to invoke (e.g., `GetLatestBlock`) |
| `request` | object | Request parameters (structure depends on method) |
| `expected` | object | Expected response field values |
| `validation.required_fields` | array | Fields that must be present in response |
| `validation.field_types` | object | Expected types for each field (number, string, boolean) |
| `validation.exact_match` | boolean | If true, entire response must match exactly |
| `validation.allow_error` | boolean | If true, errors in response are acceptable |

### Comparison Operators

Values in `expected` support operators:

- `"non-empty"` — Field must exist and be non-null
- `">0"` — Numeric comparison (greater than)
- `"<10"` — Numeric comparison (less than)
- `"==value"` — Exact match

### Example

```json
{
  "name": "GetLatestBlock - Basic",
  "method": "GetLatestBlock",
  "request": {},
  "expected": {
    "height": ">0",
    "hash": "non-empty"
  },
  "validation": {
    "required_fields": ["height", "hash"],
    "field_types": {
      "height": "number",
      "hash": "string"
    },
    "exact_match": false,
    "allow_error": false
  }
}
```

---

## Backend Profile Format

Backend profiles define connection endpoints and settings. Located in `profiles/`.

### Schema

```json
{
  "profile": "profile_name",
  "backend": "backend_type",
  "endpoint": "grpc://host:port"
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `profile` | string | Profile identifier (used with `--profile` flag) |
| `backend` | string | Backend type (e.g., `lightwalletd`) |
| `endpoint` | string | gRPC endpoint URL (`http://`, `https://`, or `grpc://`) |

### Protocol Support

- `http://` — Unencrypted HTTP/2
- `https://` — TLS-encrypted HTTP/2
- `grpc://` — Treated as unencrypted HTTP/2

### Example

```json
{
  "profile": "lightwalletd",
  "backend": "lightwalletd",
  "endpoint": "https://lightwalletd.testnet.cipherscan.app"
}
```

---

## Report Outputs

Two report formats are automatically generated:

### 1. JSON Report (`report.json`)

Machine-readable detailed results. Structure:

```json
{
  "timestamp": "2026-04-29T14:30:00Z",
  "suite": "mvp",
  "backend": "lightwalletd",
  "total_tests": 5,
  "passed_tests": 5,
  "failed_tests": 0,
  "success_rate": 100.0,
  "duration_ms": 1234,
  "results": [
    {
      "test_name": "GetLatestBlock - Basic",
      "method": "GetLatestBlock",
      "passed": true,
      "execution_time_ms": 245,
      "request": {},
      "response": {
        "height": 2000000,
        "hash": "abc123..."
      },
      "validation_details": {
        "required_fields_present": true,
        "type_checks_passed": true
      }
    }
  ]
}
```

### 2. Markdown Report (`report.md`)

Human-readable summary. Includes:
- Test suite metadata
- Passed/failed counts
- Per-test results table
- Execution times
- Pass rate percentage

Example:
```markdown
# Test Report

**Suite:** mvp  
**Backend:** lightwalletd  
**Timestamp:** 2026-04-29 14:30:00 UTC  

## Summary

| Metric | Value |
|--------|-------|
| Total Tests | 5 |
| Passed | 5 |
| Failed | 0 |
| Success Rate | 100% |
| Duration | 1.23s |

## Results

| Test | Method | Status | Time (ms) |
|------|--------|--------|-----------|
| GetLatestBlock - Basic | GetLatestBlock | ✅ | 245 |
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | All tests passed ✅ |
| `1` | One or more tests failed ❌ |
| `2` | Configuration or I/O error (missing profile/suite) |
| `3` | Connection error (backend unreachable) |
| `4` | Protocol/gRPC error |

### Usage in Scripts

```bash
cargo run --release -- run --profile lightwalletd --suite mvp
if [ $? -eq 0 ]; then
  echo "Tests passed!"
else
  echo "Tests failed!"
fi
```

---

## Out of Scope

The following features are **not** part of the current MVP:

- **Multi-backend comparison** — Comparing responses across multiple backends
- **Test case generation** — Auto-generating test cases from proto definitions
- **Custom assertions** — Complex validation logic beyond field type/value checks
- **Performance benchmarking** — Load testing or throughput analysis
- **Web UI** — Dashboard or visual test runner
- **Continuous monitoring** — Long-running backend health checks
- **Test case versioning** — Managing multiple versions of test suites
- **Network proxy/capture** — Traffic inspection or mocking
- **Authenticated endpoints** — API key or OAuth support (basic auth possible)

---

## Current Limitations

### Known Constraints

1. **Single suite per run** — Cannot run multiple suites in one invocation
   - Workaround: Run multiple commands in a script

2. **No test filtering** — All cases in a suite execute
   - Cannot skip individual tests without modifying the suite file

3. **Sequential report generation** — Reports always overwrite previous runs
   - Workaround: Use timestamped output directories

4. **Limited error context** — Some gRPC errors lack detailed diagnostics
   - Enhancement: Improve error messages in future releases

5. **No retry logic** — Failed tests don't retry automatically
   - Enhancement: Add `--retry N` flag in future version

6. **Fixed parallelism** — Runs exactly 4 tests concurrently
   - Cannot adjust concurrency level via CLI flag

7. **No response filtering** — Full responses logged (potential data size issues)
   - Enhancement: Add `--filter-responses` option

8. **Hardcoded comparison logic** — Cannot extend validation rules without code changes
   - Enhancement: Support custom validator plugins

### Planned Improvements

- [ ] CLI flag to adjust test parallelism
- [ ] Configurable report retention and cleanup
- [ ] Test case tagging/filtering system
- [ ] Response data sanitization options
- [ ] Retry mechanism with backoff
- [ ] Multi-suite runs in single invocation
- [ ] Custom validator plugin system

---

## Directory Structure

```
cts-gate/
├── src/                          # Rust source code
│   ├── main.rs                   # Entry point, CLI parsing
│   ├── client.rs                 # gRPC client implementation
│   ├── config.rs                 # Profile & test case loading
│   ├── runner.rs                 # Test execution engine
│   ├── comparator.rs             # Result validation
│   └── report.rs                 # Report generation
├── proto/                         # Protocol Buffer definitions
│   ├── service.proto             # CompactTxStreamer service
│   └── compact_formats.proto     # Message types
├── cases/                         # Test case suites
│   ├── mvp/                      # MVP test suite (5 cases)
│   └── demo_fail/                # Failure demonstration
├── profiles/                      # Backend configurations
│   └── lightwalletd.json         # Testnet lightwalletd
├── reports/                       # Generated test reports
│   ├── report.json               # Machine-readable
│   └── report.md                 # Human-readable
├── Cargo.toml                    # Rust dependencies
├── build.rs                      # Protobuf compilation script
└── README.md                     # This file
```

---

## Troubleshooting

### Connection Errors

**Error:** `Failed to connect to https://lightwalletd.testnet.cipherscan.app`

**Solution:** Check endpoint URL and network connectivity:
```bash
curl -v https://lightwalletd.testnet.cipherscan.app
```

### Missing Profile

**Error:** `Error: Profile 'unknown' not found`

**Solution:** Verify profile exists in `profiles/` directory:
```bash
ls profiles/
```

### Test Suite Not Found

**Error:** `Error: Test suite 'custom' not found in cases/`

**Solution:** Create `cases/custom/` directory with JSON test files.

### Compilation Issues

Ensure Rust is up to date:
```bash
rustup update
```

---

## Contributing

To add new test cases:

1. Create JSON file in `cases/<suite-name>/`
2. Follow the case format schema
3. Re-run suite: `cargo run -- run --profile <profile> --suite <suite-name>`

To add new profiles:

1. Create JSON file in `profiles/`
2. Follow the profile format schema
3. Use with `--profile <name>`

---

## License

[Specify your license here]

---

**Last Updated:** April 2026  
**Version:** 0.1.0 MVP
