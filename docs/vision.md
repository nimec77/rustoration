# Rustoration: Learning Guide

A step-by-step methodology for working through the **rustoration** project — fixing bugs, verifying correctness, and optimizing performance in a Rust codebase.

---

## 1. Project Overview

This is a training assignment built around two Rust subprojects:

- **`broken-app`** — a deliberately broken application containing at least five defects (off-by-one errors, use-after-free, memory leaks, invalid logic, data races, inefficient algorithms). Your job is to find and fix every one of them, then optimize the slow paths.
- **`reference-app`** — a clean, correct implementation of the same logic. Use it to understand what the code *should* do and to compare test outcomes.

Both projects live in the same repository. You will create a **Cargo workspace** at the root so that `cargo build --workspace` and `cargo test --workspace` cover everything in one command.

---

## 2. Remote Development Setup (VSCode + SSH to Ubuntu)

Set up the remote environment **before** writing any code. Several essential tools — Valgrind, `perf`, sanitizers — require Linux. macOS has limited or no support for them.

### Prerequisites

- An Ubuntu machine accessible via SSH (physical, VM, or cloud instance).
- VSCode installed on your local machine.

### Step-by-step

1. **Install the Remote - SSH extension** in VSCode (`ms-vscode-remote.remote-ssh`).

2. **Configure `~/.ssh/config`** on your local machine:
   ```
   Host rustoration-dev
       HostName <ip-or-hostname>
       User <your-user>
       IdentityFile ~/.ssh/id_ed25519
   ```

3. **Connect** — open the VSCode command palette, select *Remote-SSH: Connect to Host*, choose `rustoration-dev`, and open the project folder on the remote.

