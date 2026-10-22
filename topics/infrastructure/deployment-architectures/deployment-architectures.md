# Deployment Architectures

#infrastructure

**How** an application is split into deployable units — one process, one image run as many roles, or many independently built services. Distinct from [deployment targets](../deployment-targets/deployment-targets.md) (where it runs) and [rollout strategies](../deployment-and-release-engineering/assets/deployment-strategies.md) (how a new version replaces the old one).

Parent hub: [Infrastructure](../infrastructure.md). Code-level styles: [Software Design and Architecture — Architectural Styles](../../software-engineering/software-design-and-architecture/software-design-and-architecture.md#architectural-styles).

---

## Guides

- [Monolith](monolith.md) — one codebase, one image, one process type; scale the whole app
- [Modular Monolith](modular-monolith.md) — one codebase, one image, many deployments (API, read/write API, consumers, workers)
- [Microservices](microservices.md) — one image and deployment per service, own data, independent releases

---

## Comparison

|                     | **Monolith**          | **Modular monolith**                  | **Microservices**             |
| ------------------- | --------------------- | ------------------------------------- | ----------------------------- |
| Repos               | 1                     | 1                                     | 1 (monorepo) or many          |
| Images              | 1                     | 1                                     | One per service               |
| Deployments         | 1                     | One per role                          | One per service               |
| Database            | Shared                | Shared (often writer + readers)       | One per service (ideally)     |
| Scale independently | No                    | Per role                              | Per service                   |
| Release             | All at once           | Same image, roll out per role         | Fully independent             |
| Communication       | In-process calls      | In-process + Kafka between roles      | Network (HTTP/gRPC) + events  |
| Good fit            | Small app, small team | Most products at scale, one/few teams | Many teams, separate domains  |

```text
Monolith ──► Modular monolith ──► Microservices
          split process roles   split codebase + data per domain
```

Usual path: start as a monolith, split **process roles** when they need different scaling or isolation, split **services** only when team or domain boundaries demand it.

---

## Example projects

| Architecture     | Repo                                                       | Case study                                                                 |
| ---------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------- |
| Monolith         | [showtimex](https://github.com/vasil-sarandev/showtimex)   | [EC2 + Docker via SSM](../case-studies/deploy-to-ec2-docker-showtimex.md)  |
| Modular monolith | [node-modulith](https://github.com/vasil-sarandev/node-modulith) | [ECS + MSK](../case-studies/deploy-to-ecs-node-modulith.md)          |
