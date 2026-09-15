This directory holds starter workflows — essentially, Github actions that can be leveraged in multiple repositories to promote consistency across the entire organization in a DRY manner. They must reside in this `.github` repository in a directory named `workflow-templates`. Ref https://docs.github.com/en/actions/learn-github-actions/creating-starter-workflows-for-your-organization for additional information about how to add a new starter workflow.

After adding a new starter workflow to this directory, you'll most likely want to use it in a different repository. See https://docs.github.com/en/actions/learn-github-actions/using-starter-workflows for how to accomplish this.

## Runner requirements

Every starter workflow runs on the isolated ARC runners (`arm-isolated-{sm,md,lg}`), where
workflow steps execute inside a job pod the Kubernetes API creates. The docker-in-docker
labels (`runner-*-ng`, `arm-runner-*`) are retired; do not use them in new templates.

Each job must declare a `container:` — the isolated scale sets set
`ACTIONS_RUNNER_REQUIRE_JOB_CONTAINER=true`, and a job without one never starts. For a job
that needs no image of its own, use the multi-arch runner image:

```yaml
runs-on: arm-isolated-sm
container:
  image: 365342630876.dkr.ecr.us-east-1.amazonaws.com/actions-runner:v1.0.6
```

Pull container images from the ECR pull-through cache
(`365342630876.dkr.ecr.us-east-1.amazonaws.com/remote-cache/...`), not from a public
registry directly; the runner fleet egresses through shared addresses that upstream
registries rate-limit. The pull-through cache serves the upstream multi-arch index, so
arm64 nodes resolve a matching image.
