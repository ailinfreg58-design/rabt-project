# ربط (Rabt) — منصة أتمتة SaaS عربية

منصة أتمتة SaaS بالكامل بالعربية RTL، مشابهة لـ n8n، تتيح للمؤسسات بناء workflows بدون كود.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/rabt run dev` — run the frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Required env: `SESSION_SECRET` — secret for express-session (httpOnly cookies)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 + express-session + connect-pg-simple + express-rate-limit
- Auth: bcryptjs (argon2-equivalent strength at cost 12), session-based httpOnly cookies
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Frontend: React + Vite + Tailwind + wouter + framer-motion + react-hook-form

## Where things live

- `lib/api-spec/openapi.yaml` — API contract (source of truth)
- `lib/db/src/schema/` — DB schema: organizations.ts, users.ts, workflows.ts
- `artifacts/api-server/src/routes/` — auth.ts, workflows.ts, org.ts
- `artifacts/api-server/src/middlewares/requireAuth.ts` — session auth guard
- `artifacts/api-server/src/app.ts` — Express setup with sessions + rate limiting
- `artifacts/rabt/src/` — React frontend (RTL, Arabic)
- `artifacts/rabt/src/components/layout/Sidebar.tsx` — main layout + auth guard

## Architecture decisions

- **org_id always from session**: org_id is NEVER accepted from client body/query — always derived from req.session.orgId after authentication. Prevents IDOR and cross-org data leaks.
- **Multi-tenant isolation**: Every DB query on workflows/users filters with `WHERE org_id = <session.orgId>`.
- **Session-based auth**: httpOnly cookie sessions stored in PostgreSQL via connect-pg-simple. No JWTs client-side.
- **Constant-time password compare**: bcrypt.compare() inherently provides constant-time comparison.
- **Rate limiting**: 20 requests per 15 min on /api/auth/login and /api/auth/register.
- **Generic error messages**: Auth errors never reveal whether email exists or not.

## Product

- تسجيل دخول/تسجيل حساب مع إنشاء مؤسسة تلقائية
- لوحة تحكم تعرض إحصائيات الـ workflows (total/active/inactive) وقائمة الـ workflows
- CRUD كامل للـ workflows مع عزل تام بين المؤسسات
- إعدادات المؤسسة مع قائمة الأعضاء
- واجهة عربية RTL بالكامل بهوية بصرية Indigo + Gold + Teal

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Always run `pnpm run typecheck:libs` after changing `lib/*` schemas before typechecking artifact packages
- `pnpm --filter @workspace/api-spec run codegen` must be re-run after every OpenAPI spec change
- SESSION_SECRET env var is required for the API server to start

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
