# Agent Context

**This repo:** `ffreis-workflows-rust` — reusable GitHub Actions workflow library for
Rust projects. Covers fmt, clippy, test, matrix builds, cargo-audit, coverage,
container build, cargo-deny, docs, MSRV check, benchmarks, and Miri.

## Non-obvious rules (read before changing anything)

1. **ALL `rust-*.yml` workflows must appear in `self-test.yml`.** No exemptions.

   1a. **`rust-affected.yml`** computes which workspace crates a change touches and
   emits `cargo-args` (`--workspace` or `-p a -p b ...`) for downstream jobs to feed
   into their `*-args` inputs. It walks the **intra-workspace reverse-dependency
   graph** from `cargo metadata`, so editing a shared crate expands to every
   dependent (no false skips). It is over-approximating by design and falls back to
   `--workspace` on any uncertainty (unknown base commit, root manifest/lockfile
   change). Callers gate compile jobs on its `changed` output.

   1b. **sccache (S3 backend) is opt-in** on `rust-build/test/lint/coverage/docs`
   via the `sccache`, `sccache-bucket`, `sccache-region`, `sccache-role-arn`
   inputs. Default off → behaviour unchanged. When on with a role ARN, the job
   assumes that role via OIDC; that is why those five jobs carry
   `id-token: write` (a no-op unless sccache+role are set). `CARGO_INCREMENTAL=0`
   is forced under sccache (required — incremental + wrapper are incompatible).

   1c. **nextest is opt-in** on `rust-test.yml` (`nextest: true`). It runs
   `cargo nextest run` + a separate `cargo test --doc` (nextest skips doctests).
   Opt-in because nextest's post-`--` arg semantics differ from `cargo test`, so a
   hard swap would break callers passing `cargo test`-specific `test-args`.

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

Consumed by private Lambda repos (pinned SHA). Breaking changes to the SHA
require callers to update their pin via Renovate.

## Public repo — private-repo hygiene

This is a **public** GitHub repository. When writing commit messages, PR titles,
PR descriptions, or any other user-visible text, **never name private repos** —
website content, inventory, infra, Lambda, or data repos that are not publicly
listed. Use generic terms instead: "the fleet inventory", "a private consumer",
"internal infra", "private data repo", etc.

## Keeping this file current

- **If you discover a fact not reflected here:** add it before finishing your task.
- **If something here is wrong or outdated:** correct it in the same commit as the code change.
- **If you rename a file, command, or concept referenced here:** update the reference.
