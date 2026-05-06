# Agent Context

**This repo:** `ffreis-workflows-rust` — reusable GitHub Actions workflow library for
Rust projects. Covers fmt, clippy, test, matrix builds, cargo-audit, coverage,
container build, cargo-deny, docs, MSRV check, benchmarks, and Miri.

## Non-obvious rules (read before changing anything)

1. **ALL `rust-*.yml` workflows must appear in `self-test.yml`.** No exemptions.

2. **Miri special case:** Call Miri with `miri-args: --lib` in `self-test.yml` to avoid
   compiling Criterion benchmarks (FFI/unsafe, incompatible with Miri). Do not remove.

3. **`cargo-deny` requires `deny.toml`** at the calling repo's workspace root.
   Callers without it will fail at runtime. Do not make it optional silently.

4. **MSRV input is required** — always a concrete version (e.g., `1.80.0`), not `stable`.

5. **Coverage threshold** (`coverage-threshold`, default 80) is per-workflow input.
   Callers override per their own standard.

6. **`dtolnay/rust-toolchain` pinned to full SHA.** Renovate manages this.

7. **Concurrency is caller-controlled.** Never add `concurrency:` to reusable workflows.

8. **Container build in `self-test.yml`:** pass `push: false` or omit registry inputs —
   no registry credentials required in this repo.

## Structure

```
.github/workflows/
  rust-*.yml      ← reusable library
  devops-*.yml    ← repo-maintenance
  ci.yml
examples/hello/   ← minimal Rust project + Cargo.toml
renovate.json     ← auto-updates action SHAs
```

## Build/test

```bash
make setup              # install git hooks and verify gitleaks is installed
make fmt-check          # rustfmt check
make lint               # actionlint + clippy examples
make secrets-scan-staged
```

## Cross-repo role

Consumed by `website/ffreis-website-lambdas-rust` (pinned SHA). Breaking changes
to the SHA require callers to update their pin via Renovate.

## Keeping this file current

- **If you discover a fact not reflected here:** add it before finishing your task.
- **If something here is wrong or outdated:** correct it in the same commit as the code change.
- **If you rename a file, command, or concept referenced here:** update the reference.
