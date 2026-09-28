# twake-workflows

Reusable GitHub Actions workflows for Twake apps.

## build-and-publish-cozy-app.yml

Lints, tests, builds, runs BundleMon (when a `.bundlemonrc` exists) and publishes with `yarn cozyPublish` on `master` and version tags.

Apps must use Yarn 4.

`.github/workflows/build-and-publish-cozy-app.yml` in the app:

```yaml
name: Build and publish Cozy app

on:
  pull_request:
  push:
    branches:
      - master
    tags:
      - '[0-9]+.[0-9]+.[0-9]+'
      - '[0-9]+.[0-9]+.[0-9]+-beta.[0-9]+'

jobs:
  ci-cd:
    uses: linagora/twake-workflows/.github/workflows/build-and-publish-cozy-app.yml@main
    with:
      # Only for apps publishing with --postpublish mattermost
      mattermost-channel: '{"dev":"my-app","beta":"my-app,publication","stable":"my-app,publication"}'
    secrets:
      REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
      DOWNCLOUD_SSH_KEY: ${{ secrets.DOWNCLOUD_SSH_KEY }}
      MATTERMOST_HOOK_URL: ${{ secrets.MATTERMOST_HOOK_URL }}
```

Apps in the `linagora` organization can use `secrets: inherit` instead: it does not work across organizations.

## Adding to this repository

- Reusable workflows go directly in `.github/workflows/`: GitHub ignores subfolders. Prefix them by audience (`app-*`, `lib-*`).
- Pin third-party actions by commit SHA with a `# vX` comment.
