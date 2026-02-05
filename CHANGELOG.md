# Rustoration Project History

## Executive Summary

**Rustoration** is a Rust code repair and optimization project focused on fixing common bugs, memory safety issues, and performance problems in a demonstration codebase. The project serves as a learning exercise for identifying and resolving issues related to:

- Memory safety (use-after-free, memory leaks)
- Undefined behavior (off-by-one errors, data races)
- Logic errors (incorrect calculations)
- Performance bottlenecks (algorithmic complexity)

### Project Timeline
- **Start Date:** January 31, 2026
- **End Date:** February 5, 2026
- **Total Commits:** 18

---

## Bug Fixes

Six critical bugs were identified and fixed during this project:

| Bug | Commit | Description | Fix |
|-----|--------|-------------|-----|
| `sum_even` off-by-one UB | `7112e63` | Loop bound caused out-of-bounds access | Changed `0..=values.len()` → `0..values.len()` |
| `leak_buffer` memory leak | `0a69abe` | Raw pointer allocation never freed | Added `Box::from_raw()` call to properly deallocate |
| `use_after_free` UB | `88348a9` | Value read after Box was dropped | Reordered to read value before dropping Box |
| `average_positive` logic | `c579cad` | Division by wrong count | Fixed to divide by `positives.len()` |
| `normalize` whitespace | `4ac77dd` | Incorrect whitespace handling | Changed to use `filter(!c.is_whitespace())` |
| `race_increment` data race | `1d2d3c6` | Unsafe `static mut` in concurrent code | Replaced with `AtomicU64` for thread safety |

### Bug Details

#### 1. sum_even Off-by-One Error
**Commit:** `7112e63` (Jan 31, 2026)

The original code used an inclusive range `0..=values.len()` which caused undefined behavior by accessing one element past the end of the slice.

#### 2. Memory Leak in leak_buffer
**Commit:** `0a69abe` (Feb 3, 2026)

Raw pointer allocation via `Box::into_raw()` was never deallocated. Fixed by adding proper cleanup with `Box::from_raw()`.

#### 3. Use-After-Free in use_after_free
**Commit:** `88348a9` (Feb 3, 2026)

The function attempted to read a value after its containing Box had been dropped. Fixed by copying the value before the drop.

#### 4. Logic Error in average_positive
**Commit:** `c579cad` (Feb 1, 2026)

The average calculation divided by the wrong count, producing incorrect results. Fixed to use the count of positive numbers only.

#### 5. Whitespace Handling in normalize
**Commit:** `4ac77dd` (Feb 4, 2026)

The normalize function didn't properly handle whitespace removal. Fixed using `filter(!c.is_whitespace())`.

#### 6. Data Race in race_increment
**Commit:** `1d2d3c6` (Feb 4, 2026)

The function used `static mut` which is inherently unsafe in concurrent contexts. Replaced with `AtomicU64` to ensure thread-safe increments.

---

## Optimizations

Two significant algorithmic optimizations were implemented:

| Function | Before | After | Speedup |
|----------|--------|-------|---------|
| `slow_fib` | O(2^n) recursive | O(n) iterative | ~65,000x |
| `slow_dedup` | O(n² log n) nested loops + per-insert sort | O(n log n) HashSet + single sort | ~36x |

### Optimization Details

#### 1. Fibonacci Optimization (slow_fib)
The original implementation used naive recursive calculation with exponential time complexity O(2^n). This was replaced with an iterative approach achieving O(n) complexity.

**Performance improvement:** ~65,000x faster for typical inputs

#### 2. Deduplication Optimization (slow_dedup)
The original implementation used nested loops with sorting on each insertion, resulting in O(n² log n) complexity. Replaced with a HashSet-based approach for O(1) lookups followed by a single sort.

**Performance improvement:** ~36x faster for typical inputs

---

## Verification Artifacts

All fixes were verified using multiple analysis tools. Final verification outputs are stored in `broken-app/artifacts/`:

