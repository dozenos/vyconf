# AGENTS.md

## Project purpose
Software appliance configuration framework. Future config-session daemon for DozenOS — owns the verify/generate/apply state machine, atomic commits, rollback, and protobuf IPC. Per the in-repo README: "VyConf is an early stage of development. It's incomplete and cannot be used yet." Production DozenOS still uses the `dozenos-1x` Python config layer that wraps `dozenos1x-config` via `libdozenosconfig`.

## Tech stack
- OCaml. `dune` build system, `opam` package manager (`vyconf.opam`, `dune-project`).
- License: LGPL-2.1-or-later WITH OCaml-LGPL-linking-exception.
- Build deps (`vyconf.opam`): `menhir`, `dune (>=1.4.0)`, `ocaml-protoc`, `ounit2`, `lwt (>=4.1.0)`, `lwt_ppx`, `lwt_log`, `fileutils`, `ppx_deriving`, `ppx_deriving_yojson`, `ocplib-endian`, `xml-light`, `toml`, `sha`, `pcre2`.

## Build / test / run
```
opam install . --deps-only
dune build -p vyconf
dune runtest         # ounit2 test suite under test/
```

## Repository layout
- `src/` — daemon source (lwt-based async I/O, protobuf IPC).
- `data/` — schema/config samples.
- `scripts/` — helper scripts.
- `test/` — `ounit2` unit tests.
- `architecture.md`, `README.md` — design docs.
- `dune-project`, `vyconf.opam` — build manifest.

## Cross-repo context
Sibling to `dozenos/dozenos1x-config` (the OCaml library that owns the actual config-tree data layer). `vyconf` is the daemon that will eventually own runtime config sessions on top of `dozenos1x-config`. Currently the production runtime instead uses Python (`dozenos-1x/python/dozenos/configtree.py`) calling into `libdozenosconfig0` (built from `dozenos-1x/libdozenosconfig/`, which wraps `dozenos1x-config`). When `vyconf` lands, conf-mode/op-mode scripts in `dozenos-1x` will talk to it via protobuf IPC.

## Conventions
- Commit/PR title: `component: T12345: description` (Phorge task ID at https://dozenos.dev). Enforced by `dozenos/.github` reusable.
- Default branch: `rolling`. Release-train branches: `rolling`, `circinus`, `sagitta`, `equuleus`.
- Mergify backports via `@Mergifyio backport <branch>`.
- OCaml core repos (this, `dozenos1x-config`, `dozenos-utils`) follow MIT/LGPL upstream norms.

## Notes for future contributors
- This is pre-1.0 — APIs and IPC shapes are unstable. Coordinate any cross-repo changes with the `dozenos-1x` and `dozenos1x-config` maintainers.
- `dune subst` step (`["dune" "subst"] {pinned}`) only runs from a pinned/published opam install; vendored builds skip it.
- Architecture doc at `architecture.md` explains the verify/generate/apply phasing — read it before adding new commit semantics.
