# Contributing

This repo is used for teaching. Keep changes small, reviewable, and reproducible.

## Before you open a PR

1. Build + run C++ tests:

```bash
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build -j$(nproc)
ctest --test-dir build --output-on-failure
```

2. If you touched concurrency or parallelism, run TSan:

```bash
cmake -B build-tsan -G Ninja -DCMAKE_BUILD_TYPE=Debug -DENABLE_TSAN=ON
cmake --build build-tsan -j$(nproc)
ctest --test-dir build-tsan --output-on-failure
```

3. If you touched memory, run ASan/UBSan:

```bash
cmake -B build-asan -G Ninja -DCMAKE_BUILD_TYPE=Debug -DENABLE_SANITIZERS=ON
cmake --build build-asan -j$(nproc)
ctest --test-dir build-asan --output-on-failure
```

## Commit messages

- One logical change per commit
- Subject line: imperative mood (`Fix Particle header`, `Add SD E2 benchmark`)
- Body explains *why* when the intent is not obvious

## Documentation

Docs are built with MkDocs. If you change docs:

```bash
python3 -m pip install mkdocs mkdocs-material
mkdocs build
```

