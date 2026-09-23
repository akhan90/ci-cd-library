# CI/CD Library

Reusable GitHub Actions for CI/CD pipelines.

## Available Actions

| Action | Description |
|--------|-------------|
| `init` | YAML validation using yamllint |
| `build` | Gradle compile |
| `image` | Docker image build with Gradle Jib |
| `deploy` | Deploy to Amazon EKS |

## Usage

```yaml
name: CI/CD Pipeline

on:
  push:
  pull_request:

jobs:
  # Runs on ALL branches
  init:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: akhan90/ci-cd-library/.github/actions/init@main
        with:
          yaml-paths: './k8s'

  # Runs on ALL branches
  build:
    runs-on: ubuntu-latest
    needs: init
    steps:
      - uses: actions/checkout@v4
      - uses: akhan90/ci-cd-library/.github/actions/build@main
        with:
          java-version: '21'

  # Runs ONLY on dev/main push
  image:
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'push' && (github.ref == 'refs/heads/dev' || github.ref == 'refs/heads/main')
    steps:
      - uses: actions/checkout@v4
      - uses: akhan90/ci-cd-library/.github/actions/image@main
        with:
          image-registry: 'your-registry.ecr.aws'
          image-name: 'my-service'
          image-tag: ${{ github.sha }}

  # Runs ONLY on dev/main push
  deploy:
    runs-on: ubuntu-latest
    needs: image
    if: github.event_name == 'push' && (github.ref == 'refs/heads/dev' || github.ref == 'refs/heads/main')
    steps:
      - uses: actions/checkout@v4
      - uses: akhan90/ci-cd-library/.github/actions/deploy@main
        with:
          aws-region: 'eu-west-2'
          cluster-name: 'my-cluster'
          namespace: 'production'
```

## Action Inputs

### Init
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `yaml-paths` | No | `.` | Paths to validate |

### Build
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `java-version` | No | `21` | Java version |
| `gradle-args` | No | `` | Additional Gradle args |

### Image
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `java-version` | No | `21` | Java version |
| `image-registry` | Yes | - | Container registry URL |
| `image-name` | Yes | - | Image name |
| `image-tag` | No | `latest` | Image tag |

### Deploy
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `aws-region` | Yes | - | AWS region |
| `cluster-name` | Yes | - | EKS cluster name |
| `namespace` | No | `default` | K8s namespace |
| `manifest-path` | No | `k8s/` | Path to manifests |

## Notes

- `on: push:` without branches runs on all branch pushes
- Use `if: github.ref == 'refs/heads/dev' || github.ref == 'refs/heads/main'` to limit jobs to specific branches
- AWS credentials should be configured via OIDC or secrets in the calling workflow
- Gradle wrapper (`gradlew`) must exist in the repository for build/image actions
