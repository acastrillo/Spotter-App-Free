# Spotter Free — AWS Hosting & Launch Plan

## Goals
- Launch the web app on AWS with a secure, scalable baseline.
- Establish a clean local workflow, CI/CD, and observability.
- Close security gaps before public exposure and prioritize remaining items.

## Assumptions
- App: Next.js 15 (App Router) + Node 18, Prisma, Tailwind, Docker.
- Current DB: SQLite (dev). Production will use managed Postgres (RDS).
- Target AWS stack: ECS Fargate + ECR + ALB + SSM Parameter Store, with optional Route 53 + ACM.
- Codebase refs: `spotter-free/` (Next.js app), Docker + ECS artifacts present.

References in repo for context:
- `spotter-free/AWS_DEPLOYMENT_GUIDE.md:1` — ECS deployment walkthrough + PowerShell script
- `spotter-free/deploy-to-aws.ps1:1` — One-click ECS deployment script
- `spotter-free/aws-task-definition.json:1` — Task definition template
- `spotter-free/PRODUCTION-READINESS.md:1` — Production checklist

---

## Local Preparation
- Node and tooling: Use Node 18+, Docker Desktop, AWS CLI.
- Environment files: Copy `.env.example` to `.env.local` and set required values for local only. Never commit secrets.
  - `spotter-free/.env.example:7` defines `AUTH_SECRET` but NextAuth v4 requires `NEXTAUTH_SECRET` (see Security Gaps below).
- Build parity: Validate `npm run build` and `npm start` locally. Also validate container: `docker build -t spotter-app . && docker run -p 3000:3000 spotter-app`.
- Health check: Verify `GET /api/health` returns JSON healthy status.
- Instagram import: Confirm `POST /api/instagram-fetch` works with a valid `APIFY_API_TOKEN` (rate limit locally to avoid abuse during testing).
- Production DB plan:
  - Update `prisma/schema.prisma` datasource to `postgresql` (Production), create initial migration files (`prisma migrate dev`) and commit them.
  - Add a pipeline step to run `prisma migrate deploy` against RDS (not from the runtime container; see CI/CD below).
- Auth plan (temporary): If staying with Credentials provider short-term, ensure it is dev-only and disabled in production. Prepare switch to a managed provider (e.g., Cognito) or secure credentials with DB users.

---

## Target AWS Architecture (MVP)
- Compute: ECS Fargate service with 1–2 tasks, `256/512` baseline.
- Container registry: ECR repository (`spotter-app`).
- Networking: Default VPC (or dedicated VPC), two public subnets, ALB with target group to port `3000`.
- Secrets/config: SSM Parameter Store (SecureString) for all secrets (e.g., `NEXTAUTH_SECRET`, `APIFY_API_TOKEN`, `DATABASE_URL`, `NEXTAUTH_URL`).
- Observability: CloudWatch Logs group `/ecs/spotter-app`, basic CloudWatch alarms (CPU, 5XX, TG health).
- Optional hardening: HTTPS (ACM cert + 443 listener), WAF on ALB, Route 53 managed domain.

---

## Implementation Steps

### Phase 1 — Foundations
- IAM: Ensure an execution role for ECS tasks and a separate task role (least privilege). Current template uses a single role.
- ECR: Create `spotter-app` repo; note the repo URI.
- Parameters (SSM Parameter Store): Create SecureString parameters under a clear path, e.g. `/spotter-free/prod/...` for:
  - `NEXTAUTH_SECRET`: Strong random
  - `APIFY_API_TOKEN`: Provided secure token
  - `DATABASE_URL`: Postgres connection string
  - `NEXTAUTH_URL`: e.g., `https://app.yourdomain.com`
  - Optional: `AUTH_TRUST_HOST=true` (as needed behind ALB)

Example commands:
```
aws ssm put-parameter --name "/spotter-free/prod/NEXTAUTH_SECRET" --type SecureString --value "<random>" --overwrite
aws ssm put-parameter --name "/spotter-free/prod/APIFY_API_TOKEN" --type SecureString --value "<token>" --overwrite
aws ssm put-parameter --name "/spotter-free/prod/DATABASE_URL" --type SecureString --value "postgresql://..." --overwrite
aws ssm put-parameter --name "/spotter-free/prod/NEXTAUTH_URL" --type String --value "https://app.yourdomain.com" --overwrite
```

