# la-shared-github-workflows

This repo hosts shared github workflows used by other repositories.

Please refer to githubs documentation on how to use and implement shared workflows: https://docs.github.com/en/actions/using-workflows/reusing-workflows

### Workflows

- [js\_\_deploy-to-s3](#js__deploy-to-s3)
- [js\_\_format-lint-test](#js__format-lint-test)
- [slack-notify-after-production-deploy](#slack-notify-after-production-deploy)

## js\_\_deploy-to-s3

The [js\_\_deploy-to-s3 workflow](.github/workflows/reusable-workflow__js__deploy-to-s3.yaml) builds a js project and deploys the resulting files
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
    needs: format-lint-and-unit-tests
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__deploy-to-s3.yaml@v1
    with:
      cloudfront_distribution_id: E26N41SPUCWIRP
      cloudfront_paths: /
      build_script_name: build-staging
      aws_bucket: s3://hua-mod-web.staging.la.welt.de/${{ github.head_ref || github.ref_name }}
      build_directory: build
      branch_deploy: true
      branch_name: ${{ github.head_ref || github.ref_name }}
      branch_deploy_base_url: https://hua-mod-web.staging.la.welt.de
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.access_key_id }}
      AWS_ACCESS_KEY_SECRET: ${{ secrets.access_key_secret }}
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
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__format-lint-test.yml@main
    secrets:
      NPM_AUTH_TOKEN: ${{secrets.NPM_AUTH_TOKEN}}
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
```

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
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__slack-notify-after-production-deploy.yaml@main
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
