# Wheels Packages

The official registry for [Wheels](https://wheels.dev) packages. This repo holds package manifests and hosts their distribution tarballs as GitHub Release assets.

## What lives here

```
packages/
  wheels-sentry/
    manifest.json     ← authoritative metadata, version history
    README.md         ← listing blurb, shown on wheels.dev/packages
  wheels-hotwire/
  wheels-basecoat/
  wheels-legacy-adapter/
schema/
  manifest.schema.json         ← JSONSchema used on PRs (nullable tarball/sha256)
  manifest.strict.schema.json  ← JSONSchema used on main (tarball/sha256 required)
.github/workflows/
  validate.yml          ← runs on every PR and on push to main
  mirror-tarball.yml    ← packages + uploads release asset on PR merge
```

## How distribution works

1. Author opens a PR adding or appending a version entry with `tarball` and `sha256` set to `null`.
2. `validate.yml` checks the permissive schema, name uniqueness, that `source.repo` + `sourceTag` resolve, and that the tagged source passes the file-type allowlist + 10 MB size cap.
3. Maintainer reviews and merges.
4. `mirror-tarball.yml` fires: clones `source.repo` at `sourceTag`, produces a deterministic tarball (`tar --sort=name --mtime=@0 --owner=0 --group=0 --numeric-owner | gzip -n`), uploads it as a GH Release asset at tag `<name>-<version>`, computes sha256, and bot-commits both values back into the manifest.
5. The strict-schema job on `main` re-validates — if the mirror didn't populate both fields, this job fails and surfaces the drift immediately.

The mirror workflow can also be triggered manually (`workflow_dispatch`) to backfill any version entries left as `null` across the registry.

## How users install packages

Once the CLI ships in Wheels 4.1:

```bash
wheels packages list
wheels packages install wheels-sentry
wheels packages install wheels-sentry@1.0.0
wheels packages update wheels-sentry
```

The CLI reads manifests from this repo, downloads the tarball listed in the manifest (hosted as a GH Release asset on this repo), verifies the sha256, and activates the package into `vendor/` in the consumer's Wheels app.

## How authors submit packages

See [`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

Registry tooling and manifests: Apache 2.0. Each listed package carries its own license, declared in its manifest.
