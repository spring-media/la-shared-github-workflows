# la-shared-github-workflows

This repo hosts shared github workflows used by other repositories.

Please refer to githubs documentation on how to use and implement shared workflows: https://docs.github.com/en/actions/using-workflows/reusing-workflows

### Deployment Slack Notification Workflow

#### Usage

The Slack deployment notification consist of two extra jobs that need to be added to your deployment pipeline.

Add the job `inform-slack` to the end(!) of your github actions file.
Then you will and exchange the ID in `needs: [<ADD JOB ID>]` and in
`success: ${{ needs.<ADD JOB ID>.result == 'success' }}`.

The name of the deployment job in this example is `deploy`, so you will change the two lines with to `needs: [deploy]`
and `success: ${{ needs.deploy.result == 'success' }}` respectively.

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
    uses: spring-media/la-shared-github-workflows/.github/workflows/reusable-workflow__slack-notifications.yaml@main
    with:
      workflow: ${{ github.workflow }}
      repository: ${{ github.repository }}
      repositoryUrl: ${{ github.event.repository.url }}
      refName: ${{ github.ref_name }}
      success: ${{ needs.<ADD JOB ID>.result == 'success' }}
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```
