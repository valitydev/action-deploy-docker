# `action-deploy-docker` - **Github Action**

This action deploy docker image

## Input

| Name                 | Description                        |
| -------------------- | ---------------------------------- |
| `registry-username`  | Username for image registry        |
| `registry-password`  | Password for image registry        |
| `docker-registry`    | Docker image registry              |
| `aws-ecr-access-key` | Amazon ECR access key              |
| `aws-ecr-secret-key` | Amazon ECR secret key              |
| `aws-region`         | AWS region                         |
| `context-path`       | Build's context path               |
| `dokerfile-path`     | Path to Dockerfile                 |
| `platforms`          | List of target platforms for build |

## Example Workflow File

```yaml
name: Deploy docker image

on: [pull_request]

jobs:
  deploy:
    runs-on: ubuntu-latest
      steps:
        uses: valitydev/action-deploy-docker@v2
```
