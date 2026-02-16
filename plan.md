# TinyClaw SaaS Plan: Frictionless Agent Subscriptions

## Objective

Build a hosted product where customers can subscribe, select a skills bundle, connect channels (Telegram first), and get a production-ready AI agent runtime in minutes.

---

## Product Strategy

### Core user promise

- Buy a plan
- Paste Telegram bot token (and optionally provider key)
- Pick a bundle
- Go live immediately

### Suggested plans

- **Starter**: single agent, single channel, capped usage
- **Pro**: multi-agent team, higher limits, basic analytics
- **Business**: custom bundles, managed keys, SSO/admin controls

---

## Architecture Direction

### 1) Control Plane (new service)

Responsibilities:

- Auth, billing, subscriptions
- Tenant creation and lifecycle (create/suspend/delete)
- Secrets management (channel tokens, BYOK keys)
- Bundle catalog + assignment
- Runtime orchestration API

### 2) Execution Plane (TinyClaw runtimes)

Run TinyClaw as **per-tenant isolated runtime**:

- Isolated settings, queues, logs, files, workspace
- Isolated channel tokens
- Isolated model/provider credentials
- Independent start/stop/restart

### 3) Data model (high-level)

- `tenants`
- `subscriptions`
- `bundles`
- `bundle_versions`
- `deployments`
- `credentials` (encrypted references)
- `usage_events`

---

## Bundle System Design

Define bundle as deployable template:

- Agents config (roles, provider/model defaults)
- Teams config (leader + handoff topology)
- Prompt templates (`AGENTS.md`, heartbeat prompt)
- Skill package references
- Policy defaults (message limits, max chain depth, file handling rules)

### Bundle delivery flow

1. User chooses plan + bundle
2. Control plane resolves latest compatible bundle version
3. Runtime bootstrap writes `settings.json` and applies templates
4. Runtime starts and health-check passes
5. User receives “bot live” confirmation

---

## Credentials & Model Access

Support two modes:

1. **BYOK** (user-provided keys)
2. **Managed key routing** (your OpenRouter/provider account via proxy)

Recommendation:

- Implement a provider gateway service to centralize model routing, metering, and failover
- Keep tenant-level cost accounting in control plane
- Add per-plan model allowlists

---

## Onboarding UX

### MVP onboarding steps

1. Create account
2. Choose plan
3. Connect Telegram token
4. Pick model mode (BYOK or managed)
5. Pick bundle
6. Click deploy
7. Verify bot with test message

### Operational UX

- One-click pause/resume
- Logs and health surface
- Simple “reset memory/session” control
- Quick bundle switch with safe migration checks

---

## Deployment Plan

## Phase 0 — Foundations (1–2 weeks)

- Define tenant schema and deployment states
- Build minimal control API
- Containerize TinyClaw runtime with explicit config injection
- Add runtime health endpoint/check script

## Phase 1 — MVP (2–4 weeks)

- Stripe billing integration
- Telegram-only onboarding
- Starter + Pro bundles
- Per-tenant runtime provisioning and teardown
- Basic dashboard: status, last messages, restart

## Phase 2 — Reliability (2–3 weeks)

- Queue depth and runtime health monitoring
- Autoscaling workers for deployment jobs
- Alerting (runtime down, auth failure, cost anomalies)
- Backup and retention policies

## Phase 3 — Advanced Product (ongoing)

- WhatsApp/Discord self-serve connectors
- Team visual analytics in dashboard
- Bundle marketplace and versioning UI
- RBAC + audit logs + enterprise controls

---

## Security & Compliance Checklist

- Encrypt all secrets at rest
- Never expose raw provider keys in runtime logs
- Separate tenant storage paths and process identities
- Add rate limits per tenant/channel
- Add abuse detection and kill switch
- Prepare basic DPA/ToS/privacy policy before launch

---

## Metrics to Track

- Time-to-live-bot (signup to first successful response)
- Activation rate (sent first message within 24h)
- 7-day retention per plan
- Cost per tenant/day (model + infra)
- Gross margin by bundle
- Failure rates (deployment, channel auth, model invocation)

---

## Immediate Next Actions (this week)

1. Decide runtime isolation model (container-per-tenant recommended)
2. Define v1 bundle schema in JSON
3. Build `/deploy` API that writes tenant config + starts runtime
4. Add deployment state machine (`pending -> provisioning -> healthy -> failed`)
5. Pilot with 3 internal tenants using Telegram only