### Phase 2 — Build & Publish Image
- Build locally: `docker build -t spotter-app .`
- Authenticate and push to ECR (replace with your account/region):
```
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <acct>.dkr.ecr.<region>.amazonaws.com
docker tag spotter-app:latest <acct>.dkr.ecr.<region>.amazonaws.com/spotter-app:latest
docker push <acct>.dkr.ecr.<region>.amazonaws.com/spotter-app:latest
```

### Phase 3 — ECS Service & ALB
- Task Definition: Use the provided template and switch env to SSM-based secrets.
  - `spotter-free/aws-task-definition.json:1` currently sets static `environment`. Replace with ECS `secrets` entries referencing SSM parameter ARNs (example below).
- Cluster & Service: Create ECS cluster, target group, ALB, security groups, and ECS service (1 desired count to start). Health check path `/api/health` is implemented.
- HTTPS: Request ACM cert and add HTTPS listener on 443 (HTTP 80 -> 443 redirect). The provided script currently provisions only HTTP.

Example container `secrets` snippet for the task definition:
```
"secrets": [
  {"name": "NEXTAUTH_SECRET", "valueFrom": "arn:aws:ssm:<region>:<acct>:parameter/spotter-free/prod/NEXTAUTH_SECRET"},
  {"name": "APIFY_API_TOKEN", "valueFrom": "arn:aws:ssm:<region>:<acct>:parameter/spotter-free/prod/APIFY_API_TOKEN"},
  {"name": "DATABASE_URL", "valueFrom": "arn:aws:ssm:<region>:<acct>:parameter/spotter-free/prod/DATABASE_URL"},
  {"name": "NEXTAUTH_URL", "valueFrom": "arn:aws:ssm:<region>:<acct>:parameter/spotter-free/prod/NEXTAUTH_URL"}
]
```

### Phase 4 — DNS & TLS (Optional but Recommended)
- Route 53: Create hosted zone and `A`/`CNAME` records targeting ALB DNS.
- ACM: Issue cert for domain; attach to ALB HTTPS listener.
- Security groups: Allow 443 from Internet, restrict 80 to redirect or block.

### Phase 5 — Observability & Scaling
- CloudWatch Logs: Confirm log streams from ECS tasks (group `/ecs/spotter-app`).
- Alarms: CPU >= 80% for 10m, 5XX on ALB, unhealthy target count > 0.
- Auto Scaling: Target tracking on CPU at ~70% (min 1, max 10 tasks).

### Phase 6 — CI/CD
- Build and publish on push to `main`, update task definition, and roll ECS service.
- Run Prisma migrations in CI before deployment:
  - Create RDS and set `DATABASE_URL` (SSM).
  - Add a dedicated pipeline job (ephemeral runner) to run `npm ci && npx prisma migrate deploy` with access to SSM `DATABASE_URL`.
- Example high-level workflow:
```
name: Deploy
on:
  push:
    branches: [main]
jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '18' }
      - run: npm ci
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
  build-deploy:
    needs: migrate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & push image to ECR
        # authenticate, docker build, tag, and push
      - name: Update ECS service
        # register task def (with SSM secrets) and update service
```

---

## Security Gaps & Priorities

### Critical
- Credentials provider defaults present in code:
  - `spotter-free/src/app/api/auth/[...nextauth]/route.ts:15` and `:16` use hardcoded default email/password when env vars are missing. Remove defaults and gate the provider to dev only or replace with a proper provider.
- NextAuth secret misnamed in `.env.example`:
  - `spotter-free/.env.example:7` uses `AUTH_SECRET`, but NextAuth v4 expects `NEXTAUTH_SECRET`. Fix env naming and ensure a strong secret is set in production via SSM.
- SQLite in production:
  - `prisma/schema.prisma:1` uses SQLite. Move to RDS Postgres/MySQL, generate and commit initial migrations, and run `migrate deploy` in CI.