4. **Install the Rust toolchain** on the remote:
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   rustup toolchain install nightly   # needed for Miri and sanitizers
   rustup component add miri --toolchain nightly
   ```

5. **Install Linux tools**:
   ```bash
   sudo apt update
   sudo apt install -y valgrind linux-tools-common linux-tools-$(uname -r) heaptrack
   ```

6. **Install the rust-analyzer extension** on the remote VSCode server (it will prompt you automatically, or install it from the Extensions panel while connected).

7. **Create the Cargo workspace** — add a `Cargo.toml` at the repository root:
   ```toml
   [workspace]
   members = ["broken-app", "reference-app"]
   ```

8. **Verify the setup**:
   ```bash
   cargo build --workspace
   cargo test --workspace
   ```
   Both commands should complete (some tests in `broken-app` will fail — that is expected at this stage).

---

## 3. Finding and Fixing Bugs — Methodology

This section teaches **how** to find bugs systematically, not which bugs exist.

### 3.1 Initial triage

Run `cargo check` and `cargo test` in `broken-app`. Read every compiler warning and every test failure. Log the results — note which tests fail, what the error messages say, and any warnings the compiler emits. This is your starting map.

### 3.2 Compare with the reference

Run `cargo test` in `reference-app` to see what the correct outcomes look like. Diff the source files between the two projects to understand the intended behavior:

```bash
diff -ru broken-app/src/ reference-app/src/
```

This tells you exactly where the code diverges and what the correct logic should be.

### 3.3 Use a debugger

Set breakpoints with `lldb` (macOS) or `gdb` (Linux) to reproduce logic errors interactively. Step through the code at crash or failure points, inspect variables, and trace the flow of execution. This is especially useful for off-by-one errors and incorrect conditional logic.

```bash
# Example with gdb on Linux
cargo build --tests
gdb ./target/debug/deps/integration-<hash>
(gdb) break broken_app::some_function
(gdb) run
```

### 3.4 Run Miri

Miri detects undefined behavior — out-of-bounds access, use-after-free, aliasing violations, uninitialized memory reads:

```bash
cargo +nightly miri test
```

Read Miri's error messages carefully: they point to the exact line and explain the violation. Fix the issue, then re-run until Miri reports no errors.

### 3.5 Run Valgrind

Valgrind finds memory leaks, invalid reads, and invalid writes:

```bash
valgrind --leak-check=full cargo test --tests
```

Focus on the **leak summary** at the end. Common fixes: pair every `Box::into_raw` with a corresponding `Box::from_raw`, ensure every allocation is freed, and check that FFI boundaries handle ownership correctly.

### 3.6 Run sanitizers

Address Sanitizer (ASan) catches memory errors at runtime:

```bash
RUSTFLAGS="-Zsanitizer=address" cargo +nightly test
```

Thread Sanitizer (TSan) catches data races:

```bash
RUSTFLAGS="-Zsanitizer=thread" cargo +nightly test
```

If TSan reports races, look for shared mutable state — replace `static mut` with `AtomicU64` (or another appropriate atomic type), use `Mutex`, or redesign the shared access.

### 3.7 Add regression tests

For **every** bug you fix, write a test that would have caught it. This prevents regressions and documents what went wrong.

### 3.8 Final verification

Re-run all tools — `cargo test`, Miri, Valgrind, ASan, TSan — until every one reports a clean result. Do not move on to optimization until correctness is fully established.

---

## 4. Finding Bottlenecks & Optimization — Methodology

Optimize only after all bugs are fixed. Profile first, guess never.

### 4.1 Profile with `perf` and flamegraphs

```bash
cargo build --release
perf record -g ./target/release/demo
perf report
```

Generate a flamegraph for a visual overview of where CPU time is spent:

```bash
# Using cargo-flamegraph
cargo install flamegraph
cargo flamegraph --bin demo
```

### 4.2 Heap profiling

Use `heaptrack` or Valgrind's massif tool to find excessive allocations:

```bash
heaptrack ./target/release/demo
# or
valgrind --tool=massif ./target/release/demo
ms_print massif.out.<pid>
```

### 4.3 Identify hotspots

Document which functions consume the most CPU time and which allocate the most memory. Save flamegraph PNGs and profiling summaries to `artifacts/`.

### 4.4 Write "before" benchmarks

Use [Criterion](https://docs.rs/criterion) in `benches/`. Cover the hot paths with large inputs so that improvements are measurable:

```bash
cargo bench
```

Save the results to `artifacts/baseline_before.txt`.

### 4.5 Apply optimizations

Make at least:

- **One algorithmic optimization** — improve time complexity, choose a better data structure, eliminate redundant work.
- **One micro-optimization** — reduce allocations, replace `.clone()` with borrows, use iterators and slices, apply buffered I/O.

For each change, explain **why** it helps — not just what you changed.

### 4.6 Write "after" benchmarks

Re-run the same benchmarks after optimizing:

```bash
cargo bench
```

Save results to `artifacts/baseline_after.txt`. Compare the two runs and document the speedup (wall-clock time, throughput, allocation count — whichever is relevant).

---

## 5. Tools & Verification — When and Why

| Tool | What it detects | When to run | Command |
|------|----------------|-------------|---------|
| `cargo test` | Logic errors, panics | After every change | `cargo test --workspace` |
| `cargo +nightly miri test` | UB (out-of-bounds, use-after-free, aliasing) | After fixing unsafe code | `cargo +nightly miri test` |
| Valgrind | Memory leaks, invalid reads/writes | After fixing allocations | `valgrind --leak-check=full cargo test --tests` |
| ASan | Memory errors at runtime | On key tests | `RUSTFLAGS="-Zsanitizer=address" cargo +nightly test` |
| TSan | Data races | On concurrency tests | `RUSTFLAGS="-Zsanitizer=thread" cargo +nightly test` |
| `perf` + flamegraph | CPU hotspots | Before/after optimization | `perf record -g ...; perf report` |
| Criterion | Benchmark timing | Before/after optimization | `cargo bench` |
| `heaptrack` / `massif` | Heap allocation patterns | When profiling allocations | `heaptrack ./target/release/demo` |

---

## 6. Submission Checklist

Before submitting, confirm every item:

- [ ] All tests pass: `cargo test --workspace`
- [ ] Miri reports no undefined behavior: `cargo +nightly miri test`
- [ ] Valgrind reports no leaks or errors
- [ ] ASan and TSan report no errors on key tests
- [ ] A regression test exists for every fixed bug
- [ ] "Before" benchmarks saved in `artifacts/`
- [ ] "After" benchmarks saved in `artifacts/`
- [ ] Flamegraph / profiling screenshots saved in `artifacts/`
- [ ] Optimizations documented in code or commit messages
- [ ] Public GitHub repository with full commit history preserved
