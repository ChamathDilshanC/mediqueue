# MediQueue

Main integration repository for the MediQueue healthcare queue platform.

The implementation is maintained in pinned child repositories:

- `backend/` - [mediqueue-backend](https://github.com/ChamathDilshanC/mediqueue-backend)
- `frontend/` - [mediqueue-frontend](https://github.com/ChamathDilshanC/mediqueue-frontend)
- `configuration/` - [mediqueue-configuration](https://github.com/ChamathDilshanC/mediqueue-configuration)

## Structure

- `backend/` - pinned backend submodule containing FastAPI, migrations, worker and tests.
- `configuration/` - pinned configuration submodule containing JSON Schema-validated snapshots.
- `frontend/` - pinned frontend submodule for the Next.js client.
- `docs/` - architecture, stack and repository rules.

## Local development

```powershell
git clone --recurse-submodules https://github.com/ChamathDilshanC/mediqueue.git
Set-Location backend
Copy-Item .env.example .env
python -m pip install -e '.[test]'
python -m backend.init_db
pytest
python -m uvicorn backend.main:app --reload
```

Validate the pinned configuration separately from the repository root:

```powershell
python configuration\validate.py configuration\defaults.json
```

PostgreSQL is the production database. SQLite is supported for local development and tests. Run `alembic upgrade head` against the configured PostgreSQL database before deploying the API. Authentication requires a verified Supabase JWT in staging/production; development-header authentication is disabled by default.

## API documentation and operational endpoints

Start the API and open:

- Themed API reference: `http://127.0.0.1:8000/docs`
- Swagger playground: `http://127.0.0.1:8000/swagger`
- ReDoc: `http://127.0.0.1:8000/redoc`
- OpenAPI JSON: `http://127.0.0.1:8000/openapi.json`

The generated documentation includes request fields, response models, authentication requirements, role/scope behavior, error responses, idempotency requirements and queue-state side effects.

The API now provides 67 method/path operations, including user registration/login/recovery, hospital/branch onboarding, database-backed memberships, department/room/doctor/schedule management, patients, appointments, visits and queue operations. See [the backend API guide](backend/README.md) for the full resource matrix and rollout requirements. The documentation uses the requested white/lime theme with responsive navigation, live search and request/response examples. Application frontend implementation remains separate.

| Method | Endpoint | Purpose | Authentication |
| --- | --- | --- | --- |
| `GET` | `/health` | API liveness; does not contact the database | Public |
| `GET` | `/health/ready` | Executes `SELECT 1` against the configured database | Public |
| `GET` | `/v1/config/public` | Returns only the configuration snapshot's `publicAllowlist` keys | Public |
| `GET` | `/v1/queues/{queue_id}/snapshot` | Returns token labels/statuses for an authorized branch | Supabase JWT + staff role |
| `POST` | `/v1/queues/{queue_id}/tokens` | Checks a patient into a queue | Supabase JWT + reception/staff/admin + `Idempotency-Key` |
| `POST` | `/v1/queues/{queue_id}/call-next` | Calls one waiting token transactionally | Supabase JWT + doctor/staff/admin + `Idempotency-Key` |
| `POST` | `/v1/tokens/{token_id}/{action}` | Performs `recall`, `skip`, or `complete` | Supabase JWT + authorized role |

Database connection status is intentionally exposed only as `ready`/`unavailable`; credentials, hostnames and SQL errors are never returned by the health endpoint. The readiness check uses the same `DATABASE_URL` loaded by the application.

## Supabase and Google login

The API verifies Supabase-issued access tokens and resolves permissions from active local memberships. Email/password login and registration are available through `/v1/auth/*`. Google login is configured in Supabase Auth and can be initiated by a future web client with `signInWithOAuth({ provider: "google" })`; Google credentials must not be placed in this backend repository.

1. Create a Supabase project.
2. In **Authentication → Providers → Google**, enable Google and add the Google OAuth client ID/secret.
3. Add the local and deployed callback URLs shown by Supabase to Google Cloud OAuth credentials.
4. Copy the Supabase project URL into `SUPABASE_URL` and the public anon key into `SUPABASE_ANON_KEY`.
5. Configure `SUPABASE_JWKS_URL` (or the project JWT secret where applicable) for backend token verification.
6. Keep `SUPABASE_SERVICE_ROLE_KEY`, database passwords and OAuth client secrets only in the deployment secret store.

The current `.env` is a local, secret-free SQLite configuration. Replace only the empty Supabase values when the project is available; do not commit secrets.
