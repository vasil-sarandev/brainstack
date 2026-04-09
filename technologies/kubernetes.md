
# Kubernetes (K8s)

#technology 

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It turns a collection of individual servers (nodes) into a single, unified computing resource that self-heals and scales based on demand.

## Resources

- [Kubernetes Documentation: Concepts](https://kubernetes.io/docs/concepts/)
- [K8s Interactive Tutorials](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Helm: The K8s Package Manager](https://helm.sh/)

## Core Concepts

- **Cluster & Nodes:** A cluster consists of a **Control Plane** (the brain) and multiple **Worker Nodes** (the muscle—usually VPSes or bare metal).
- **Pods:** The smallest deployable unit. It’s a logical wrapper around one or more containers that share the same network and storage.
- **Desired State:** You don’t tell K8s _how_ to start a server; you give it a **YAML manifest** describing what you want (e.g., "3 replicas of my API"), and K8s makes it happen.
- **Self-Healing:** If a container crashes or a node dies, K8s automatically restarts or reschedules those pods on healthy hardware without human intervention.
- **Service Discovery & Load Balancing:** K8s gives a stable DNS name and IP to a group of pods, automatically balancing traffic across them regardless of which node they live on.
- **Horizontal Scaling:** Automatically increases or decreases the number of pods based on CPU/RAM usage or custom metrics.
- **Declarative Updates:** Supports "Rolling Updates" where new versions are deployed one by one, ensuring zero downtime during releases.
- **Bin Packing:** Intelligently places containers on nodes to maximize resource utilization and minimize infrastructure costs.