# AGENTS.md

## Scope
- This repo is the BOOM generator only; it is not self-running. Instantiate and simulate it through Chipyard, not directly from this repo.
- CI pins the expected Chipyard revision in `CHIPYARD.hash`. If BOOM behavior depends on Chipyard integration, verify against that exact commit.

## Repo shape
- Main source lives under `src/main/scala`.
- There are two parallel codepaths: `boom.v3.*` and `boom.v4.*`. Keep edits version-scoped unless you have verified both need the change.
- The real top-level integration points are the tile/config files, not random pipeline leaf modules:
  - `src/main/scala/v3/common/config-mixins.scala`
  - `src/main/scala/v3/common/tile.scala`
  - `src/main/scala/v3/exu/core.scala`
  - `src/main/scala/v4/common/config-mixins.scala`
  - `src/main/scala/v4/common/tile.scala`
  - `src/main/scala/v4/exu/core.scala`

## Build and verification
- Style check is `make checkstyle`, which runs `sbt scalastyle test:scalastyle`.
- SBT is the primary checked build path in repo config: Scala `2.13.10`, sbt `1.9.1`.
- A Mill build file exists (`build.sc`), but CI and the Makefile use sbt. Prefer sbt-oriented verification unless you have a reason not to.
- CI verification order is effectively: prepare Chipyard/toolchains -> build a specific Verilator config -> run focused tests for that config.

## How CI actually tests BOOM
- CI does not build this repo standalone. It clones Chipyard, checks out `CHIPYARD.hash`, then replaces `chipyard/generators/boom` with this checkout.
- RTL builds happen from Chipyard Verilator make targets, driven by `.circleci/do-rtl-build.sh`.
- Config key to Chipyard config mapping is defined in `.circleci/defaults.sh`:
  - `smallboom` -> `SmallBoomConfig`
  - `mediumboom` -> `MediumBoomConfig`
  - `largeboom` -> `LargeBoomConfig`
  - `megaboom` -> `MegaBoomConfig`
  - `rv32boom` -> `SmallRV32BoomConfig`
  - `hwachaboom` -> `HwachaLargeBoomConfig`

## Focused test commands
- RISC-V regression driver: `.circleci/run-tests.sh <key>`
- Csmith regression driver: `.circleci/build-run-csmith-tests.sh <key> <runs>`
- CI coverage is config-specific:
  - `smallboom`: asm + benchmark tests, csmith 50
  - `mediumboom`: asm + benchmark tests, csmith 50
  - `largeboom`: benchmark tests only, csmith 40
  - `megaboom`: benchmark tests only, csmith 40
  - `rv32boom`: asm only
  - `hwachaboom`: special `run-rv64uv-p-asm-tests` flow with `esp-tools`
- `smallrv32boom` jobs exist in CI config but are commented out in the workflow; do not assume they are exercised regularly.

## Toolchain and environment quirks
- `hwachaboom` uses `esp-tools`, not the default `riscv-tools`.
- CI expects a prebuilt or installed Chipyard workspace plus toolchains; local reproduction usually means following `.circleci/prepare-for-rtl-build.sh` logic, not just running sbt in-place.
- Csmith setup clones and installs csmith into `$RISCV`; it also requires host packages like `m4` and `cmake`.

## Style rules worth not missing
- Indentation is 2 spaces, no tabs.
- Scala variables use lowerCamelCase; Chisel hardware signals use lower_snake_case.
- Control-structure/function opening braces stay on the same line; class/object opening braces go on the next line.
- New Scala files are expected to satisfy `scalastyle-config.xml`, including the Berkeley copyright/header format.

## Existing repo-specific caveats
- `README.md` and CI agree that BOOM development is validated through Chipyard/Verilator, not as a standalone executable project.
- There is at least one repo-local customization in `v3`: `WithMyMediumBooms` in `src/main/scala/v3/common/config-mixins.scala`. Do not assume all configs are upstream-standard.
