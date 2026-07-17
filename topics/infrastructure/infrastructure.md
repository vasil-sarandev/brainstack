# Infrastructure

#infrastructure

Build → ship → run. Deployment pipelines, release strategies, and case studies.

Concept notes here; tool reference under [Technologies](../../../technologies/technologies.md) (Docker, AWS, GitHub Actions, Kubernetes).

---
## Resources

- **Deep Dives**
	- [Deployment & Release Engineering](deployment-and-release-engineering/deployment-and-release-engineering.md) — CI/CD, rollout strategies, feature flags, rollbacks, Autoscaling
	- [Deployment targets](deployment-targets/deployment-targets.md) — where workloads run (overview)
		- [EC2](deployment-targets/ec2.md) · [ECS](deployment-targets/ecs.md) · [EKS](deployment-targets/eks.md) · [S3 static SPA](deployment-targets/s3-static-spa.md)

- **Case Studies**
	- [ecs-node-distributed-monolith](ecs-node-distributed-monolith.md) — one image, API + Kafka consumers, GitHub Actions → ECR → ECS, Obeservability (CloudWatch + Prometheus & Grafana)


---
