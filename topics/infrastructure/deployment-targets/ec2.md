# Deploy to EC2

#infrastructure

Run **non-containerized** or **VM-managed** workloads: compiled binaries, JARs, interpreted apps under systemd, or Docker Compose on a single host without an orchestrator.

Parent: [Deployment targets](deployment-targets.md). Service details: [AWS EC2](../../../technologies/aws/services/ec2.md).

---

## Pipeline shape

```text
CI: mvn package / npm run build / go build
  → artifact (JAR, tarball, binary)
  → upload to S3 or Artifactory
  → deploy: copy to EC2 + restart process
```

Production never runs `git pull && npm install` on the server. It runs a **known built artifact**.

---

## Common patterns

**Single instance** — one EC2, systemd unit or process manager, Elastic IP or Route 53 A record. Simplest; no auto-healing.

**ASG + ALB** — stateless API behind an Application Load Balancer; Auto Scaling Group replaces unhealthy instances and scales count. Standard production pattern for JVM/Node APIs without containers.

**CodeDeploy** — push artifact to S3; CodeDeploy agent on instances pulls, stops old version, starts new one. Supports rolling and blue/green at the instance layer.

**Docker on EC2 (no ECS)** — install Docker on the VM, pull from ECR, `docker run` via user data or systemd. Possible but you own restarts, scaling, and placement yourself — usually a stepping stone to ECS.

---

## Config and secrets

- Env vars in systemd unit files, `.env` on disk (avoid secrets in git), or fetched at boot from **SSM Parameter Store** / **Secrets Manager**
- **IAM instance profile** on the launch template — no long-lived AWS keys on disk
- Security groups: ALB → app port only; SSH via SSM Session Manager preferred over open port 22

---

## CI deploy (conceptual)

```yaml
# after build + upload artifact to S3
- aws deploy create-deployment
    application-name: my-api
    deployment-group-name: production
    s3-location: bucket=my-artifacts,key=api-${{ github.sha }}.jar
```

Or: new AMI / launch template version → ASG instance refresh.

---

## When to use something else

| Need | Consider |
| --- | --- |
| Multiple processes from one image, independent scale | [ECS](ecs.md) or [EKS](eks.md) |
| Static frontend only | [S3 static SPA](s3-static-spa.md) |
| Managed container scheduling | ECS / EKS instead of DIY Docker on EC2 |
