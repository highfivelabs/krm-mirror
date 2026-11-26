# krm-mirror

Copies High Five Labs' KRM charts and images from Artifact Registry to
GitHub's container registry, by digest, every hour.

```
us-central1-docker.pkg.dev/highfivelabs-prd/krm/<path>   (published by factory)
        |  .github/workflows/mirror.yml, hourly
        v
ghcr.io/highfivelabs/krm/<path>                          (this repository's packages)
```

Install from `cr.highfivelabs.dev`, never from either registry directly:

```sh
helm install config-connector oci://cr.highfivelabs.dev/charts/config-connector --version <version>
```

`cr.highfivelabs.dev` serves ghcr.io first and Artifact Registry when ghcr.io
throttles, fails or does not have an artifact yet, so a new release is
pullable before this workflow has copied it.

## What it does

For each path in [`mirror.txt`](mirror.txt), the workflow lists the tags in
Artifact Registry (read anonymously: the repository is public) and, for each
tag missing on ghcr.io, copies the artifact by digest with
[crane](https://github.com/google/go-containerregistry), then checks that the
digest on ghcr.io is the same. A tag that already exists on ghcr.io with a
different digest fails the run and is never overwritten: release tags do not
move. A path with no repository in Artifact Registry yet (listed ahead of its
first publish, as each extension is) is reported as not published yet, not
as a failure.

The workflow runs on a schedule and by hand (`workflow_dispatch`) only, never
on pull requests. It uses this repository's `GITHUB_TOKEN` with
`packages: write`, and no other credential.

## Packages

The packages under `ghcr.io/highfivelabs/krm/` are created by this
repository's token, so they are linked to this repository. A new package
starts private: an org owner makes it public once, in the package's
settings (Package settings, Danger Zone, Change visibility, Public).
Until then, `cr.highfivelabs.dev` serves it from Artifact Registry.

## Contributing

This repository does not take pull requests. Its contents, settings and
workflow are managed from High Five Labs' infrastructure repository; changes
made here are overwritten. To report a problem with a chart or image, open
an issue on
[highfivelabs/krm-registry](https://github.com/highfivelabs/krm-registry).
