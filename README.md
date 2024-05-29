# Reusable GitHub Actions Workflows

Collection of reusable GitHub Actions Workflows we use inside our org.

## `auto-merge-dependabot-pr.yml`

Workflow that can automatically merge Dependabot pull requests after the CI builds has run successfully.
Below is an example workflow you can use in your repository. The workflow is executed when the "Integrate" ran completed successfully.

```yaml
# .github/workflows/auto-merge.yml
name: Merge me!

on:
  workflow_run:
    types:
      - completed
    workflows:
      - 'Integrate'

jobs:
  merge-me:
    uses: 2media/reusable-actions-workflows/.github/workflows/auto-merge-dependabot-pr.yml@main
    secrets:
      MERGE_ME_GITHUB_TOKEN: ${{ secrets.MERGE_ME_GITHUB_TOKEN }}
```

## `backup-restore.yml`

Workflow to run [laravel-backup-restore](https://github.com/stefanzweifel/laravel-backup-restore) and restore a database backup in a Laravel app. The backup must be stored on an AWS S3 bucket.

The passed `app_name` will be used as `APP_NAME` and will be used to find the latest backup for your app.

```yml
# .github/workflows/backup-restore.yml
name: backup-restore

on:
  workflow_dispatch:
  schedule:
    - cron: "0 14 1 * *"

jobs:
  restore:
    uses: 2media/reusable-actions-workflows/.github/workflows/backup-restore.yml@main
    with:
      # Value of `APP_NAME` env variable. Used to locate backup on remote disk.
      app_name: 'My Laravel App'
      database_name: 'laravel_app'
      php_version: '8.3'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_DEFAULT_REGION: ${{ secrets.AWS_DEFAULT_REGION }}
      AWS_BACKUP_BUCKET: ${{ secrets.AWS_BACKUP_BUCKET }}
      BACKUP_ARCHIVE_PASSWORD: ${{ secrets.BACKUP_ARCHIVE_PASSWORD }}
```


## `image-optim.yml`

Workflow to compress images in a repository by using [calibreapp/image-actions](https://github.com/calibreapp/image-actions).yml
The example workflow below runs the Action when images are pushed to a pull request.

```yaml
# .github/workflows/image-optim.yml
name: Image Optimisation

on:
  pull_request:
    paths:
      - '**.jpg'
      - '**.jpeg'
      - '**.png'
      - '**.webp'

jobs:
  build:
    uses: 2media/reusable-actions-workflows/.github/workflows/image-optim.yml@main
```

## `integrate-landingpages-backend.yml` & `integrate-landingpages-frontend.yml`

The `integrate-landingpages-*` workflows can be used in our landingpage repositories to validate that code changes do not break the build.

The `backend`-workflow checks out the repository, installs composer dependencies and creates a new production Jigsaw build.   
The `frontend`-workflows checks out the repository, install NPM dependencies and creates a new production build of our frontend assets (CSS and JS). The job is only run when the commit has been made by Dependabot.

```yaml
# .github/workflows/integrate.yml
name: "Integrate"

on:
    pull_request: null
    push:
        branches:
            - "main"

jobs:
    build:
        uses: 2media/reusable-actions-workflows/.github/workflows/integrate-landingpages-backend.yml@main
        with:
          phpVersion: 8.1
        secrets:
          COMPOSER_GITHUB_TOKEN: ${{ secrets.COMPOSER_GITHUB_TOKEN }}

    frontend-build:
        uses: 2media/reusable-actions-workflows/.github/workflows/integrate-landingpages-frontend.yml@main
```

## `laravel-pint-fixer.yml`

Workflow to run Laravel Pint and automatically fix code style violations. Changes are pushed back to the GitHub repository.

```yaml
# .github/workflows/laravel-pint-fixer.yml
name: laravel-pint

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  laravel-pint:
    uses: 2media/reusable-actions-workflows/.github/workflows/laravel-pint-fixer.yml@main
```

## `nginx-file-watcher.yml`

Workflow that notifies us when a pull request updates a NGINX file. Should encourage a better code review.

```yaml
# .github/workflows/nginx-file-watcher.yml
name: NGINX File Watcher

on:
  pull_request:
    paths:
      - 'nginx/**'

jobs:
  nginx:
    uses: 2media/reusable-actions-workflows/.github/workflows/nginx-file-watcher.yml@main

```

## `on-demand-landingpage-build.yml`

Workflow to create a fresh build of a landingpage and push the build back to GitHub.   
The workflow can be used to trigger new builds manually from the GitHub UI, without cloning the repo to your local machine, or by creating a new build on a schedule.

The example below allows us to create new builds through the GitHub UI or through the GitHub API.

```yaml
# .github/workflows/manual-build.yml
name: Manual Build

on:
  workflow_dispatch:
  repository_dispatch:
    types: [manual_build]

jobs:
  build:
    uses: 2media/reusable-actions-workflows/.github/workflows/on-demand-landingpage-build.yml@main
    with:
      commit_message: 'Manually triggered build through GitHub Actions'
    secrets:
      COMPOSER_GITHUB_TOKEN: ${{ secrets.COMPOSER_GITHUB_TOKEN }}
```

The example workflow below creates a new build at the beginning of the year.

```yaml
# .github/workflows/scheduled-build.yml
name: New Yearly Build

on:
  schedule:
  - cron: '0 1 1 1 *'

jobs:
  build:
    uses: 2media/reusable-actions-workflows/.github/workflows/on-demand-landingpage-build.yml@main
    with:
      commit_message: 'Yearly Build'
    secrets:
      COMPOSER_GITHUB_TOKEN: ${{ secrets.COMPOSER_GITHUB_TOKEN }}
```

## `php-cs-fixer.yml`

Workflow to run `php-cs-fixer` and automatically fix code style violations. Changes are pushed back to the GitHub repository.

```yaml
# .github/workflows/php-cs-fixer.yml
name: php-cs-fixer

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  php-cs-fixer:
    uses: 2media/reusable-actions-workflows/.github/workflows/php-cs-fixer.yml@main
```

## `rector.yml`

Workflow to run [Rector](https://getrector.com/) in our apps.
To prevent potential breaking changes, the workflow will create a new pull request with the changes made by Rector.

We suggest you run the workflow on a schedule. Like for example once a month.

```yaml
# .github/workflows/rector.yml
name: Rector

on:
  schedule:
  - cron: '15 2 3 * *'

jobs:
  rector:
    uses: 2media/reusable-actions-workflows/.github/workflows/rector.yml@main
```
## `release-drafter.yml`

Workflow to run [release-drafter](https://github.com/release-drafter/release-drafter) in our apps.
To prevent usage spikes, the job is only executed for non Dependabot users.

```yaml
# .github/workflows/release-drafter.yml
name: Release Drafter

on:
  push:
    branches:
      - main

jobs:
  update_release_draft:
    uses: 2media/reusable-actions-workflows/.github/workflows/release-drafter.yml@main
```

## `sentry-release-tracking.yml`

Workflow to notify Sentry about a new [release](https://docs.sentry.io/product/releases/).

The example workflow below is triggerd by a [deployer](https://deployer.org/) deployment where an API request to the GitHub API is made with the `deployer_deployment` event type.

```yaml
# .github/workflows/sentry-release-tracking.yml
name: Sentry Release Tracking

on:
  repository_dispatch:
    types: [deployer_deployment]

jobs:
  track:
    uses: 2media/reusable-actions-workflows/.github/workflows/sentry-release-tracking.yml@main
    with:
      project_name: 'todo-app'
    secrets:
      SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
```

## `update-changelog.yml`

Workflow to automatically update a projects `CHANGELOG.md` with the release notes of a new GitHub release.

```yaml
# .github/workflows/update-changelog.yml
name: "Update Changelog"

on:
  release:
    types: [released]

jobs:
  update:
    uses: 2media/reusable-actions-workflows/.github/workflows/update-changelog.yml@main
```
