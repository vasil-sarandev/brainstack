# Deploy to EC2

#infrastructure

Run workloads on a **VM you manage directly** — compiled binaries, JARs, interpreted apps under systemd, or Docker containers. The artifact type doesn't dictate this target; what does is wanting full control over the host instead of handing placement/restarts/scaling to an orchestrator (ECS/EKS) or a PaaS.

Case study: [Deploy showtimex to EC2 with Docker](../case-studies/deploy-to-ec2-docker-showtimex.md).

---

## Pipeline shape

Two equally valid artifact paths — pick based on what you're already building, not because one is "more correct":

```text
Non-containerized:
CI: mvn package / npm run build / go build
  → artifact (JAR, tarball, binary)
  → upload to S3 or Artifactory
  → deploy: copy to EC2 + restart process

Docker on EC2:
CI: docker build → push to ECR (:git-sha, immutable tag)
  → deploy: SSM Run Command on the instance
      → docker pull the new tag → stop old container → run new one
```

Production never runs `git pull && npm install` (or `docker build`) on the server. It runs a **known built artifact** — a versioned tarball/JAR or an immutably-tagged image, either way built once in CI.

---

## Common patterns

**Single instance** — one EC2, systemd unit or process manager (or a single long-running Docker container with `--restart unless-stopped`), Elastic IP or Route 53 A record. Simplest; no auto-healing. Fine for low-traffic services or a single-instance test/hobby deploy — see the [case study](../case-studies/deploy-to-ec2-docker-showtimex.md) for one built this way end to end.

**ASG + ALB** — stateless API behind an Application Load Balancer; Auto Scaling Group replaces unhealthy instances and scales count. Standard production pattern for JVM/Node APIs.

**CodeDeploy** — push artifact to S3; CodeDeploy agent on instances pulls, stops old version, starts new one. Supports rolling and blue/green at the instance layer.

**Docker on EC2 via SSM Run Command** — install Docker on the VM, pull from ECR, `docker run` triggered remotely by CI over [SSM Run Command](../../../technologies/aws/services/ssm.md) — no SSH key, no open port 22, no agent-based orchestrator. You own restarts (`--restart unless-stopped`), health (target group health checks, not an orchestrator's), and placement (one fixed instance, no automatic rescheduling) yourself. A deliberate choice for "I want a container running on a box I fully control," not a lesser version of ECS — but if you need multiple replicas, zero-downtime rolling deploys across instances, or auto-healing, that's exactly what ECS/EKS automate for you (see [ECS vs EC2, concretely](#ecs-vs-ec2-concretely) below).

---

## Config and secrets

- Env vars in systemd unit files, `.env` on disk (avoid secrets in git), or fetched at boot from **SSM Parameter Store** / **Secrets Manager**
- **IAM instance profile** on the launch template — no long-lived AWS keys on disk
- Security groups: ALB → app port only; SSH via SSM Session Manager preferred over open port 22

---

## CI deploy (conceptual)

Non-containerized:

```yaml
# after build + upload artifact to S3
- aws deploy create-deployment
    application-name: my-api
    deployment-group-name: production
    s3-location: bucket=my-artifacts,key=api-${{ github.sha }}.jar
```

Or: new AMI / launch template version → ASG instance refresh.

Docker on EC2, via SSM (see [SSM — Run Command for a CI/CD deploy](../../../technologies/aws/services/ssm.md#run-command-for-a-cicd-deploy-conceptual) for the full snippet):

```yaml
# after build + push to ECR
- aws ssm send-command
  --instance-ids "$INSTANCE_ID"
  --document-name "AWS-RunShellScript"
  --parameters commands='["bash /opt/app/deploy.sh"]'
# then poll get-command-invocation for success/failure
```

---

## When to use something else

| Need                                                 | Consider                               |
| ---------------------------------------------------- | -------------------------------------- |
| Multiple processes from one image, independent scale | [ECS](ecs.md) or [EKS](eks.md)         |
| Static frontend only                                 | [S3 static SPA](s3-static-spa.md)      |
| Managed container scheduling                         | ECS / EKS instead of DIY Docker on EC2 |
