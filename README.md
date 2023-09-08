> [!WARNING]  
> This repo is now archived and the code has been [moved](https://github.com/AssemblyAI/DeepLearning/blob/master/.github/workflows/.aspect-workflows-reusable.yaml) to the DL repo.

# Aspect Workflows runs Bazel fastest

Set up your GitHub Actions workflow to run Bazel using Aspect.

Aspect Workflows is a self-hosted runner infrastructure for
getting best-case performance of running Bazel on your CI/CD pipeline.

See https://docs.aspect.build/v/workflows for more documentation.

## Setup

This action depends on infrastructure that's deployed by Aspect Workflows.
First sign up for a trial: <https://aspect.build/workflows>

GitHub Actions has a critical restriction: you cannot re-use a workflow definition from another
GitHub org and also target self-hosted runners.

From [GitHub docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-self-hosted-runners):

> Called workflows that are owned by the same user or organization as the caller workflow can access
> self-hosted runners from the caller's context.

For this reason, we recommend you fork this repository into your GitHub org.
Alternatively, you can vendor the file into your monorepo by copying
`.github/workflows/aspect-workflows.yaml` into the same path in your repo.

## Usage

Edit your CI workflow, e.g. `.github/workflows/ci.yaml` to use the reusable workflow.
It reads your `.aspect/workflows/config.yaml` to understand your Bazel CI preferences for this repo.

If you forked the repo to your org, then replace `my-org` with your org in this snippet:

```yaml
jobs:
  aspect-workflows:
    name: Aspect Workflows
    uses: my-org/workflows-action/.github/workflows/aspect-workflows.yaml@5.5.0
```

If you vendored the file, then instead it will be:

```yaml
jobs:
  aspect-workflows:
    name: Aspect Workflows
    uses: ./.github/workflows/aspect-workflows.yaml
```

You may want to start out with Aspect Workflows only triggering on certain branches during the trial.
You can use an `if` statement like the following to run on `main` and on pull requests coming from a branch named `aspect-build/*`.

```yaml
jobs:
  aspect-workflows:
    if: github.ref == 'refs/heads/main' || startsWith(github.head_ref, 'aspect-build/')
```

## Continuous delivery

See https://docs.aspect.build/v/workflows/delivery for an overview of how Continuous Delivery is
modeled in Aspect Workflows.

To run a delivery job with GitHub Actions, create another workflow file.
By default we look for `delivery.yaml`.

See the `delivery.yaml` file in this repository for an example.
Copy this file into your `.github/workflows` folder, then modify as needed.

For example, you might need to run a step that does authentication, using a GitHub Action like
`aws-actions/configure-aws-credentials` or `docker/login-action`.

## Slack notifications

You can get a notification when a build fails on a release branch.
Then your oncall can acknowledge the problem and work with code owners to quickly revert.

Confusingly, we're going to use a Slack feature that's also called "workflows".
You can read about it in the [Slack docs](https://slack.com/help/articles/360053571454-Set-up-a-workflow-in-Slack).

### 1. Create the Slack Workflow

1. In slack, click your workspace name in the upper-left.
1. Select _Tools_ from the menu.
1. Select _Workflow Builder_
1. In the pop-up window, click _Create_ in the top right.
1. Enter a name, for example _Github Actions Buildcop_.
1. In the next dialog, select _Webhook_ to start this workflow.
1. Click _Add Variable_ and use the key `gha_url` with a _Data type_ of _Text_.
1. Click _Next_.
1. Click _Add Step_. You can choose what to do, for example, _Send a message_.
   You'll be able to add the `gha_url` variable in the message.
   This is will be filled in with a link back to the broken build on GitHub Actions.
1. Click _Publish_. Copy the resulting webhook URL.

### 2. Provide the webhook URL to GitHub Actions

1. Choose whether the secret will be in the Organization settings or the Repository settings.
1. In the GitHub UI, add a secret in the settings.
   See the [GitHub docs](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository).
1. We suggest naming the secret `SLACK_WEBHOOK_URL`. The value should be the webhook URL you copied earlier.

### 3. Configure Aspect Workflows

1. Add a `secrets` section to the `aspect-workflows` job in your `ci.yaml` file.
   It should look like this:

```yaml
jobs:
  aspect-workflows:
    ...
    secrets:
      slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```
