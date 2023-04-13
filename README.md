# Aspect Workflows runs Bazel fastest

Set up your GitHub Actions workflow to run Bazel using Aspect.

Aspect Workflows is a self-hosted runner infrastructure for
getting best-case performance of running Bazel on your CI/CD pipeline.

See https://docs.aspect.build/v/workflows for more documentation.

## Usage

This action depends on infrastructure that's deployed by Aspect Workflows.
First sign up for a trial: <https://aspect.build/workflows>

Then, edit your `.github/workflows/ci.yaml` file to use our reusable workflow.
It reads your `.aspect/workflows/config.yaml` to understand your Bazel CI preferences for this repo.

```yaml
jobs:
  aspect-workflows:
    name: Aspect Workflows
    uses: aspect-build/workflows-action/.github/workflows/aspect-workflows.yaml@5.3.3
```
