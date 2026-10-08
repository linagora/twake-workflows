# twake-workflows

Reusable GitHub Actions workflows for Twake apps.

## build-and-publish-cozy-app.yml

Fails on any critical vulnerability in production dependencies (`yarn npm audit`), lints, tests, builds, runs BundleMon (when a `.bundlemonrc` exists) and publishes with `yarn cozyPublish` on the default branch (`master` or `main`) and version tags.

An advisory with no fix available can be ignored by its ID in `npmAuditIgnoreAdvisories` of the app's `.yarnrc.yml`.

A new push to a pull request cancels the run for the previous commit. Runs on the default branch and tags are never cancelled.

Apps must use Yarn 4.

`.github/workflows/build-and-publish-cozy-app.yml` in the app:

```yaml
name: Build and publish Cozy app

on:
  pull_request:
  push:
    branches:
      - master # or main
    tags:
      - '[0-9]+.[0-9]+.[0-9]+'
      - '[0-9]+.[0-9]+.[0-9]+-beta.[0-9]+'

jobs:
  ci-cd:
    uses: linagora/twake-workflows/.github/workflows/build-and-publish-cozy-app.yml@v1
    with:
      # Only for apps publishing with --postpublish mattermost
      mattermost-channel: '{"dev":"my-app","beta":"my-app,publication","stable":"my-app,publication"}'
    secrets:
      REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
      DOWNCLOUD_SSH_KEY: ${{ secrets.DOWNCLOUD_SSH_KEY }}
      MATTERMOST_HOOK_URL: ${{ secrets.MATTERMOST_HOOK_URL }}
```

Apps in the `linagora` organization can use `secrets: inherit` instead: it does not work across organizations.

## publish-manifest.yml

Publishes a standalone app to the registry: the archive holds only `manifest/manifest.webapp` and `manifest/icon.svg`, no code. The home and the bar open the URL held by the manifest's `client_url_flag` instead of a subdomain.

The version comes from the `vX.Y.Z` tag (or `<tag-prefix>X.Y.Z`) and must equal the one in `package.json` (or `package-json`), so the manifest carries no `version` in git (the workflow sets it, and sets `icon` to `icon.svg`). The archive goes to downcloud and the version to the `dev` channel as `X.Y.Z-dev.<commit>` unless `channel` says `beta` (`X.Y.Z-beta.<run>`) or `stable` (`X.Y.Z`).

`manifest/manifest.webapp` in the app (`client_url_flag` is the flag holding the app URL on each context):

```json
{
  "name": "Chat",
  "name_prefix": "Twake",
  "slug": "chat",
  "type": "webapp",
  "licence": "AGPL-3.0",
  "categories": ["cozy"],
  "source": "https://github.com/linagora/twake-space-chat",
  "editor": "Cozy",
  "developer": { "name": "Twake Workplace", "url": "https://twake.app" },
  "standalone": true,
  "client_url_flag": "chat.embedded-app-url",
  "permissions": {
    "banners": {
      "description": "Required by the cozy-bar to display platform messages",
      "type": "io.cozy.banners",
      "verbs": ["GET", "PUT"]
    },
    "apps": {
      "description": "Required by the cozy-bar to display the icons of the apps",
      "type": "io.cozy.apps",
      "verbs": ["GET"]
    },
    "settings": {
      "description": "Required by the cozy-bar to display storage usage",
      "type": "io.cozy.settings",
      "verbs": ["GET"]
    },
    "files": {
      "description": "Required to get shortcuts",
      "type": "io.cozy.files",
      "verbs": ["GET"],
      "selector": "name",
      "values": ["Home"]
    }
  }
}
```

`.github/workflows/publish-manifest.yml` in the app:

```yaml
name: Publish manifest

on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'

jobs:
  publish:
    uses: linagora/twake-workflows/.github/workflows/publish-manifest.yml@v1
    secrets:
      REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
      DOWNCLOUD_SSH_KEY: ${{ secrets.DOWNCLOUD_SSH_KEY }}
```

`with: channel: beta` or `stable` publishes to that channel instead of `dev`.

In a monorepo whose tags carry a prefix, `tag-prefix` and `package-json` say where the version is:

```yaml
    with:
      tag-prefix: frontend-v
      package-json: apps/frontend/package.json
```

## Releasing

Merging to `master` ships nothing until a release is published.

1. In Releases, draft a new release.
2. Type a new tag `vX.Y.Z` created on publish from `master`. Never pick the major tag itself: with immutable releases, it could never move again.
3. Generate the release notes and publish.

`update-major-tag.yml` then moves the major tag (`v1`) to the release.

Publish a new major version (`v2.0.0`) when apps have to change their workflow: renaming a workflow file, an input, a secret or a job (it renames the check apps may require), or adding a required input.

## Adding to this repository

- Reusable workflows go directly in `.github/workflows/`.
- Pin third-party actions by commit SHA with a `# vX` comment.
