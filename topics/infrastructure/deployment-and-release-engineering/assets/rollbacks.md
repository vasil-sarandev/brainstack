# Rollbacks

#infrastructure

Getting production back to a **known good state** after a bad deploy or incident. Speed matters — the longer bad code runs, the worse the blast radius.

Parent: [Deployment & Release Engineering](../deployment-and-release-engineering.md)

---
## Two kinds of "rollback"

| Type | What you revert | Speed | When |
|------|-----------------|-------|------|
| **Behaviour rollback** | Feature flag off | Seconds | Logic bug in new feature, old path still works |
| **Deploy rollback** | Previous image/artifact tag | Minutes | Bad build, crash loop, broken startup |

Use [feature flags](feature-flags.md) for the first; deployment rollback for the second.

---
## Container rollback (ECS / EKS)

Images in [ECR](ecr.md) are **immutable by tag**. Keep previous tags — never overwrite `latest` in prod.

**ECS:**

1. Identify last good image tag (git SHA, semver)
2. Register new task definition revision pointing at that tag
3. Update service → rolling replace back to old tasks

Often automated: "rollback to previous task definition revision" in console or CI.

**EKS:**

```bash
kubectl rollout undo deployment/my-api
# or pin to specific revision
kubectl rollout undo deployment/my-api --to-revision=3
```

**Prerequisite:** CI tags every build (`abc1234`, `1.2.3`). You can't rollback to what you didn't keep.

---
## Traditional artifact rollback (EC2)

Same idea, different artifact:

1. Previous JAR/tarball still in S3 (versioned path or dated folder)
2. Copy to instances
3. `systemctl restart myapp`

Or **CodeDeploy** rollback to previous deployment group revision.

---
## Database migrations and rollbacks

**Code rollback ≠ schema rollback.** This is the hard part.

| Migration style | Rollback story |
|-----------------|----------------|
| **Backward-compatible** (add nullable column) | Roll back app; old code ignores new column ✓ |
| **Destructive** (drop column, rename) | App rollback may **break** against new schema ✗ |

**Expand / contract pattern:**

1. **Expand** — add new column/table; deploy code that writes to both
2. **Migrate** data
3. **Contract** — remove old column; deploy code that only uses new

Rollback is safe between expand and contract. After contract, rollback needs a forward migration or restore from backup.

**Rule:** design migrations so **app rollback is always possible** until the contract phase is complete.

---
## Rollback vs fix-forward

| Rollback | Fix-forward |
|----------|-------------|
| Bad deploy, clear previous version works | Data already corrupted; schema already contracted |
| Fast, low risk | Patch on top of bad version |
| Preferred default | When rollback causes more harm (e.g. ran destructive migration) |

When in doubt during an incident: **rollback first**, investigate second.

---
## What makes rollbacks easy

Built into the pipeline from day one:

- **Immutable, tagged artifacts** — every merge to `main` → unique ECR tag
- **Health checks** — ALB / K8s probes fail bad tasks before full rollout completes
- **Deployment circuit breaker** — ECS can auto-rollback if new tasks fail health checks
- **Flags for risky features** — behaviour rollback without redeploy
- **Backward-compatible migrations** — expand/contract, no big-bang schema changes with code deploy

See [Infrastructure Hands On](../../infrastructure-hands-on/infrastructure-hands-on.md) for hands-on ECS setup with health checks and CI push to ECR.

---
## Related

- [Deployment Strategies](deployment-strategies.md) — blue/green enables instant traffic flip back
- [Feature Flags](feature-flags.md)
- [GitHub Actions](../../../../../technologies/github-actions/github-actions.md) — CI that tags builds
- [ECS](ecs.md)

---
