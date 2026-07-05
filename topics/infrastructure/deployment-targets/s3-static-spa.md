# Deploy static SPA to S3

#infrastructure

Host a **frontend single-page app** (React, Vue, Vite, Next.js static export) as static files — no server process. API runs elsewhere ([ECS](ecs.md), [EC2](ec2.md), etc.).

Parent: [Deployment targets](deployment-targets.md).

---

## Pipeline shape

```text
CI: npm run build  →  dist/ or build/
  → aws s3 sync dist/ s3://my-app-bucket --delete
  → CloudFront invalidation (/* or /index.html)
```

Artifact is a **folder of files**, not a container image.

---

## Architecture

```text
Browser → CloudFront (CDN) → S3 bucket (private, OAI/OAC access only)
                ↓
           API on ECS / ALB / API Gateway  (separate deploy)
```

- **S3 bucket** — stores `index.html`, JS, CSS, assets
- **CloudFront** — HTTPS, caching, geographic edge; origin = S3
- Bucket **not** public; CloudFront accesses via Origin Access Control (OAC)

SPA routing: CloudFront custom error response **403/404 → 200 `/index.html`** so client-side routes work on refresh.

---

## CI example (conceptual)

```yaml
- run: npm ci && npm run build
- run: aws s3 sync ./dist s3://$BUCKET --delete
- run: aws cloudfront create-invalidation --distribution-id $DIST_ID --paths "/*"
```

Auth to AWS: OIDC role with `s3:PutObject`, `cloudfront:CreateInvalidation` on scoped resources.

---

## Config

| Concern | Approach |
| --- | --- |
| API URL | Build-time env (`VITE_API_URL`, `NEXT_PUBLIC_*`) baked into JS at CI |
| Secrets | Never in the static bundle — only public config |
| Preview envs | Separate bucket + CloudFront distribution or path prefix per branch |
| Cache busting | Hashed filenames from bundler; invalidate `index.html` on deploy |

---

## When not to use

| Need | Use instead |
| --- | --- |
| Server-side rendering (Next.js App Router on Node) | ECS / EKS / EC2 |
| API + static in one deploy unit | Container on ECS, or monolith on EC2 |
| WebSockets from same origin as static files | Usually co-locate or use API Gateway |
