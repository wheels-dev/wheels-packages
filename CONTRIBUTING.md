# Contributing to Wheels Packages

This registry is the official distribution channel for Wheels packages. This document explains how to submit a new package, publish a new version of an existing one, and what the review process looks like.

## Before you submit

Your package must:

1. **Live in its own public git repo** on GitHub. Monorepos and subpath-based submissions are not accepted in Phase 1.
2. **Have a `package.json` manifest** at the repo root with `name`, `version`, `wheelsVersion`, and either `provides.mixins` or at least one of `provides.services` / `provides.middleware`.
3. **Declare `wheelsVersion` as a SemVer range** (e.g., `">=4.0"`). Packages that don't declare this will be rejected.
4. **Be tagged at the version you're submitting.** The tag name must match `v<version>` — e.g., manifest version `1.2.0` → tag `v1.2.0`.
5. **Ship only allowlisted file types:** `.cfc`, `.cfm`, `.cfml`, `.md`, `.json`, `.js`, `.mjs`, `.ts`, `.css`, `.scss`, `.html`, `.txt`, `.sql`, `.yml`, `.yaml`, `.gitkeep`. Anything else will block CI until a maintainer reviews and approves.
6. **Stay under 10 MB uncompressed.** Packages larger than this need explicit maintainer approval.
7. **Declare a license** in the manifest's `license` field (SPDX identifier).

## Submitting a new package

1. **Fork this repo.**
2. **Create a new directory** under `packages/` named exactly after your package — e.g., `packages/wheels-foo/`.
3. **Add `manifest.json`** following the schema at `schema/manifest.schema.json`. Leave `versions[].tarball` and `versions[].sha256` as `null` — CI fills these on merge.
4. **Add `README.md`** — a one-paragraph listing blurb. Shown on `wheels.dev/packages/<name>`.
5. **Open a PR** with title `Add wheels-foo v1.0.0`.
6. **CI runs** — validates schema, name uniqueness, source-repo resolvability, tag existence, file-type allowlist, size cap.
7. **Maintainer reviews** — confirms author, glances at the author's repo, merges if everything looks good.
8. **Mirror workflow fires on merge** (Phase 2) — packages the tagged source into a deterministic tarball, uploads to this repo's Releases, computes sha256, commits both back into your manifest.
9. **Users can now install** via `wheels packages install wheels-foo`.

## Publishing a new version

1. **Tag the new version in your own repo** (`v1.1.0` etc.).
2. **Open a PR here** that appends to `packages/wheels-foo/manifest.json`'s `versions[]` array. Do not modify previous entries.
3. **CI and review** same as above.
4. **Users run `wheels packages update wheels-foo`** — opt-in only; no auto-pull.

## Review criteria

Maintainers look at:

- Does the author repo look real (commits, README, tests)?
- Does the manifest match the schema?
- Does the tag at `source.repo` resolve?
- Does the package fit the ecosystem (Wheels app augmentation, not e.g. a general-purpose CFML library masquerading as a package)?

## What we don't do

- **We don't host source code.** Your repo stays authoritative.
- **We don't auto-update.** Version bumps require an explicit PR.
- **We don't accept author-hosted tarball URLs** (Attack A defense — see the registry design spec).
- **We don't support private packages yet.** Post-4.1.

## Getting help

Open an issue on this repo or ping `#wheels-packages` on Discord.

## Site rebuild trigger

Merging a PR that changes `packages/**` or `schema/**` on `main` automatically
rebuilds [packages.wheels.dev](https://packages.wheels.dev) via a
`repository_dispatch` event fired at `wheels-dev/wheels`. The trigger is
implemented in [`.github/workflows/notify-site.yml`](.github/workflows/notify-site.yml)
and requires a repository secret named `NOTIFY_WHEELS_TOKEN` — a fine-grained
PAT scoped to `wheels-dev/wheels` with `contents: write`.

If a registry merge does not rebuild the site within a few minutes:
1. Check the `Notify wheels.dev site` workflow run in this repo's Actions tab.
2. If it succeeded, check the `Deploy static sites` workflow in `wheels-dev/wheels`.
3. If the notify workflow failed with a 401/403, the PAT has likely expired — rotate and update the secret.
