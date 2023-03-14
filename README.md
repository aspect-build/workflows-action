# Run Bazel with Aspect Workflows for GitHub Actions

This composite action sets up Aspect Workflows, which is a self-hosted runner infrastructure for
getting best-case performance of running Bazel on your CI/CD pipeline.

See https://docs.aspect.build/v/workflows for more documentation.

## Usage

This action depends on infrastructure that's deployed by Aspect Workflows.
First sign up for a trial: <https://aspect.build/workflows>

Then, edit your `.github/workflows/ci.yaml` file to use it, for example:

```yaml
jobs:
    run-workflows:
        runs-on: [self-hosted, aspect-workflows]
        steps:
            # Check for the latest tag when you install
            - uses: aspect-build/workflows-action@v5.2.3
```
