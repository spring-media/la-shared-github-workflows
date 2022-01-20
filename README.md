# la-shared-github-workflows
This repo hosts shared github workflows used by other repositories.

### Deployment Slack Notification Workflow
#### Usage
The Slack deployment notification consist of one extra job that need to be added to your deployment pipeline.

Add the job `inform-slack` to the end(!) of your github actions file.
Then you will and exchange the ID in the job's `needs: []` line.
The name of the deployment job in this example is `deploy`, so you will change the the line with
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
    uses: spring-media/la-shared-github-workflows/workflows/reusable-workflow__slack-notifications.yaml@main
    with:
      workflow: ${{ github.workflow }}
      repository: ${{ github.repository }}
      repositoryUrl: ${{ github.event.repository.url }}
      refName: ${{ github.ref_name }}
      success: ${{ success() }}
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

```

