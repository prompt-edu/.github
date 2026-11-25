# Reusable Workflows

> This guide explains the reusable GitHub Actions workflows in this repository and how a repository adopts them. For the commit format itself, see [Conventional Commits](../CONTRIBUTING.md#conventional-commits).

| Workflow                                            | Purpose                                                    | Used by                                             |
| --------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------- |
| [`pr-title.yml`](../.github/workflows/pr-title.yml) | Checks that the PR title follows Conventional Commits      | prompt, prompt-lib, prompt-shared-state, prompt-sdk |
| [`release.yml`](../.github/workflows/release.yml)   | Runs semantic-release with the release GitHub App          | prompt-lib, prompt-shared-state, prompt-sdk         |

Other repositories, e.g. prompt-intro-course, can adopt them the same way.

A caller must grant at least the permissions that the called job declares: `pull-requests: read` for the PR title check and `contents: read` for the release.

---

## How Releases Work

1. A pull request gets a Conventional Commits title, e.g. `feat(table): add column pinning`. The PR title check makes sure it does.
2. The PR is squash-merged, so the PR title becomes the commit on `main`.
3. On the push to `main`, the release job runs semantic-release. It looks at all commits since the last release tag and decides the next version (`feat` → minor, `fix`/`perf` → patch, `!` or `BREAKING CHANGE:` → major, anything else → no release).
4. If a release is due, semantic-release does whatever the repository's `.releaserc.json` configures, e.g. publish to npm, commit the new version in `package.json` back to `main`, push the tag and create a GitHub release with generated notes.

Release notes live in the GitHub releases only; there is no `CHANGELOG.md`.

---

## Versioning of the Workflows

Callers reference the workflows by branch:

```yaml
uses: prompt-edu/.github/.github/workflows/release.yml@main
```

This means **every change merged to `main` here rolls out to all callers immediately**, with their next workflow run. When changing a workflow:

- Keep it backward compatible: do not rename or remove inputs or secrets, and give new inputs a default that keeps today's behavior.
- Test the change before merging, e.g. by pointing a caller's branch at your branch (`@your-branch`) in a test PR.
- actionlint and the PR title check run on every PR in this repository.

---

## `pr-title.yml`

Checks the PR title with [`amannn/action-semantic-pull-request`](https://github.com/amannn/action-semantic-pull-request). The scope is optional.

| Input   | Type   | Default | Description                                                              |
| ------- | ------ | ------- | ------------------------------------------------------------------------ |
| `types` | string | `''`    | Newline-separated list of allowed types. Empty uses the action defaults. |

The default types are `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore` and `revert`, as listed in [Conventional Commits](../CONTRIBUTING.md#types-and-releases).

Call it from a job with the id `pr-title`. The check then shows up as **`pr-title / conventional-commit`**, which is the name to require in the ruleset.

---

## `release.yml`

Runs semantic-release in a single job (`release`):

1. Checks out the full history with `persist-credentials: false`, so git pushes with the App token instead of `GITHUB_TOKEN`.
2. Sets up Node.
3. If the repository has a `package.json`: installs a pinned Corepack (Node.js no longer bundles it from v25 on), enables it, and runs `yarn install --immutable`. Corepack then uses the Yarn version from the repository's `packageManager` field.
4. Runs `build-command` if set.
5. Creates an installation token of the org release GitHub App. This happens only now, so that dependency install scripts and the build never run while a token that can push to `main` exists.
6. Runs semantic-release:
   - With a `package.json`: `yarn semantic-release`, i.e. the version locked in the repository's own devDependencies.
   - Without one (e.g. a Go module): exact versions of `semantic-release` and `conventional-changelog-conventionalcommits` (9.x) via `npx`. They are pinned in the job's `env` of `release.yml` and bumped by Renovate in this repository. Only the plugins bundled with semantic-release (commit-analyzer, release-notes-generator, npm, github) and the `conventionalcommits` preset are available then.

The workflow assumes nothing about the plugins; the repository's `.releaserc.json` decides what a release does.

| Input           | Type    | Default  | Description                                                                                                      |
| --------------- | ------- | -------- | ---------------------------------------------------------------------------------------------------------------- |
| `app-client-id` | string  | required | Client ID of the release GitHub App. Pass `${{ vars.RELEASE_APP_CLIENT_ID }}`.                                   |
| `node-version`  | string  | `'24.x'` | Node.js version. semantic-release 25 requires Node `^22.14 \|\| >=24.10`.                                        |
| `npm-publish`   | boolean | `true`   | Pass the npm token to semantic-release as `NPM_TOKEN`. When `false`, `NPM_TOKEN` is not set at all.              |
| `build-command` | string  | `''`     | Command to run after installing dependencies and before semantic-release, e.g. `yarn build`. Skipped when empty. |

| Secret            | Required | Description                                                                                         |
| ----------------- | -------- | --------------------------------------------------------------------------------------------------- |
| `app-private-key` | yes      | Private key of the release GitHub App. Pass `${{ secrets.RELEASE_APP_PRIVATE_KEY }}`.               |
| `npm-token`       | no       | npm token for publishing. Pass `${{ secrets.NPM_AUTH_TOKEN }}`. Used only with `npm-publish: true`. |

The job itself only has `contents: read`; all writes (version commit, tag, GitHub release, PR and issue comments) use the App token.

---

## Adopting the Workflows

### (a) npm Package (prompt-lib, prompt-shared-state)

Add the release job to the existing quality workflow, so it only runs after the quality job passes on `main`:

```yaml
# .github/workflows/build-package.yml
name: Quality Package

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  # Cancel outdated PR runs, but never cancel a release on main halfway.
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

permissions:
  contents: read

jobs:
  quality-package:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with:
          node-version: '24.x'
      - run: corepack enable
      - run: yarn install --immutable
      - run: yarn biome check
      - run: yarn test
      - run: yarn build

  release:
    needs: quality-package
    if: github.event_name == 'push'
    uses: prompt-edu/.github/.github/workflows/release.yml@main
    with:
      app-client-id: ${{ vars.RELEASE_APP_CLIENT_ID }}
      build-command: yarn build
    secrets:
      app-private-key: ${{ secrets.RELEASE_APP_PRIVATE_KEY }}
      npm-token: ${{ secrets.NPM_AUTH_TOKEN }}
```

With `cancel-in-progress: false` on `main`, a running release always finishes. If several merges arrive while one is running, only the newest waiting run is kept, and it releases all of their commits.

`.releaserc.json`:

```json
{
  "branches": ["main"],
  "plugins": [
    ["@semantic-release/commit-analyzer", { "preset": "conventionalcommits" }],
    ["@semantic-release/release-notes-generator", { "preset": "conventionalcommits" }],
    "@semantic-release/npm",
    [
      "@semantic-release/git",
      {
        "assets": ["package.json"],
        "message": "chore(release): ${nextRelease.version} [skip ci]"
      }
    ],
    "@semantic-release/github"
  ]
}
```

devDependencies: `semantic-release@^25`, `@semantic-release/git@^11` and `conventional-changelog-conventionalcommits@^9`. Keep the preset on 9: preset 10 needs `conventional-changelog-writer@9`, but the release-notes-generator of semantic-release 25 still renders with writer 8. The `conventionalcommits` preset is needed so that `feat!:` counts as a breaking change.

Also add the PR title caller from (c).

### (b) Tag and GitHub Release Only (prompt-sdk)

A Go module has no version file, so there is no npm publish and no commit back to `main`:

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  release:
    uses: prompt-edu/.github/.github/workflows/release.yml@main
    with:
      app-client-id: ${{ vars.RELEASE_APP_CLIENT_ID }}
      npm-publish: false
    secrets:
      app-private-key: ${{ secrets.RELEASE_APP_PRIVATE_KEY }}
```

If the release should wait for the tests on `main`, add the job to the test workflow instead, with `needs:` and `if: github.event_name == 'push'` as in (a).

`.releaserc.json` (no `@semantic-release/npm` and no `@semantic-release/git`):

```json
{
  "branches": ["main"],
  "plugins": [
    ["@semantic-release/commit-analyzer", { "preset": "conventionalcommits" }],
    ["@semantic-release/release-notes-generator", { "preset": "conventionalcommits" }],
    "@semantic-release/github"
  ]
}
```

A `v2.0.0` tag of a Go module is only installable once the module path ends in `/v2`. That change must land in the same PR as the breaking change.

Also add the PR title caller from (c).

### (c) PR Title Check Only (prompt)

```yaml
# .github/workflows/pr-title.yml
name: PR Title

on:
  pull_request:
    types: [opened, edited, reopened, synchronize]

permissions:
  pull-requests: read

jobs:
  pr-title:
    uses: prompt-edu/.github/.github/workflows/pr-title.yml@main
```

The `edited` type re-runs the check when only the title changes. No `.releaserc.json` is needed.

---

## Repository Settings

For every repository that adopts the workflows:

- **Merge button**: allow squash merging only (disable merge commits and rebase merging) and set the default squash commit message to **Pull request title**. In the API these are two settings: `squash_merge_commit_title: PR_TITLE` and `squash_merge_commit_message: BLANK`.
  - With `COMMIT_OR_PR_TITLE` as the title, a PR with a single commit uses that commit's message instead of the checked title.
  - With `COMMIT_MESSAGES` or `PR_BODY` as the body, the squash commit carries text the PR title check never sees. A `BREAKING CHANGE:` footer in a branch commit or the PR description would then release a major version. With `BLANK`, only the checked title decides the release, and a deliberate footer can still be typed in the merge dialog.
- **`main` ruleset**:
  - Require a pull request before merging.
  - Require the status check **`pr-title / conventional-commit`**. A repository that used a local PR title workflow before had a check with a different name; replace it in the ruleset.
  - Add the release GitHub App as a **bypass actor** (needed wherever semantic-release pushes a commit to `main`, i.e. with `@semantic-release/git`).
- **Renovate** (`renovate.json`): set `"semanticCommits": "enabled"` and remove any `commitMessagePrefix`, so that the defaults apply: `fix(deps)` for dependencies (patch release) and `chore(deps)` for dev dependencies and GitHub Actions (no release). Check which type grouped PRs get.
- **Secrets and variables**: the repository needs access to the `RELEASE_APP_CLIENT_ID` org variable and the `RELEASE_APP_PRIVATE_KEY` org secret. npm packages also need their own `NPM_AUTH_TOKEN` repository secret.
- **Baseline tag**: semantic-release computes the next version from the latest `vX.Y.Z` tag on `main`. That tag must exist before the first automated release, and `package.json` should show the same version. Without a tag, semantic-release releases `1.0.0`.

---

## Release GitHub App (Org Admins, One-Time Setup)

Releases are created with an org-owned GitHub App (e.g. `prompt-release-bot`) instead of `GITHUB_TOKEN`, because:

- the App can be a bypass actor on the `main` ruleset, so it can push the version commit, and
- releases and tags created with `GITHUB_TOKEN` do not trigger other workflows (e.g. `on: release`), while those created with an App token do.

Setup:

1. Create a GitHub App owned by the organization. A webhook is not needed; it can be installed on this account only.
2. Repository permissions:
   - **Contents: read and write**: version commit, tags and GitHub releases.
   - **Issues: read and write** and **Pull requests: read and write**: the `@semantic-release/github` plugin comments "released in vX.Y.Z" on included PRs and issues, adds a `released` label, and opens an issue when a release fails. The original plan (prompt-edu/.github#2) only listed contents. Repositories can turn these features off with the plugin's `successComment`, `failComment` and `releasedLabels` options, but the App's permissions are shared by all of them.
3. Install the App on the organization, for all repositories that release.
4. Generate a private key. Store the App's **client ID** as the org variable `RELEASE_APP_CLIENT_ID` and the **private key** as the org secret `RELEASE_APP_PRIVATE_KEY`, available to the releasing repositories.
5. Add the App as a bypass actor on the `main` ruleset of each repository that commits back to `main` (see [Repository Settings](#repository-settings)).

---

## npm Authentication

npm publishing currently uses a token: the caller passes its `NPM_AUTH_TOKEN` secret, and the workflow sets it as `NPM_TOKEN` for `@semantic-release/npm`, but only when `npm-publish` is `true`. Otherwise `NPM_TOKEN` is not set at all.

The switch to [npm trusted publishing](https://docs.npmjs.com/trusted-publishers) (OIDC, no token) is tracked in prompt-edu/prompt-lib#295. It needs `id-token: write` and `NPM_TOKEN` to be fully absent during publishing, and it is still open whether npm accepts a publish from a reusable workflow, since it may validate the calling workflow's filename.

If the npm publish fails after the version commit and the tag were pushed, publish the package from that tag by hand.
