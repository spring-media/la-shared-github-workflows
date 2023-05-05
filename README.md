# la-shared-github-workflows

This repo hosts shared github workflows used by other repositories.

Please refer to githubs documentation on how to use and implement shared workflows: https://docs.github.com/en/actions/using-workflows/reusing-workflows

### Workflows

- [aws-remove-bucket](#aws-remove-bucket)
- [js\_\_build-and-deploy](#js__build-and-deploy)
- [js\_\_deploy-to-s3](#js__deploy-to-s3)
- [js\_\_format-lint-test](#js__format-lint-test)
- [js\_\_run-e2e-tests](#js__run-e2e-tests)
- [golang\_\_format-unit-tests](#golang__format-unit-tests)
- [slack-notify-after-production-deploy](#slack-notify-after-production-deploy)

## aws-remove-bucket

[aws-remove-bucket workflow](.github/workflows/reusable-workflow__aws-remove-bucket.yaml) removes a given bucket from S3. We need this for cleanup of branch deploys to S3 buckets after they got merged.

### Usage

```yaml

jobs:
  …
  cleanup-after-merge:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__aws-remove-bucket.yaml@v1
    with:
      aws_bucket: s3://hua-mod-web.staging.la.welt.de/${{ github.head_ref || github.ref_name }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.access_key_id }}
      AWS_ACCESS_KEY_SECRET: ${{ secrets.access_key_secret }}
  …
```

## js\_\_build-and-deploy

The [js\_\_build-and-deploy workflow](./.github/workflows/reusable-workflow__js__build-and-deploy.yml) runs the given
build and deploy npm scripts to build and deploy a project

Required parameters

- `build_script_name` (string) – the npm command to execute the build.
- `deploy_script_name` (string) – the npm command to execute the deploy.

Required secrets:

- `AWS_ACCESS_KEY_ID`
- `AWS_ACCESS_KEY_SECRET`
- `LA_TECH_USER_AUTH_TOKEN`
- `NPM_AUTH_TOKEN`

### Usage

```yaml
jobs:
  …
  deploy-branch:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__build-and-deploy.yml@v1
    with:
      build_script_name: <the npm script to build the project>
      deploy_script_name: <the npm script to deploy the project>
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.access_key_id }}
      AWS_ACCESS_KEY_SECRET: ${{ secrets.access_key_secret }}
      NPM_AUTH_TOKEN: ${{secrets.NPM_AUTH_TOKEN}}
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
```

## js\_\_deploy-to-s3

The [js\_\_deploy-to-s3 workflow](./.github/workflows/reusable-workflow__js__deploy-to-s3.yaml) builds a js project and deploys the resulting files
to a given S3 bucket. It can also be used for branch based deployments of frontend projects.

Required variables to pass in

- `cloudfront_distribution_id` (string) – the Cloudfront distribution id that needs to be invalidated to serve the latest content
- `cloudfront_paths` (string) – the Cloudfront paths that should be invalidated
- `build_script_name` (string) – the npm command to execute the build
- `build_directory` (string) – the output directory the executed build saves its files to
- `aws_bucket` (string) – the aws bucket to deploy to

If using it for branch based deploy you also need to pass in:

- `branch_deploy` (boolean) – Whether the deploy should be branch based (post a comment to the PR after the deploy and sets environment variables when building to set the paths properly in projects)
- `branch_name` – the name of the current branch
- `branch_deploy_base_url` – The base url for the deployed branches

These are the required secrets:

- `AWS_ACCESS_KEY_ID`
- `AWS_ACCESS_KEY_SECRET`
- `LA_TECH_USER_AUTH_TOKEN`
- `NPM_AUTH_TOKEN`

### Usage

```yaml
jobs:
  …
  deploy-branch:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__deploy-to-s3.yaml@v1
    with:
      cloudfront_distribution_id: E26N41SPUCWIRP
      cloudfront_paths: /*
      build_script_name: build-staging
      aws_bucket: s3://hua-mod-web.staging.la.welt.de/${{ github.head_ref || github.ref_name }}
      build_directory: build
      branch_deploy: true
      branch_name: ${{ github.head_ref || github.ref_name }}
      branch_deploy_base_url: https://hua-mod-web.staging.la.welt.de
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.access_key_id }}
      AWS_ACCESS_KEY_SECRET: ${{ secrets.access_key_secret }}
      NPM_AUTH_TOKEN: ${{secrets.NPM_AUTH_TOKEN}}
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
```

## js\_\_format-lint-test

The [js\_\_format-lint-test workflow](./.github/workflows/reusable-workflow__js__format-lint-test.yml) will install dependencies and then run these npm commands on the project:

- `install`
- `format-check`
- `lint`
- `test`

If the projects `package.json` doesn't provide all of these scripts, the workflow will fail.

It also requires to secrets to be passed in:

- `NPM_AUTH_TOKEN`
- `LA_TECH_USER_AUTH_TOKEN`

The node version has to be set via a `.node-version` file.

### Usage

```yaml
jobs:
  …
  format-lint-and-unit-test:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__format-lint-test.yml@v1
    secrets:
      NPM_AUTH_TOKEN: ${{secrets.NPM_AUTH_TOKEN}}
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
```

## js\_\_run-e2e-tests

[js\_\_run-e2e-tests workflow](./.github/workflows/reusable-workflow__js__run-e2e-tests.yaml) will setup up a job that will run all the cypress tests configured in your project

### Usage

```yaml
jobs:
  …
  run-e2e-tests:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__run-e2e-tests.yaml@v1
    secrets:
      CYPRESS_LOGINUSER: ${{ secrets.WELT_MODERATOR_USER_USERNAME }}
      CYPRESS_LOGINPWD: ${{ secrets.WELT_MODERATOR_USER_PASSWORD }}
      NPM_AUTH_TOKEN: ${{secrets.NPM_AUTH_TOKEN}}
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
  …
```

## golang\_\_format-unit-tests

The [golang\_\_format-unit-tests workflow](./.github/workflows/reusable-workflow__golang__format-unit-tests.yml) will set up golang version and format & run unit tests on the project:

### Usage

```yaml
jobs:
  …
  format-and-unit-tests:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__golang__format-unit-tests.yml@v1
    with:
      go_version: <the go version of the project>
    secrets:
      ACCESS_KEY_ID: ${{ secrets.access_key_id }}
      ACCESS_KEY_SECRET: ${{ secrets.access_key_secret }}
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
      LA_TECH_USER_SSH_KEY: ${{ secrets.la_tech_user_ssh_key }}



## slack-notify-after-production-deploy

The [slack-notify-after-production-deploy workflow](./.github/workflows/reusable-workflow__slack-notify-after-production-deploy.yaml) notifies our slack channel `#t_loyalty_notifications` whether a deployment was successful or not.

### Usage

The Slack deployment notification consists of an extra job that needs to be added to your deployment pipeline.

Add the job `inform-slack` to the end(!) of your github actions file.
Then you will and exchange the ID in `needs: [<ADD JOB ID>]` and in
`success: ${{ needs.<ADD JOB ID>.result == 'success' }}`.

The name of the deployment job in this example is `deploy`, so you will change the two lines with to `needs: [deploy]`
and `success: ${{ needs.deploy.result == 'success' }}` respectively.

The job handles every case, by: `if: ${{ always() }}` it will always run, no matter the state of the jobs before, the
line `needs: [<ADD JOB ID>]` makes it wait until the deployment job has finished (in any way) and
`success: ${{ needs.<ADD JOB ID>.result == 'success' }}` hands in a boolean, `true` in case the run was successful,
`false` in case it was a failure, skipped or cancelled.

```yaml
name: deploy

on:
  push:
    branches:
      - master

jobs:
  deploy: ... A lot of stuff usually happens here ...

  slack-notify-after-production-deploy:
    needs: [<ADD JOB ID>]
    if: ${{ always() }}
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__slack-notify-after-production-deploy.yaml@v1
    with:
      workflow: ${{ github.workflow }}
      repository: ${{ github.repository }}
      repositoryUrl: ${{ github.event.repository.url }}
      refName: ${{ github.ref_name }}
      success: ${{ needs.<ADD JOB ID>.result == 'success' }}
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

## Releasing

Versioned releases for workflows are done via tags. The whole process is described here: https://github.com/actions/toolkit/blob/main/docs/action-versioning.md

Since all the shared actions live in one repository, we only have a unified tag for all workflows.
The current version is `v1`. Should you make any major breaking changes please increment the major version number.
For patch and minor upgrades please consider creating a new semver release as well as updating the
major version tag so that all actions. This is best practice and described in the document linked above.

```
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```

Please note that this is currently a manual process. This might change in the future.