- Secrets management:
  - Ensure all secrets (`NEXTAUTH_SECRET`, `APIFY_API_TOKEN`, `DATABASE_URL`) are stored in SSM (SecureString) and injected via ECS `secrets`. Do not bake into images or use plain `environment`.

### High
- Rate limiting & abuse protection:
  - `spotter-free/src/app/api/instagram-fetch/route.ts:1` lacks rate limiting. Add per-IP rate limits and timeouts; consider a simple Redis-backed limiter or AWS WAF rate-based rules.
- Security headers & CSP:
  - `spotter-free/next.config.ts:1` lacks headers. Add HSTS, CSP, X-Content-Type-Options, Referrer-Policy, Permissions-Policy. Example shown below.
- Cookie/session hardening:
  - Ensure secure cookies, correct `NEXTAUTH_URL`, and proxy awareness (`AUTH_TRUST_HOST`) behind ALB.
- IAM least privilege:
  - `spotter-free/deploy-to-aws.ps1:1` creates a broad task execution role. Split task vs execution roles and restrict SSM `parameter/spotter-free/*` access.

### Medium
- WAF:
  - Attach AWS WAF to ALB (or CloudFront) with common rule sets (SQLi, XSS, bot control/rate limit).
- Dependency scanning:
  - Add Dependabot/Snyk + GitHub Advanced Security (if available).
- Build-time code scanning:
  - Enable `eslint` in CI and add a basic SAST step.

### Low
- HTTPS enforcement:
  - Add 301 redirect from HTTP to HTTPS at ALB and/or app level.
- CloudFront in front of ALB:
  - Optional caching + WAF centralization; useful at scale.

Suggested `next.config.ts` headers example:
```
export default {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'no-referrer' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
          // Example CSP; refine as needed for Next.js and any external domains
          { key: 'Content-Security-Policy', value: "default-src 'self'; img-src 'self' data: https:; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; connect-src 'self' https://api.apify.com;" },
        ],
      },
    ]
  },
}
```

---

## Post-Launch Runbook (MVP)
- Verify: ALB target health, ECS task count, `/api/health` returns healthy, homepage loads.
- Logs: Tail CloudWatch log group for errors; verify no secrets in logs.
- Metrics: CPU/memory utilization, ALB 4XX/5XX, latency.
- Scaling: Validate target tracking policy by load test or manual scaling event.
- Backups: Enable automated RDS backups and snapshot retention.
- Rotation: Establish a rotation schedule for `NEXTAUTH_SECRET` and `APIFY_API_TOKEN`.

---

## Timeline (Suggested)
- Week 0.5: Fix auth defaults, env naming, add security headers.
- Week 1: RDS setup, Prisma migration, SSM secrets, ECS deploy (HTTP).
- Week 1.5: HTTPS with ACM, DNS via Route 53, WAF basics, alarms.
- Week 2: CI/CD (migrations + deploy), rate limiting on endpoints, final review.

---

## Open Questions
- Region, domain, and DNS registrar (for ACM/Route 53 setup)?
- Expected traffic profile (for autoscaling and WAF tuning)?
- Preferred auth provider (Cognito/Auth0/NextAuth Credentials w/ DB)?
- Database choice and sizing for MVP (RDS Postgres vs. MySQL)?

---

## Notes & Fit with Current Repo
- Docker image is configured to run Next.js standalone (`spotter-free/Dockerfile:1`), listening on port 3000 with `HOSTNAME=0.0.0.0`.
- Task definition uses `/api/health` health checks which are implemented (`spotter-free/src/app/api/health/route.ts:1`).
- The included PowerShell script (`spotter-free/deploy-to-aws.ps1:1`) is a fast path for an HTTP-only deploy. For production, extend it to:
  - Create SSM parameters and swap plain `environment` → `secrets` in the task definition.
  - Provision ACM cert and attach an HTTPS listener.
  - Split execution vs task roles and scope SSM access.

This plan unifies the existing deployment artifacts with production-grade secrets, networking, and security hardening to safely launch on AWS.