### Test Results
| Artifact | Purpose | Status |
|----------|---------|--------|
| `test_output-final.txt` | Integration test results | 8/8 tests passing |
| `check_output-final.txt` | Cargo check compilation | Clean compilation |

### Memory Safety Verification
| Artifact | Tool | Purpose | Status |
|----------|------|---------|--------|
| `miri_output-final.txt` | Miri | Undefined behavior detection | 8/8 tests passing |
| `asan_output-final.txt` | AddressSanitizer | Memory error detection | 8/8 tests passing |
| `valgring_output-final.txt` | Valgrind | Memory leak detection | No application leaks* |
| `tsan_output-final.txt` | ThreadSanitizer | Data race detection | 8/8 tests passing |

\* Minor "possibly lost" bytes are from Rust runtime internals, not application code.

### Performance Analysis
| Artifact | Purpose |
|----------|---------|
| `bench_output-final.txt` | Benchmark timing results |
| `flamegraph-1.svg` | Initial CPU profile |
| `flamegraph_bench-1.svg` | Benchmark flamegraph iteration 1 |
| `flamegraph_bench-2.svg` | Benchmark flamegraph iteration 2 |
| `flamegraph_bench-3.svg` | Benchmark flamegraph iteration 3 |
| `flamegraph_bench-final.svg` | Final optimized benchmark profile |

### Final Benchmark Results
```
sum_even:   20-90ns
slow_fib:   110-120ns
slow_dedup: 258-334µs
```

---

## Commit Timeline

| Date | Commit | Description |
|------|--------|-------------|
| 2026-01-31 | `6e3fb78` | Add Rustoration project documentation and setup |
| 2026-01-31 | `1145ca4` | Update dependencies to latest versions in Cargo.toml and Cargo.lock |
| 2026-01-31 | `aba21b2` | Refactor import order and formatting in benchmark and concurrency files |
| 2026-01-31 | `6ae2eee` | Add output files for broken-app warnings and test results |
| 2026-01-31 | `7112e63` | **Fix off-by-one error in sum_even function loop** |
| 2026-02-01 | `1f94151` | Style: Reorder module declarations in `lib.rs` for consistency |
| 2026-02-01 | `189869b` | Refactor time measurement imports in baseline benchmark and add output files |
| 2026-02-01 | `c579cad` | **Add benchmark and test output files; implement average_positive function** |
| 2026-02-01 | `8e90414` | Add Miri and Valgrind output files for memory leak analysis |
| 2026-02-03 | `0a69abe` | **Fix memory leak in leak_buffer function by dropping allocated buffer** |
| 2026-02-03 | `88348a9` | **Fix use_after_free function to return correct value and add test** |
| 2026-02-04 | `1d2d3c6` | **Refactor concurrency functions to use atomic operations for thread safety** |
| 2026-02-04 | `4ac77dd` | **Refactor normalize function to remove whitespace and improve test** |
| 2026-02-04 | `751148d` | Add ASan, Miri, TSan, and Valgrind output files for memory analysis |
| 2026-02-04 | `9024681` | Refactor code structure for improved readability and maintainability |
| 2026-02-04 | `de5a7e0` | Refactor code structure for improved readability and maintainability |
| 2026-02-05 | `38320bb` | Refactor code structure for improved readability and maintainability |
| 2026-02-05 | `2dd8caa` | Add output files for Miri, testing, ThreadSanitizer, and Valgrind |

**Bold** entries indicate bug fix commits.

---

## Verification Summary

All critical issues have been resolved and verified:

- **8/8 integration tests passing**
- **No undefined behavior detected** (Miri clean)
- **No memory errors** (ASan clean)
- **No data races** (TSan clean)
- **No application memory leaks** (Valgrind clean)
- **Performance optimizations verified** via benchmarks and flamegraphs

The codebase is now free of the originally identified bugs and significantly optimized for the targeted functions.
