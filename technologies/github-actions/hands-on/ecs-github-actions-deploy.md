# ECS deploy via GitHub Actions

#technology #github-actions #aws #docker

Hands-on walkthrough for the natural follow-up to [ECR push via GitHub Actions](ecr-github-actions-oidc.md): after the image lands in ECR, register a new ECS task definition revision and update the service — a real rolling deploy, not just a build.

Part of: [GitHub Actions](../github-actions.md). AWS background: [ECS](../../aws/services/ecs.md), [ECR](../../aws/services/ecr.md), [IAM](../../aws/services/iam.md).

Reference implementation: [node-modulith](https://github.com/vasil-sarandev/node-modulith) — workflow [`.github/workflows/ci-cd.yml`](https://github.com/vasil-sarandev/node-modulith/blob/main/.github/workflows/ci-cd.yml), app deployment notes in [`docs/deployment.md`](https://github.com/vasil-sarandev/node-modulith/blob/main/docs/deployment.md). Full manual build-out of the underlying infra (VPC/SGs/MSK/ECS/ALB) documented in the [ECS + MSK case study](../../../topics/infrastructure/case-studies/deploy-to-ecs-node-modulith.md) — this page is specifically the CI/CD layer on top of that.

---

## Resources

- **AWS**
  - [RegisterTaskDefinition API](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_RegisterTaskDefinition.html)
  - [UpdateService API](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_UpdateService.html)
  - [iam:PassRole explained](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html)
- **Actions**
  - [aws-actions/amazon-ecs-render-task-definition](https://github.com/aws-actions/amazon-ecs-render-task-definition)
  - [aws-actions/amazon-ecs-deploy-task-definition](https://github.com/aws-actions/amazon-ecs-deploy-task-definition)

---

## Architecture

```text
merge to main
     │
     ├─ lint + test
     ├─ docker build (runtime) → push to ECR
     │     tagged :{git-sha} (immutable, what gets deployed)
     │     tagged :main       (floating, for convenience/debugging)
     │
     └─ deploy-to-ecs (matrix, one leg per service)
           │
           ├─ describe-task-definition <service>-task   (always fetches latest revision)
           ├─ render new image into it                  (amazon-ecs-render-task-definition)
           └─ register + update service                 (amazon-ecs-deploy-task-definition)
                 waits for service stability before the job reports done
```

Each ECS service gets its own independent rolling deployment off the same image — a broken deploy on one consumer doesn't block the API or the other consumer. This assumes the ECS cluster, task definitions, and services already exist (see the [case study](../../../topics/infrastructure/case-studies/deploy-to-ecs-node-modulith.md) for standing those up by hand) — this workflow only ever registers *new revisions* of task definitions that already exist, it doesn't create infrastructure from scratch.

---

## AWS setup (extends the ECR hands-on's OIDC role)

Builds directly on [ECR push via GitHub Actions](ecr-github-actions-oidc.md) — same OIDC provider, same IAM role, just more permissions attached to it.

### Extend the OIDC role's policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EcsDeploy",
      "Effect": "Allow",
      "Action": [
        "ecs:DescribeTaskDefinition",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService",
        "ecs:DescribeServices"
      ],
      "Resource": "*"
    },
    {
      "Sid": "PassExecutionRole",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::ACCOUNT_ID:role/YOUR_ECS_TASK_EXECUTION_ROLE"
    }
  ]
}
```

`RegisterTaskDefinition` doesn't support resource-level scoping (hence `"*"`), but **`PassRole` should stay scoped to exactly the execution role this workflow needs** — never `"*"` there, or the workflow could pass an arbitrary role to anything it registers.

---

## Gotchas (learned the hard way)

**`AWS_REGION` must be a `vars.*`, not a `secrets.*`, if you're passing anything derived from it as a cross-job output.** GitHub Actions masks any log/output content containing a registered secret value — including a region string embedded inside a larger value like an ECR registry hostname (`{account}.dkr.ecr.{region}.amazonaws.com`). If the region is a `secrets.AWS_REGION`, any job output containing that substring gets **silently dropped** when crossing a job boundary via `needs.<job>.outputs.*` — no error, just an empty string on the receiving end. Region isn't actually sensitive information, so store it as a repository **variable** instead (`vars.AWS_REGION`) and this never happens. Confirmed by a controlled test: two outputs set identically in the same job, one containing the masked substring and one not — only the clean one made it across.

**`describe-task-definition --task-definition <family>` (no revision suffix) always returns the *latest* revision.** Handy — it means fixing something in the console (e.g. a wrong execution role on one task def) is picked up automatically on the next workflow run without touching the workflow itself.

**`iam:PassRole` failures name the exact role that's wrong.** If you see `not authorized to perform: iam:PassRole on resource: arn:...:role/some-other-role`, that's not a policy bug — it means whatever task definition the workflow fetched genuinely has a different execution role attached than the one your CI role is allowed to pass. Go fix the task definition, not the policy (unless the policy is pointed at the wrong role entirely).

**ARNs, account IDs, and role names showing up unmasked in logs is expected, not a leak.** They're identifiers, not credentials — knowing a role's ARN doesn't let anyone assume it without also passing the trust policy's conditions (e.g. a valid GitHub-signed OIDC token scoped to your exact repo) and holding real credentials. What *should* be masked and was — access key ID, secret access key, session token — is exactly what `configure-aws-credentials` does mask.

---

## Workflow behaviour

**File:** `.github/workflows/ci-cd.yml`

| Job | When | What |
| --- | --- | --- |
| `run-linter` | push + PR | `npm run lint` |
| `run-tests` | push + PR | `npm run test` |
| `build-and-push-image` | push to `main` only | OIDC → ECR login → build → push (`:sha` + `:main`) |
| `deploy-to-ecs` | after `build-and-push-image` | matrix over services → register new revision → update service, wait for stability |

---

## Workflow snapshot

This may be out of date, but still decided it's nice to have, you can reference the repository for the latest.

```yaml
name: Continiuous Integration & Deployment
run-name: ${{ github.run_id }} - Node Modulith CI&CD - 🚀

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
          aws-region: ${{ vars.AWS_REGION }}

      - uses: aws-actions/amazon-ecr-login@v2
        id: ecr-login

      - name: Build and push Docker image to ECR
        id: build-and-push
        env:
          ECR_REGISTRY: ${{ steps.ecr-login.outputs.registry }}
          ECR_REPOSITORY: node-modulith
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -f ./Dockerfile --target runtime \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:main \
            .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:main
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> "$GITHUB_OUTPUT"
    outputs:
      image: ${{ steps.build-and-push.outputs.image }}

  deploy-to-ecs:
    runs-on: ubuntu-latest
    needs: [build-and-push-image]
    permissions:
      id-token: write
      contents: read
    strategy:
      matrix:
        include:
          - service: api
            container: api
          - service: user-marketing-consumer
            container: user-marketing-consumer
          - service: product-restocked-consumer
            container: product-restocked-consumer
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Download current task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition ${{ matrix.service }}-task \
            --query taskDefinition > task-definition.json

      - name: Render new image into task definition
        id: render
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ matrix.container }}
          image: ${{ needs['build-and-push-image'].outputs.image }}

      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.render.outputs.task-definition }}
          service: ${{ matrix.service }}
          cluster: node-modulith-cluster-1
          wait-for-service-stability: true
```
