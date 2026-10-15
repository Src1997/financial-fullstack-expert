# Pack 08 — Auth + Audit

## When to load

- Touching login/register/session/MFA/email-verification flows.
- Adding admin routes or capability checks.
- Changing control-plane write paths (anything that must be audited).

## Scope / Out of scope

**Scope:** Dual-backend auth (local/upstream), security layers, RBAC
capability model, `audit_log()` discipline, endpoint inventory.

**Out of scope:** Platform-L vs Feature-L classification (→ pack 06),
frontend auth UI (→ pack 02 + web auth rules), verification (→ pack 11).

## Must know

### Dual-backend auth (`AUTH_MODE`)

| Backend | Behavior |
| ------- | -------- |
| `local` | Passwords in `fin_users` (bcrypt), JWT issued locally with `jti` |
| `upstream` | Proxy to external auth service; mirror user to `fin_users` via `ensure_local_user()` |

Admin console **always** uses `local` (`force_local=True`).

### Local vs upstream session classification

`_is_local_session()` (`app/domains/auth/service.py`) routes every request
to the local stack or the upstream delegate:

- `aud in ("impersonation", "admin", "client")` → **local**
- claims carry both `jti` + `exp` → **local** (local JWTs always do;
  upstream `claims_from_userinfo` only yields `sub`/`username`/`role`)
- otherwise → **upstream**

Failure mode seen in production debugging: the check only recognized
`aud=impersonation` / `mock.` prefix, so locally issued client JWTs were
forwarded upstream (401), the C-end cleared its token on restore, and
`/navigation/menu` silently ran as anonymous — capability-gated modules
(e.g. chat) disappeared for logged-in users. When touching token issuance
or the session resolver, verify **both** directions: local tokens resolve
locally, upstream tokens still delegate.

### Security layers (in order)

1. **Turnstile** — `require_turnstile_token()` on login/register/send-code.
2. **Rate limit** — `check_login_rate_limit()` per IP + username (sliding window).
3. **JWT** — local tokens carry `jti` for session revocation; upstream tokens are opaque.
4. **Session** — `FinSession` tracks `jti` / `expires_at` / `revoked_at`; logout revokes.
5. **MFA** — upstream-only; `verify_mfa_login()` completes second factor.

### RBAC

- Admin routes require the `AdminUser` dependency (role check).
- Capability checks via `require_capability("capability.name")`.
- Role→capability mapping lives in the `fin_admin_role_capabilities`
  row-level table; read entry `app.shared.rbac.get_role_capabilities()`.
- Admin routes use the unified prefix `/api/admin/{domain}/...`
  (red line #19 — `_context_router` auto-mounts `admin_router.py`).

### Endpoints (inventory)

`POST /api/auth/login`, `/api/auth/admin/login`, `/api/auth/login-code`,
`/api/auth/register` (gated by `registration_enabled`), `/api/auth/send-code`,
`/api/auth/reset-password`, `/api/auth/change-password`,
`/api/auth/mfa/verify-login`, `/api/auth/logout`; `GET /api/auth/info`,
`/api/auth/security-config` (public Turnstile config),
`/api/auth/oauth/{provider}` (google, github).

### Audit discipline

All control-plane writes (user lifecycle, permission matrix, config publish,
crawler scheduling, content hard-delete, script changes) must persist an
immutable record to `fin_audit_logs`:

```python
from app.shared.audit import audit_log

audit_log(
    db,
    request=request,          # IP / UA / request_id
    actor=admin,              # AdminUser dict or user_id string
    action="user.disable",    # dot.separated action type
    target_type="user",
    target_id=str(user_id),
    before={"status": "active"},
    after={"status": "disabled"},
)
```

Rules:

- **Same transaction** — `audit_log()` uses `db.flush()`; rollback removes the
  audit record too (red line #8).
- **Automatic masking** — `password` / `token` / `secret` / `api_key`-like
  fields become `"***"`.
- **Non-blocking** — audit failure logs a warning; business op proceeds.
- **Append-only** — no UPDATE/DELETE path; corrections are compensating entries.
- Admin read API: `GET /api/admin/audit-logs` (`action`, `actorId`,
  `targetType` filters; read-only, requires `AdminUser`).

### Adding auth features

1. Schema in `schemas.py` → service in `service.py` (backend resolver pattern)
   → route in `router.py` or `admin_router.py`.
2. Update seed fixtures if new RCC keys needed.
3. Export OpenAPI + sync frontend contracts (`pnpm api:export` → `api:sync`,
   pack 01).

## Do / Don't

| Do | Don't |
|----|-------|
| Issue local JWTs with `jti` | Don't log passwords, tokens, or full PII |
| Keep Turnstile + rate limit on auth endpoints | Don't bypass them for convenience |
| Audit control-plane writes in the same transaction | Don't substitute `logger.info` / `print` for `audit_log()` |
| Use `require_capability()` for admin authorization | Don't do admin login via the upstream backend |

## Touch map

| What | Path |
| ---- | ---- |
| Auth domain | `financial-api/app/domains/auth/` |
| JWT / password | `financial-api/app/core/security/auth.py` |
| Turnstile | `financial-api/app/core/security/turnstile.py` |
| Login rate limit | `financial-api/app/core/rate_limit/login.py` |
| Session model | `financial-api/app/models/system/session.py` |
| Audit helper | `financial-api/app/shared/audit.py` |
| Audit model / API | `financial-api/app/models/system/audit.py`, `app/domains/system/audit_log/` |
| Auth rule | `financial-api/docs/rules/modules/auth.md` |
| Audit rule | `financial-api/docs/rules/modules/audit.md` |

## SSOT links

| Doc | Path |
| --- | ---- |
| Auth | `financial-api/docs/rules/modules/auth.md` |
| Audit | `financial-api/docs/rules/modules/audit.md` |
| Conventions skill | `financial-api/docs/skills/project/financial-api-conventions/` |
| Auth skill | `financial-api/docs/skills/project/financial-api-auth/` |
| RCC auth keys | `financial-api/docs/rules/modules/app_config.md` (`auth.*`) |

## Related packs

- **06** — RCC + Capability (`policy.write` vs `config.write`)
- **03** — API architecture (layering the new endpoints must follow)
- **11** — Verification (audit + auth test expectations)
