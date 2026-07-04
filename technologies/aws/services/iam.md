# AWS IAM (Identity and Access Management)

#technology #aws

**IAM** controls **who** can do **what** on **which resources** in your AWS account. Every API call is authorized through IAM — human console users, CI pipelines, [EC2](ec2.md) instances, [ECS](ecs.md) tasks, and [EKS](eks.md) pods all get permissions via users, groups, or (preferably) **roles**.

Part of: [Amazon Web Services](aws.md). Shows up everywhere: [ECR](ecr.md) pull/push, GitHub Actions OIDC, instance profiles, IRSA.

---

## Resources

- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM policy reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [Roles terms and concepts](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [OIDC federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html)
- [Hands-on: ECR push via GitHub Actions (OIDC)](../../../github-actions/hands-on/ecr-github-actions-oidc.md)

---

## Core Concepts

- **Principal**  
  The identity making a request — IAM user, assumed role session, AWS service.

- **User vs role**  
  **Users** = long-lived humans (console/CLI). **Roles** = temporary credentials for workloads or federation. Prefer roles for apps, CI, and EC2/containers — no static access keys in repos.

- **Group**  
  Collection of users; attach policies once at group level (e.g. `Developers`, `ReadOnly`).

- **Policy**  
  JSON document allowing or denying actions. Two main types:
  - **Identity-based** — attached to user, group, or role (`Principal` is implicit).
  - **Resource-based** — attached to a resource (S3 bucket policy, ECR repository policy); includes explicit `Principal`.

- **Role = trust policy + permission policies**  
  **Trust policy** — who can assume the role (`sts:AssumeRole`, `sts:AssumeRoleWithWebIdentity`). **Permission policies** — what the role can do after assumption.

- **ARN**  
  Unique resource identifier used in policies, e.g. `arn:aws:ecr:eu-central-1:123456789012:repository/myapp/api`.

- **Evaluation**  
  Default **deny**. Explicit **Allow** in any applicable policy grants access unless an explicit **Deny** wins. Request must pass **trust** (for roles) and **permissions**.

- **Managed vs inline**  
  **AWS managed** — AWS-maintained (`ReadOnlyAccess`). **Customer managed** — your reusable policies. **Inline** — embedded on one user/role; harder to reuse.

- **Instance profile**  
  Wrapper that attaches a role to an [EC2](ec2.md) instance — apps on the host call AWS APIs with temporary credentials.

---

## Common access patterns

**Human admin** — IAM user or SSO (IAM Identity Center) → group with admin or scoped policies. MFA on privileged accounts.

**CI/CD (GitHub Actions)** — OIDC identity provider + IAM role with trust scoped to `repo:org/name`. No access keys in GitHub Secrets. See [ECR push via GitHub Actions (OIDC)](../../../github-actions/hands-on/ecr-github-actions-oidc.md).

**EC2 workload** — instance profile on the launch template; role permissions for S3, SSM, [ECR](ecr.md) pull, etc.

**ECS** — **task execution role** (pull image, write logs) separate from **task role** (app runtime AWS API access).

**EKS** — **IRSA**: Kubernetes service account mapped to an IAM role; pod gets scoped credentials without broad node permissions.

**Least privilege** — scope `Resource` ARNs and `Action` lists; avoid `*` on actions/resources unless required (e.g. `ecr:GetAuthorizationToken`).

---

## Policy examples

### Identity policy — scoped ECR push (CI or deploy role)

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
      "Resource": "arn:aws:ecr:REGION:ACCOUNT_ID:repository/myapp/api"
    }
  ]
}
```

### Trust policy — GitHub Actions OIDC

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
          "token.actions.githubusercontent.com:sub": "repo:ORG/REPO:*"
        }
      }
    }
  ]
}
```

Tighter: `"token.actions.githubusercontent.com:sub": "repo:ORG/REPO:ref:refs/heads/main"` for main-branch pushes only.

### Identity policy — ECS task execution role (ECR pull + logs)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:REGION:ACCOUNT_ID:log-group:/ecs/myapp:*"
    }
  ]
}
```

Attach a separate **task role** for application permissions (e.g. `s3:GetObject` on one bucket) — keeps image pull separate from app access.

---