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
  manifest.schema.json  ← JSONSchema, CI-enforced
.github/workflows/
  validate.yml          ← runs on every PR
  mirror-tarball.yml    ← packages + uploads release asset on merge (Phase 2)
```

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
