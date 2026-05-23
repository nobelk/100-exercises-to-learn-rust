# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is Mainmatter's "100 Exercises To Learn Rust" course — a Cargo workspace of ~98 small exercise crates plus an mdBook that explains the concepts. Learners progress through `exercises/<section>/<exercise>/src/lib.rs`, filling in stubs marked with `TODO`, `todo!()`, or `__` until the in-file `#[cfg(test)]` module passes.

**The tests are the specification.** Every exercise's `lib.rs` carries a "⚠️ DO NOT MODIFY THE TESTS ⚠️" banner. When fixing or completing an exercise, change only the implementation under test — never the assertions, the `mod tests` block, or the function signatures the tests depend on.

Reference solutions live on the [`solutions` branch](https://github.com/mainmatter/100-exercises-to-learn-rust/tree/solutions), not on `main`.

## Common commands

The course is meant to be driven by Mainmatter's `wr` ("workshop runner") tool, which under the hood just runs `cargo test` from each exercise directory in order. The raw cargo equivalents:

```sh
# Run the tests for a single exercise (cd into its directory first, or use -p)
cargo test -p factorial
cargo test -p factorial -- --nocapture       # show stdout from passing tests
cargo test -p factorial tests::first         # one test by path

# Build / test the entire workspace
cargo build --workspace
cargo test  --workspace

# Format all Rust + Markdown + TOML (CI enforces this via `dprint/check`)
dprint fmt
dprint check

# Build the mdBook (requires installing the local preprocessor plugins first)
cargo install --path helpers/mdbook-exercise-linker
cargo install --path helpers/mdbook-link-shortener
cd book && mdbook build
```

Crate names for `-p` are the directory's leaf name (e.g. `exercises/02_basic_calculator/05_factorial` → `-p factorial`), not the full path.

## Workspace layout

`Cargo.toml` declares the workspace with `members = ["exercises/*/*", "helpers/common", "helpers/mdbook-exercise-linker", "helpers/mdbook-link-shortener", "helpers/ticket_fields"]`, so every exercise directory two levels deep under `exercises/` is automatically a workspace member.

- `exercises/01_intro` … `08_futures` — chapters of the course. Each exercise is a self-contained crate with its own `Cargo.toml` and a single `src/lib.rs`. Most have no dependencies; later ones (e.g. `08_futures/*`) pull in `tokio`, `anyhow`, etc.
- `helpers/common`, `helpers/ticket_fields` — small shared crates that several mid-course exercises depend on (the recurring "ticket" domain model).
- `helpers/mdbook-exercise-linker`, `helpers/mdbook-link-shortener` — mdBook preprocessor binaries used only by the book build, configured in `book/book.toml`.
- `book/` — the mdBook source (`book/src/SUMMARY.md` is the table of contents; chapter folders mirror the `exercises/` layout). The CI pipeline renders it to HTML, PDF (via pandoc + LuaLaTeX), and ePUB (via calibre).
- `site/_redirects` — Netlify-style redirects generated from `book/link2alias.json` by `helpers/json2redirects.sh`. CI's `is_fresh` job re-runs the script and fails if the committed file drifts; regenerate with `./helpers/json2redirects.sh book/link2alias.json > site/_redirects` after editing the alias map.

## Conventions worth knowing

- **`dead_code = "allow"` in many exercise `Cargo.toml`s is intentional** — the course defers explaining visibility, so unused-code warnings are suppressed early on. Don't "fix" these by deleting items the tests don't yet reference.
- **`[profile.dev.package.copy] overflow-checks = true`** in the root `Cargo.toml` is load-bearing for one specific exercise that teaches overflow behavior. Don't change this profile setting.
- Formatting is governed by `dprint.json` (uses the markdown and toml dprint plugins). Rust files go through standard `cargo fmt`. The `formatter` CI job runs `dprint/check@v2.2`.
- CI (`.github/workflows/ci.yml`) does much more than build code: it produces the book artifacts, link-checks the rendered HTML with `lycheeverse/lychee-action`, and runs Gravity on the PDF. A failing CI run is often a book/link/font issue, not a code issue.