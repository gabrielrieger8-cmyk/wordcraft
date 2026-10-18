# pyxis fork

This fork exists only to build the **web (wasm) app** that the pyxis-ai hub embeds, from
upstream **release tags** plus a small stack of pyxis patch commits. Nothing here is meant
to replace upstream; the patches are also proposed upstream as PRs when they make sense.

## Branch model

- `main`, `release`, and every other upstream branch: untouched mirrors of upstream.
- `pyxis-main`: `<upstream release tag>` + pyxis commits, always rebased (never merged):
  1. the pipeline commit (`.github/workflows/pyxis-web.yml` + this file);
  2. functional patches (e.g. origin-allowlisted `postMessage` bridge for sandboxed iframes,
     save bridge), one commit each.
- `pyxis-v<x.y.z>`: the `pyxis-main` stack as rebased onto upstream `v<x.y.z>`; kept as the
  history of each base. (`pyxis` itself can't be a branch here: `pyxis/*` upstream-PR
  branches already live in this fork and git refs can't be both a file and a folder.)
- Tags `v<x.y.z>-pyxis.<n>`: a build. Pushing one runs `pyxis-web` and publishes the release
  `<app>-web-<x.y.z>-pyxis.<n>.zip` + `.sha256`. `n` restarts at 1 on every upstream bump
  (`.0` = the pipeline only, no functional patch).

## Build

`.github/workflows/pyxis-web.yml` reproduces upstream's release `web` job
(storytold/craft-common-actions `release.yml`): stable Rust + `wasm32-unknown-unknown`,
trunk 0.21.14 (upstream's sha256-verified installer, pinned), `packaging/web/package.sh`
(`trunk build --release`, wasm-opt `-Oz` from `data-wasm-opt="z"`, no craft-fonts). The only
difference is the version string (`<APP>_VERSION=<x.y.z>-pyxis.<n>`), which names the zip and
its top folder. Manual run: Actions -> pyxis-web -> Run workflow (upstream_tag, pyxis_rev) on
`pyxis-main`. The inherited upstream workflows are disabled in this fork (they need secrets).

## Upstream bump

`~/github/craft-forks/sync-upstream.sh <app> v<x.y.z>` (owner's machine): fetches the
upstream tag, rebases the stack onto it in `pyxis-v<x.y.z>`, stops on a conflict, then moves
`pyxis-main` and pushes the tag `v<x.y.z>-pyxis.1`.
