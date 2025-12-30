
---

### File: `exercises/SD-E2-benchmark-and-optimize/README.md` (REPLACE)

```markdown
# SD-E2: Benchmark and Optimise

**Timetable slot:** Software Design **E2** (60 minutes)

This exercise is a constrained optimisation loop: measure first, change one thing, measure again.

## Learning objectives

By the end you can:

- Run Google Benchmark and interpret basic results
- Use a baseline to avoid “optimising blind”
- Apply one targeted optimisation (layout or loop) without breaking correctness

## Timebox plan (60 min)

- 0–10: build and run the baseline benchmark
- 10–25: choose one optimisation lever (layout or loop)
- 25–45: implement the change and re-run benchmark
- 45–55: validate correctness (same numeric results)
- 55–60: write down baseline vs improved results + commit

## Success criteria

- `bench_layout` runs
- You capture baseline and improved numbers
- Correctness is preserved within tolerance

## Where to work

Start in:

- `starter/`

## Commands

```bash
cd exercises/SD-E2-benchmark-and-optimize/starter

cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build -j"$(nproc)"

./build/bench_layout --benchmark_format=console
```

## Stretch (only if you finish early)

* Add a second benchmark case (different particle count)
* Add a lightweight correctness check inside the benchmark code path
