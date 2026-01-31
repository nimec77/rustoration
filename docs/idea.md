You must complete a training assignment. Its description is below:
Assignment
You have two projects: broken-app (to fix and optimize) and reference-app (to verify behavior). Your task is to fix broken-app, confirm the absence of UB, and speed up critical sections.

Broken-app contains:
At least 5 different defects (off-by-one, use-after-free/aliasing in unsafe, leak, invalid logic, inefficient algorithm, data race in the thread module).
Failing tests for some of the bugs and comment tags to identify the rest.
Intentionally slow functions with unnecessary allocations and poor asymptotics.
Simple "before" benchmarks (criterion or harness=false).

Project structure:
broken-app/
├── Cargo.toml
├── src/
│ ├── lib.rs # Main logic with bugs and unsafes
│ ├── algo.rs # Low-performance parts
│ └── bin/
│ └── demo.rs # Run example
├── tests/
│ └── integration.rs # Tests, some fail due to bugs
├── benches/
│ └── baseline.rs # Benchmarks (your changes)
├── scripts/
│ ├── profile.sh # Template Profiling (perf/valgrind)
│ └── compare.sh # Before/after comparison script (optional)
└── artifacts/ # Your before/after measurements

reference-app/
├── Cargo.toml
└── src/… # Clean implementation for comparing behavior

Step-by-Step Implementation
Step 1. Familiarization
Clone both projects (broken-app, reference-app).
In broken-app, run cargo check, run tests, and log any failing cases.
In reference-app, build and run tests to understand the expected behavior/outcomes.

Step 2. Finding and Fixing Bugs
Use a debugger (gdb/lldb) to reproduce logic errors.
Run cargo +nightly miri test to find UB. Run Valgrind (valgrind --leak-check=full cargo test --tests) to detect memory leaks/errors.
Run sanitizers (nightly): RUSTFLAGS="-Zsanitizer=address" and "-Zsanitizer=thread" for key tests.
Fix all found issues. Remove or rewrite unsafe tests if necessary.
Add regression tests for each bug found.

Step 3. Confirm Correctness
Rerun the tests, Miri, and Valgrind until clean results are achieved.
Add regression tests for the previously found bugs.

Step 4. Find Bottlenecks
Run profiling (perf + flamegraph; heaptrack/massif if necessary) on the scripts from bin/demo.rs or your tests.
Document the functions/sections that take up the most time/alloc; include a screenshot or flamegraph PNG.

Step 5. Benchmarks before optimization
Write reproducible benchmarks in benches/ (criteria with CSV/plot is recommended). Be sure to cover hot paths and "large" inputs.
Save the "before" results (e.g., artifacts/baseline_before.* or a separate commit).

Step 6. Optimization
Make at least two optimizations: one algorithmic (complexity/data structures), one micro (alloc/copy/iterators).
Rewrite hot sections: eliminate unnecessary allocations, fix complexity, remove unnecessary clones, apply iterators/slices/buffering.
Avoid "magic" and unnecessary dependencies; stick to the standard library and obvious techniques.

Step 7. Testing "after"
Run the tests again, use Miri, Valgrind, and make sure the optimizations didn't break correctness. Repeat sanitizers (ASan/TSan) on key tests.
Run the benchmarks "after" and save the results for comparison.
Compare measurements and briefly document the speedup (factor, alloc reduction, etc.).

Artifacts to submit
Link to the broken-app repository/fork with fixes and optimizations (history is preserved).
Link to the unmodified reference app (or commit hash) you were using as a reference.
Logs/run reports: cargo test, cargo +nightly miri test, valgrind --leak-check=full ..., sanitizers (ASan/TSan), cargo bench (before/after).
Benchmarks: criterion (or text) artifacts before/after, with the same sets of inputs.
Screenshots/profile text (perf/flamegraph) with bottlenecks highlighted. Brief description of optimizations and regression tests.
Optional: separate "pre-bench" and "post-bench" commits.

Check yourself before submitting for review
Make sure:
You have compiled all project artifacts.
All tests pass.
All reported bugs have been fixed.
Regression tests have been added.
cargo +nightly miri test without UB.
Valgrind without memory leaks/errors.
ASan/TSan find no errors in key tests.
Benchmarks record the speedup of the optimized version relative to the original.
Optimizations are documented, the code is readable, without unnecessary dependencies.

What we will look for during review
Correct use of the debugger, Miri, and Valgrind (confirmed runs).
Clear "before" and "after" data (time/alloc/instructions) are recorded.
Justification of optimizations: algorithmic changes > pointless micro-tweaks.
No new UB/leaks after optimization.
Clear tests and benchmarks, reproducible results.

How to submit a project
After completing the project, your project repository should be located in your GitHub account.and be public. The submission process is as follows:

Prepare the repository:
Place all artifacts in a folder within the repository.
Ensure all files are committed.
Verify that the project compiles: cargo build --workspace.
Ensure the README contains all necessary instructions.
In the project tutorial:
Insert the link to your GitHub repository in the link field.
Example: https://github.com/your-username/blog-project.
Click "Submit."
Wait for review:
The reviewer will review your project within a few days.
Review includes running all components and checking the code.
If there are any comments:
The reviewer will leave comments as Issues on GitHub.
Read all comments carefully.
Ask clarifying questions in the Issue comments if anything is unclear. Fixing Issues:
Make the necessary changes to the code.
Commit the changes to the repository.
Reply to the reviewer's comments in the Issue, indicating what has been fixed.
Re-Review:
The reviewer will review the changes.
If any issues remain, new comments will be added to the Issue.
If all issues are closed, the project is accepted.
After all issues are closed, you will see a message indicating that the project work has been submitted.
