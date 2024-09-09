## Create a New Repository on GitHub

1. Go to GitHub and log in to your account.
2. Click the `+` icon in the top right corner and select `New repository`.
3. Fill out the repository name and description, then click `Create repository`.
 
## Add GitHub as a Remote

Add your newly created GitHub repository as a remote:

```
git remote add github https://github.com/username/repository-name.git
```

## Create a GitHub Personal Access Token:

1. Go to GitHub [Personal Access Tokens](https://github.com/settings/tokens).
2. Click `Generate new token`.
3. Add a note, select the `repo` scope for full control of private repositories, and any other necessary scopes.
4. Click `Generate token` and copy the token. This will be used as your GitHub authentication token.

## Push Your Repository to GitHub

Push your repository’s branches to GitHub:

```
git push github --all
```

## Create a GitHub Action to Build and Push Docker Images

In your GitHub repository, create the necessary directory for GitHub Actions workflows:

```
mkdir -p .github/workflows
```

Create a new file in the `.github/workflows` directory, e.g., `build-and-push.yml`:

```
name: Build and push docker image to Docker Hub

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

  workflow_dispatch:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v4
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Login to DockerHub
        uses: docker/login-action@v2 
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - 
        name: Docker meta
        id: docker_meta
        uses: crazy-max/ghaction-docker-meta@v2
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/slat
          tags: |
            type=sha
            type=semver,pattern={{raw}}
            type=ref,event=branch
      - 
        name: Build & Push image
        uses: docker/build-push-action@v2
        with:
          context: .
          file: Dockerfile
          push: true
          tags: ${{ steps.docker_meta.outputs.tags }}
          labels: ${{ steps.docker_meta.outputs.labels }}
```

## Add Secrets to Your GitHub Repository:

- Go to your GitHub repository and navigate to `Settings`.
- Select `Secrets and variables` → `Actions`.
- Click `New repository secret` and add the following secrets:
  * `DOCKERHUB_USERNAME`: Your Docker Hub username.
  * `DOCKERHUB_PASSWORD`: Your Docker Hub password or access token.

## Commit and Push the Workflow File

Add, commit, and push the workflow file to your GitHub repository:

```
git add .github/workflows/build-and-push.yml
git commit -m "Add GitHub Actions workflow for Docker build and push"
git push
```


