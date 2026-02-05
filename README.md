# Rustoration

A Rust code repair and optimization learning project demonstrating how to identify and fix common bugs, memory safety issues, and performance problems using modern Rust tooling.

## Overview

Rustoration is a workspace containing two crates:

- **broken-app**: Contains intentionally buggy code that demonstrates common Rust pitfalls
- **reference-app**: Contains the reference implementation for comparison

The project showcases various types of bugs and their fixes:

- Memory safety issues (use-after-free, memory leaks)
- Undefined behavior (off-by-one errors, data races)
- Logic errors (incorrect calculations)
- Performance bottlenecks (algorithmic complexity)

## Project Structure

```
rustoration/
├── Cargo.toml              # Workspace configuration
├── broken-app/             # Main crate with fixed code
│   ├── src/
│   │   ├── lib.rs          # Core functions (sum_even, leak_buffer, etc.)
│   │   ├── algo.rs         # Algorithm functions (slow_fib, slow_dedup)
│   │   ├── concurrency.rs  # Thread-safety functions (race_increment)
│   │   └── bin/demo.rs     # Demo binary
│   ├── tests/
│   │   └── integration.rs  # Integration tests
│   ├── benches/
│   │   ├── baseline.rs     # Simple benchmarks
│   │   └── criterion.rs    # Criterion benchmarks
│   └── artifacts/          # Verification output files
├── reference-app/          # Reference implementation
└── docs/
    └── CHANGELOG.md        # Detailed project history
```

## Getting Started

### Prerequisites

- Rust 2024 Edition (nightly recommended for full tooling support)
- Cargo

### Optional Tools (for verification)

```bash
# Miri - undefined behavior detection
rustup +nightly component add miri

# LLVM sanitizers (AddressSanitizer, ThreadSanitizer)
rustup component add llvm-tools-preview

# Valgrind (system package)
sudo apt install valgrind  # Debian/Ubuntu
```

### Installation

```bash
git clone https://github.com/yourusername/rustoration.git
cd rustoration
cargo build
```

## Usage

### Run Tests

```bash
# Run all integration tests
cargo test -p broken-app

# Run specific test
cargo test -p broken-app sums_even_numbers
```

### Run Benchmarks

```bash
# Simple baseline benchmarks
cargo bench -p broken-app --bench baseline

# Criterion benchmarks (if available)
cargo bench -p broken-app --bench criterion
```

### Run Demo

```bash
cargo run -p broken-app --bin demo
```

## Verification Tools

### Miri (Undefined Behavior Detection)

```bash
cargo +nightly miri test -p broken-app
```

### AddressSanitizer (Memory Errors)

```bash
RUSTFLAGS="-Z sanitizer=address" cargo +nightly test -p broken-app \
    --target x86_64-unknown-linux-gnu
```

### ThreadSanitizer (Data Race Detection)

```bash
RUSTFLAGS="-Z sanitizer=thread" cargo +nightly test -p broken-app \
    --target x86_64-unknown-linux-gnu
```

### Valgrind (Memory Leak Detection)

```bash
cargo test -p broken-app --no-run
valgrind --leak-check=full ./target/debug/deps/integration-*
```

## Functions

### Core Functions (`lib.rs`)

| Function | Description |
|----------|-------------|
| `sum_even(values: &[i64])` | Sum all even values in a slice |
| `leak_buffer(input: &[u8])` | Count non-zero bytes in a buffer |
| `normalize(input: &str)` | Remove whitespace and convert to lowercase |
| `average_positive(values: &[i64])` | Calculate average of positive values only |
| `use_after_free()` | Demonstrates use-after-free pattern (unsafe) |

### Algorithm Functions (`algo.rs`)

| Function | Description |
|----------|-------------|
| `slow_fib(n: u64)` | Calculate nth Fibonacci number |
| `slow_dedup(values: &[u64])` | Remove duplicates and sort |

### Concurrency Functions (`concurrency.rs`)

| Function | Description |
|----------|-------------|
| `race_increment(iterations, threads)` | Thread-safe counter increment |
| `read_after_sleep()` | Read counter after delay |
| `reset_counter()` | Reset global counter |

## Bugs Fixed

| Bug | Type | Fix |
|-----|------|-----|
| `sum_even` off-by-one | Undefined Behavior | Fixed loop bound `0..=len` → `0..len` |
| `leak_buffer` memory leak | Memory Leak | Added `Box::from_raw()` cleanup |
| `use_after_free` | Undefined Behavior | Read value before dropping Box |
| `average_positive` logic | Logic Error | Divide by `positives.len()` |
| `normalize` whitespace | Logic Error | Use `filter(!c.is_whitespace())` |
| `race_increment` data race | Data Race | Replace `static mut` with `AtomicU64` |

## Performance Optimizations

| Function | Before | After | Improvement |
|----------|--------|-------|-------------|
| `slow_fib` | O(2^n) recursive | O(n) iterative | ~65,000x |
| `slow_dedup` | O(n² log n) | O(n log n) | ~36x |

## Test Results

All tests passing after fixes:

```
running 8 tests
test averages_only_positive ... ok
test counts_non_zero_bytes ... ok
test dedup_preserves_uniques ... ok
test fib_small_numbers ... ok
test normalize_simple ... ok
test sums_even_numbers ... ok
test test_use_after_free ... ok
test race_increment_is_correct ... ok

test result: ok. 8 passed; 0 failed
```

## Documentation

See [docs/CHANGELOG.md](docs/CHANGELOG.md) for detailed project history including:
- Complete commit timeline
- Detailed bug descriptions and fixes
- Verification artifacts and tool outputs
- Performance benchmarks

## License

This project is for educational purposes.

## Acknowledgments

- [Miri](https://github.com/rust-lang/miri) - Undefined behavior detection
- [Valgrind](https://valgrind.org/) - Memory error detection
- [Criterion](https://github.com/bheisler/criterion.rs) - Benchmarking library
