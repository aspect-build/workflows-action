# Aspect Workflows runs Bazel fastest

Set up your GitHub Actions workflow to run Bazel using Aspect.

Aspect Workflows is a self-hosted runner infrastructure for
getting best-case performance of running Bazel on your CI/CD pipeline.

See https://docs.aspect.build/workflows for more documentation.

## Setup

This action depends on infrastructure that's deployed by Aspect Workflows.
First sign up for a trial: <https://aspect.build/workflows>

GitHub Actions has a critical restriction: you cannot re-use a workflow definition from another
GitHub org and also target self-hosted runners.

From [GitHub docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-self-hosted-runners):

> Called workflows that are owned by the same user or organization as the caller workflow can access
> self-hosted runners from the caller's context.

To work around this, either:

1.  Vendor the file into the repository by copying `.github/workflows/.aspect-workflows-reusable.yaml` to the same location.
2.  Fork this repository into your GitHub organization

## Usage

Start from a standard GitHub Actions workflow file and reference Aspect Workflow's reusable workflow file.

This may be a new or existing file; for example many repositories have an existing `.github/workflows/ci.yaml` file.

The reusable workflow is configured via the `.aspect/workflows/config.yaml` file.

If you forked the repo to your org, then replace `my-org` with your org in this snippet:

```yaml
jobs:
    aspect-workflows:
        name: Aspect Workflows
        uses: my-org/workflows-action/.github/workflows/.aspect-workflows-reusable.yaml@5.12.22
```

If you vendored the file, then instead it will be:

```yaml
jobs:
    aspect-workflows:
        name: Aspect Workflows
        uses: ./.github/workflows/.aspect-workflows-reusable.yaml
```

You may want to start out with Aspect Workflows only triggering on certain branches during the trial.
You can use an `if` statement like the following to run on `main` and on pull requests coming from a branch named `aspect-build/*`.

```yaml
jobs:
    aspect-workflows:
        if: github.ref == 'refs/heads/main' || startsWith(github.head_ref, 'aspect-build/')
```

## Continuous Delivery

See https://docs.aspect.build/workflows/configuration/delivery for a high-level overview of the CD process.

To install it, add another YAML file to create a new GitHub Actions Workflow, commonly
`.github/workflows/aspect-workflows-delivery.yaml`.

First, allow manual delivery to be triggered in the GitHub Actions web UI, collecting two user inputs (the commit SHA and targets to deliver):

```yaml
on:
    workflow_dispatch:
        inputs:
            delivery_commit:
                description: The commit to checkout and run the delivery from. Targets listed in the delivery manifest for this commit will be delivered unless specific targets are listed in `delivery_targets`.
                type: string
                required: true
            workspace:
                description: The workspace to deliver from
                type: string
                required: false
                default: "."
            delivery_targets:
                description: List of Bazel targets to deliver, delimited by spaces. For example, \`//app/a:push_release //app/b:push_release\`. If empty, targets listed in the delivery manifest for the target commit will be delivered.
                type: string
                required: false
```

Now define the job. This has a few steps:

1. Configure the environment.
2. Checkout the repository at the commit to be delivered.
3. Check that the build agent is healthy.
4. Run Aspect Workflows 'composite action' with the `delivery` task, providing the user inputs via environment variables.

```yaml
jobs:
    delivery:
        name: Aspect Workflows Delivery
        runs-on: [self-hosted, aspect-workflows, aspect-default]
        steps:
            - name: Workflows environment
              run: /etc/aspect/workflows/bin/configure_workflows_env
            - uses: actions/checkout@v4
              with:
                  ref: ${{ inputs.delivery_commit }}
                  fetch-depth: 0
            - name: Agent health check
              run: /etc/aspect/workflows/bin/agent_health_check
            - name: Run delivery
              uses: aspect-build/workflows-action@5.12.22
              with:
                  task: delivery
                  workspace: ${{ inputs.workspace }}
              env:
                  DELIVERY_COMMIT: ${{ inputs.delivery_commit }}
                  DELIVERY_TARGETS: ${{ inputs.delivery_targets }}
```

A full, operational example of this file: <https://github.com/aspect-build/rules_jasmine/blob/main/.github/workflows/aspect-workflows-delivery.yaml>

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
