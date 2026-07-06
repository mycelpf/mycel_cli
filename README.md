# mycel

`mycel` is the umbrella CLI for the `mycel_*` family. Its first capability is the
**app-builder**: it builds, runs, tests, serves, and deploys a *federated app*
described by a single `mycelfile`.

A federated app is a **pure compiled core** (the shell + host) plus a set of
**vertically-complete modules** loaded into it **at runtime**. Isolation is
enforced by physics — the WASM sandbox, the Shadow-DOM boundary, and a per-module
Postgres schema — not by convention. The CLI compiles each module to WASM, builds
the core shell, and assembles them into one runnable app.

## Install

```bash
brew install --cask mycelpf/tap/mycel
```

Or download a platform binary directly from [Releases](https://github.com/mycelpf/mycel_cli/releases)
and put it on your `PATH`.

## Quickstart

```bash
mycel doctor               # check toolchains + that the mycelfile resolves
mycel run -f ./path/to/app # build wasm + UI + core, then launch the dev loop
```

`mycel doctor` is the first thing to run on a new machine — it reports any missing
toolchain (cargo, tinygo, wasm-opt, node/npm, go) and where to configure it.

## Commands

| Command   | What it does                                                              |
|-----------|----------------------------------------------------------------------------|
| `new app` | Scaffold a new mycel app (shell + infra + a seed module) at a directory   |
| `add module` | Scaffold a lean module into an existing app and wire it in             |
| `build`   | Build every module (WASM + UI) plus the core shell + host                 |
| `run`     | Build, then launch the app — the local dev loop                           |
| `test`    | Run the cross-language test pyramid (host + modules + UI)                 |
| `doctor`  | Check toolchains, machine config, and that the mycelfile resolves         |
| `server`  | Launch the local web console over the build engine (loopback by default)  |
| `deploy`  | Package the app for a target (`--to local\|docker\|aws\|gcp`)             |
| `cluster` | Multi-app clusters: up/down/status/verify/add-app/deploy/promote/…        |
| `login` / `logout` | Sign in via `mycel_tenants` (device flow) — needed for `cluster promote`'s `--actor` identity and any future authenticated verb |
| `version` | Print the mycel version                                                   |

`mycel <verb>` is shorthand for `mycel app <verb>` — the app-builder is the default
noun; future family tools mount as their own nouns. Run `mycel help` for the full
flag list, including `cluster`'s sub-verbs (`environments`, `pipeline`,
`deployments`, `manifest`, `promote`, …).

## License

Apache License, Version 2.0 — see [LICENSE](LICENSE). Copyright © Cogniworks
Private Limited. "mycel" and the `mycel_*` family names are trademarks of
Cogniworks Private Limited (see [NOTICE](NOTICE)); the license does not grant
rights to them beyond what Apache-2.0 itself allows for describing the
software's origin.

Third-party dependency attributions are listed in
[third_party_licenses/NOTICES](third_party_licenses/NOTICES).
