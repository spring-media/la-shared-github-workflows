# la-shared-github-workflows

This repo hosts shared github workflows used by other repositories.

Please refer to githubs documentation on how to use and implement shared workflows: https://docs.github.com/en/actions/using-workflows/reusing-workflows

### Workflows

- [aws-remove-bucket](#aws-remove-bucket)
- [js\_\_build-and-deploy](#js__build-and-deploy)
- [js\_\_deploy-to-s3](#js__deploy-to-s3)
- [js\_\_format-lint-test](#js__format-lint-test)
- [js\_\_security](#js__security)
- [js\_\_run-e2e-tests](#js__run-e2e-tests)
- [golang\_\_security](#golang__security)
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

- `branch_deploy` (boolean) – Whether the deploy should be branch based (sets environment variables when building to set the paths properly in projects)
- `comment_branch_link` (boolean) - Wether the branch link should be posted as a comment to the PR after the deploy
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

## js\_\_security

The [js\_\_security](./.github/workflows/reusable-workflow__js__security.yml) will 

- install snyk and check for vulnerabilities on the project
- check for leaked secrets using [trufflehog](https://github.com/trufflesecurity/trufflehog)

Required secrets:

- `LA_SNYK_TOKEN`

Optional inputs:

- `snyk-severity-threshold` can be passed as input. It's not required but with this option, only vulnerabilities of the specified level or higher will be checked.

### Usage

```yaml
jobs:
  …
  security:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__js__security.yml@v1
     secrets:
       LA_SNYK_TOKEN: ${{ secrets.LA_SNYK_TOKEN }}
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

## golang\_\_security

The [golang\_\_security workflow](./.github/workflows/reusable-workflow__golang__security.yml) will 

- set up snyk and check vulnerabilities on the project
- check for leaked secrets using [trufflehog](https://github.com/trufflesecurity/trufflehog)

Required secrets:

- `LA_TECH_USER_AUTH_TOKEN`
- `LA_TECH_USER_SSH_KEY`
- `LA_SNYK_TOKEN`

Required inputs:
- `go-version` the go version of the project

Optional inputs:
- `snyk-severity-threshold` can be passed as input. It's not required but with this option, default value is critical, only vulnerabilities of the specified level or higher will be checked.

### Usage

```yaml
jobs:
  …
  security:
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__golang__security.yml@v1
    with:
      snyk-severity-threshold: <vulnerabilities threshold not required and default value is critical>
      go-version: <the go version of the project>
    secrets:
      LA_TECH_USER_AUTH_TOKEN: ${{ secrets.LA_TECH_USER_AUTH_TOKEN }}
      LA_TECH_USER_SSH_KEY: ${{ secrets.LA_TECH_USER_SSH_KEY }}
      LA_SNYK_TOKEN : ${{secrets.LA_SNYK_TOKEN}}
  …
```

## golang\_\_format-unit-tests

The [golang\_\_format-unit-tests workflow](./.github/workflows/reusable-workflow__golang__format-unit-tests.yml) will set up golang version and check formatted properly and run unit tests on the project.

Rquired secrets

- `ACCESS_KEY_ID`
- `ACCESS_KEY_SECRET`
- `LA_TECH_USER_AUTH_TOKEN`
- `LA_TECH_USER_SSH_KEY`

Required parameters
- `go-version` the go version of the project


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
      LA_TECH_USER_SSH_KEY: ${{ secrets.LA_TECH_USER_SSH_KEY }}
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
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__slack-notify-after-production-deploy.yaml@v1
    with:
      workflow: ${{ github.workflow }}
      repository: ${{ github.repository }}
      repositoryUrl: ${{ github.event.repository.html_url }}
      refName: ${{ github.ref_name }}
      success: ${{ needs.<ADD JOB ID>.result == 'success' }}
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

## Releasing

Whenever any shared workflow is updated or added, we must release a new version. Having the workflows versioned allows users of the shared workflows to bind to
a specific version of an action.

Semver version number update guide for the shared github action workflows:

- Patch version: Fixing a problem with an existing action without changing anything (e.g. `v1.5.6` ➡️ `v1.5.7`)
- Minor version: Adding new actions, adding functionality to actions without breaking existing functionality (e.g. `v1.5.6` ➡️ `v.1.6.0`)
- Major versions: Any breaking change to existing actions (e.g. `v1.5.6` ➡️ `v2.0.0`)

Releasing a new version is currently a manual process: https://github.com/spring-media/la-shared-github-workflows/releases/new

In addition to the semver version release we also have a major version tag that tracks the latest semver version. 
This allows user to bing to the latest major version of the shared workflows and automatically get all improvements, but no 
breaking changes.

When you release a new version you must update the `v1` tag like so

```
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```

This whole process is best practice and described here: https://github.com/actions/toolkit/blob/main/docs/action-versioning.md
