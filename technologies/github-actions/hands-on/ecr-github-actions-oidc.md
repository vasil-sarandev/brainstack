# ECR push via GitHub Actions (OIDC)

#technology #github-actions #aws #docker

Hands-on walkthrough for a common CI pattern: build a Docker image in GitHub Actions and push to AWS ECR — no long-lived AWS keys in the repo.

Part of: [GitHub Actions](../github-actions.md). AWS background: [ECR](../../aws/services/ecr.md), [IAM](../../aws/services/iam.md).

Reference implementation: [showtimex](https://github.com/vasil-sarandev/showtimex) — workflow [`.github/workflows/ci-cd.yml`](https://github.com/vasil-sarandev/showtimex/blob/main/.github/workflows/ci-cd.yml), app deployment notes in [`docs/deployment.md`](https://github.com/vasil-sarandev/showtimex/blob/main/docs/deployment.md).

---

## Resources

- **GitHub**
  - [Configuring OIDC in AWS](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- **AWS**
  - [Push a Docker image to ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html)
  - [Create a role for GitHub OIDC](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)
- **Actions**
  - [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
  - [aws-actions/amazon-ecr-login](https://github.com/aws-actions/amazon-ecr-login)

---

## Architecture

```text
push to main
     │
     ▼
┌─────────────────────────────────────────────┐
│ GitHub Actions (ubuntu-latest)              │
│  1. lint + test (all pushes / PRs)          │
│  2. OIDC → assume IAM role                   │
│  3. docker login to ECR                     │
│  4. docker build (Dockerfile target prod)   │
│  5. docker push                             │
└─────────────────────────────────────────────┘
     │
     ▼
Amazon ECR  repository: showtimex/api
  tag: <git-sha> (immutable tags — no `latest` overwrite)
```

Runtime deploy (EC2, RDS, SSM) is **separate** — CI only publishes the image.

---

## AWS setup (one-time, console)

### 1. ECR repository

- **Service:** Elastic Container Registry
- **Repository name:** `showtimex/api` (must match `ECR_REPOSITORY` in the workflow exactly, including `/`)
- **Region:** pick one region and use it everywhere (ECR, GitHub secret `AWS_REGION`, future EC2/RDS)

Full image URI pattern:

```text
{account-id}.dkr.ecr.{region}.amazonaws.com/showtimex/api:<git-sha>
```

**Tag immutability:** if enabled on the repository, each tag can only be pushed once. CI pushes **`${{ github.sha }}` only** — do not push `latest` unless the repo uses mutable tags.

### 2. OIDC identity provider (per AWS account)

IAM → **Identity providers** → Add provider:

| Field    | Value                                         |
| -------- | --------------------------------------------- |
| Type     | OpenID Connect                                |
| URL      | `https://token.actions.githubusercontent.com` |
| Audience | `sts.amazonaws.com`                           |

### 3. IAM role for GitHub Actions

Create role → **Web identity** → select the GitHub OIDC provider → audience `sts.amazonaws.com`.

**Trust policy** — scope to the repo (example from Showtimex):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:vasil-sarandev/showtimex:*"
        }
      }
    }
  ]
}
```

Tighter option: restrict to `main` only:

```text
repo:vasil-sarandev/showtimex:ref:refs/heads/main
```

**Permissions** — attach ECR push access. Options:

- **Quick:** managed policy `AmazonEC2ContainerRegistryFullAccess` (broad — all repos in the account)
- **Better:** custom policy scoped to `arn:aws:ecr:REGION:ACCOUNT_ID:repository/showtimex/api`

Custom policy minimum:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ecr:GetAuthorizationToken",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "arn:aws:ecr:REGION:ACCOUNT_ID:repository/showtimex/api"
    }
  ]
}
```

Copy the role **ARN** for GitHub.

---

## GitHub repository secrets

Settings → Secrets and variables → Actions:

| Secret         | Value                            |
| -------------- | -------------------------------- |
| `AWS_ROLE_ARN` | IAM role ARN from step 3         |
| `AWS_REGION`   | ECR region (e.g. `eu-central-1`) |

No `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` when using OIDC.

---

## Workflow behaviour

**File:** `.github/workflows/ci-cd.yml`

| Job                    | When                | What                            |
| ---------------------- | ------------------- | ------------------------------- |
| `run-linter`           | push + PR           | `npm run lint`                  |
| `run-tests`            | push + PR           | `npm run test`                  |
| `build-and-push-image` | push to `main` only | OIDC → ECR login → build → push |

---

## Workflow snapshot

This may be out of date, but still decided it's nice to have, you can reference the repository for the latest.

```yaml
name: Continiuous Integration & Deployment
run-name: ${{ github.run_id }} - Showtimex CI&CD - 🚀

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  run-linter:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: ./.github/actions/setup-runner
      - name: Run linter
        run: npm run lint

  run-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: ./.github/actions/setup-runner
      - name: Run tests
        run: npm run test

  build-and-push-image:
    runs-on: ubuntu-latest
    needs: [run-linter, run-tests]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v6

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ secrets.AWS_REGION }}

      - uses: aws-actions/amazon-ecr-login@v2
        id: ecr-login

      - name: Build and push Docker image to ECR
        env:
          ECR_REGISTRY: ${{ steps.ecr-login.outputs.registry }}
          ECR_REPOSITORY: showtimex/api
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -f docker/Dockerfile --target prod \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```
