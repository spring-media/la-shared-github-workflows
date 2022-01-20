# la-shared-github-workflows
This repo hosts shared github workflows used by other repositories.

### Deployment Slack Notification Workflow
#### Usage
The Slack deployment notification consist of two extra jobs that need to be added to your deployment pipeline.

Add the two jobs `inform-slack-success` and `inform-slack-failure` to the end(!) of your github actions file.
Then you will and exchange the ID in each job's `needs: []` line.
The name of the deployment job in this example is `deploy`, so you will change the two lines with
`needs: [<ID OF DEPLOYMENT JOB>]` to `needs: [deploy]`.

```yaml
name: deploy

on:
  push:
    branches:
      - master

jobs:
  deploy:
    ... A lot of stuff usually happens here ...
  
  inform-slack-success:
    needs: [<ID OF DEPLOYMENT JOB>]
    if: ${{ success() }}
    uses: spring-media/la-shared-github-workflows/workflows/reusable-workflow__slack-notifications.yaml@main
    with:
      workflow: ${{ github.workflow }}
      repository: ${{ github.repository }}
      repositoryUrl: ${{ github.event.repository.url }}
      refName: ${{ github.ref_name }}
      status: 'success'
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

  inform-slack-failure:
    needs: [<ID OF DEPLOYMENT JOB>]
    if: ${{ failure() }}
    uses: spring-media/la-shared-github-workflows/workflows/reusable-workflow__slack-notifications.yaml@main
    with:
      workflow: ${{ github.workflow }}
      repository: ${{ github.repository }}
      repositoryUrl: ${{ github.event.repository.url }}
      refName: ${{ github.ref_name }}
      status: 'failure'
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
    


```

