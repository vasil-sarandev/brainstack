# AWS ALB (Application Load Balancer) & Target Groups

#technology #aws

**ALB** is AWS's Layer 7 (HTTP/HTTPS) load balancer — a stable, internet-facing (or internal) entry point that routes to a **target group**, which tracks the actual backend IPs/instances and health-checks them. The two are always used together; covered here as one page since neither is very useful without the other.

Part of: [Amazon Web Services](../aws.md). Routes to targets in a [VPC](vpc.md) — commonly [ECS](ecs.md) tasks (`awsvpc` mode, IP targets) or [EC2](ec2.md) instances/ASGs.

---

## Resources

- [Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [Target groups for ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
- [Target group health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)

---

## Core Concepts

- **Console location is a historical artifact, not a hint about dependencies**
  ALB/ELB lives under the **EC2 console** even for setups with zero EC2 instances (e.g. all-Fargate ECS) — Elastic Load Balancing predates ECS/Fargate and AWS never relocated it. Its IAM actions are `elasticloadbalancing:*`, unrelated to `ec2:*`; there's no functional coupling to EC2 at all.

- **Listener**
  Bound to the ALB at a protocol/port (e.g. HTTP:80). Created **as part of the "create load balancer" wizard**, not a separate step — an ALB isn't functional without at least one, so AWS bundles it in. This matters later: creating an ECS *service* that attaches to an existing ALB will, by default, try to **create a new listener** rather than reuse the one already there — if the port's taken, you'll see `Listener port already exists`. Fix: explicitly select "use an existing listener" in the service wizard.

- **Target group**
  The actual routing/health destination a listener forwards to. Does two things the ALB itself doesn't:
  - **Membership tracking** — the list of current targets (IPs, instance IDs, or Lambda ARNs depending on target type).
  - **Health checking** — runs on its own schedule against each target independently; only targets it currently considers healthy receive traffic.

- **Target type: IP vs Instance**
  Fargate tasks (`awsvpc` networking) require target type **IP**, not Instance — that's what lets the ALB route directly to a task's ENI rather than an EC2 instance ID.

- **Don't manually register targets for ECS-backed target groups**
  When an ECS *service* is configured with a load balancer, ECS registers/deregisters the real task IPs into the target group automatically as tasks start/stop/redeploy. Manually adding an IP in the target-group wizard just creates a stale entry — skip that step entirely for ECS use cases.

- **Health check path has no sane default**
  Defaults to `/` in the console wizard regardless of whether your app actually has a route there. A 404 (or connection refused, if the app doesn't even start — see below) means the target never goes healthy and the service never stabilizes. Make sure a real health endpoint exists (e.g. `GET /health` returning 200) before wiring up the target group.

- **Deregistration delay**
  Default 300 seconds (5 minutes) before an old target is considered fully drained during a deployment. This is usually the biggest reason a rolling ECS deployment looks "done" in the console (new task healthy, serving traffic) well before the deployment officially reports **stable** — the old task is still finishing its drain window.

---

## Why bother with an ALB for a single-instance service

Worth calling out since it's non-obvious: with desired count = 1, an ALB isn't doing load balancing in the traditional multi-replica sense. Its actual value:

- **A stable address.** Backend IPs (especially Fargate tasks) are ephemeral and change on every restart/redeploy — the ALB gives you one fixed DNS name that always routes to whatever's currently healthy.
- **Health-check-gated rolling deploys.** A new task only starts receiving traffic once it passes its health check; the old one only stops once it's confirmed drained. Without this, a redeploy is a hard cutover with a gap where nothing's listening.

If neither of those matter (a true one-off test, no redeploys expected), skipping the ALB and giving the task itself a public IP + tight security group is a legitimate shortcut — just loses both properties above.

---

## Failure mode to know: app never starts, not just "degraded"

If a service's startup code blocks on something failing (e.g. a Node app that awaits a Kafka connection before calling `app.listen()`), the target group never sees anything to health-check at all — connection refused, not a 404. ECS keeps replacing the task (desired count reconciliation), the deployment circuit breaker eventually trips, and this can look confusing in the ALB/target group UI (targets flapping between "initial" and "unhealthy," never "healthy"). Worth checking the app's own startup ordering, not just the target group config, when a service refuses to go healthy.

---
