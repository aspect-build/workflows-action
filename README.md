# Aspect Workflows runs Bazel fastest

Set up your GitHub Actions workflow to run Bazel using Aspect.

Aspect Workflows is a self-hosted runner infrastructure for
getting best-case performance of running Bazel on your CI/CD pipeline.

See https://docs.aspect.build/v/workflows for more documentation.

## Usage

This action depends on infrastructure that's deployed by Aspect Workflows.
First sign up for a trial: <https://aspect.build/workflows>

Then, edit your `.github/workflows/ci.yaml` file to use it.

-   Target the Aspect-managed runners with `runs-on`, using at least `self-hosted` and `aspect-workflows` tags.
-   Add a step, and put this action in a `uses` entry.

For example:

```yaml
jobs:
    bazel:
        runs-on: [self-hosted, aspect-workflows]
        steps:
            - uses: aspect-build/workflows-action@5.2.6
```
