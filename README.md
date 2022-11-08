# action.setup-tester

GitHub composite Action to check if a Layer tester in a Dockerfile exists. If the Layer exists the action will set up docker buildx for the tester Layer, pushes the image to ghcr with the tag tester and runs the testimage

## example

```yaml
name: Build Docker Image

on:
  push:
    branches:
      - develop

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Login to Github Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Set up docker buildx for tester
        id: tester
        env:
          PAT: ${{  secrets.PAT  }}
        uses: entrecode/action.setup-tester@v1
        with:
          token: ${{ env.PAT }}
          repository: ${{  github.repository  }}

      - name: Build and push
        if: steps.tester.outputs.tester == 'failure' || steps.tester.outputs.testrunner == 'success'
        uses: docker/build-push-action@v3
        with:
          context: .
          file: ./Dockerfile
          push: true
          target: runner
          tags: ghcr.io/${{ github.repository }}:${{ env.IMAGE_TAG }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            PAT=${{ secrets.PAT }}
```

## inputs

All inputs are requried to run the action

| Name               | Type     | Description                                                      |
|--------------------|----------|------------------------------------------------------------------|
| `token`            | String   | GitHub secret to run docker build command (e.g., `secrets.PAT`)  |
| `repository`       | String   | GitHub Repository to push docker image                           |

## outputs

The outpus are used to check if a layer tester exists or if the tests run successfully

| Name               | Type     | Description                                                      |
|--------------------|----------|------------------------------------------------------------------|
| `tester`           | String   | outcome of the tester (`success` or `failure` )                  |
| `testrunner`       | String   | outcome of the tesrunner (`success` or `failure` )               |
