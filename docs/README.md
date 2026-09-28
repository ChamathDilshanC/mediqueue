# MediQueue

MediQueue is a web-first healthcare queue and patient-flow platform with a future mobile client.

**Project owner:** ChamathDilshanC

## Documentation

- [Architecture](architecture.md): repositories, services, queue consistency, security and configuration flow.
- [Technology stack](TECH_STACK.md): implementation choices and repository mapping.
- [Repository rules](AGENTS.md): attribution, code boundaries and release workflow.

The intended GitHub organization has four repos: `mediqueue`, `mediqueue-frontend`, `mediqueue-backend` and `mediqueue-configuration`. Once created, the latter three are pinned as submodules under `frontend/`, `backend/` and `configuration/`.

The backend is planned as a Python service built with FastAPI. The backend repository will contain the FastAPI application entry point (`main.py`), domain services, migrations, tests and the notification worker.

### Configuration delivery

Application configuration is maintained in the separate `mediqueue-configuration` repository and included in this repository as the `configuration/` Git submodule. Vercel deployment must:

1. Check out the exact submodule commit pinned by the main repository.
2. Validate the configuration schema and environment/tenant overrides.
3. Build the validated, immutable configuration snapshot into the FastAPI deployment.
4. Serve only the allowlisted public settings to browser clients through the Config API.

The running application must not fetch configuration from GitHub during a request or queue operation. Configuration changes require a reviewed configuration commit, a submodule pointer update and a new Vercel deployment. Secrets remain in Vercel environment variables or the relevant managed secret store and never enter the configuration repository.

The FastAPI backend is hosted on Vercel as serverless Python functions. Long-running notification processing must not depend on a Vercel function remaining alive; it will run through a separately deployed worker or managed job that consumes the PostgreSQL outbox.

### Backend commenting standard

Backend source files must use professional, purpose-focused documentation:

- `main.py` must begin with a module docstring describing the application entry point and its responsibility.
- Public route handlers and service methods must use concise docstrings that explain what the method does, its important inputs, its return value and any relevant authorization or side effects.
- Add inline comments only for non-obvious decisions, such as transaction boundaries, idempotency handling or queue-state invariants. Do not comment self-explanatory Python code.
- Comments must describe the current behavior and must be updated whenever the implementation changes.

This repository currently contains the architecture documentation. The remote repos and implementation must be set up before the application can run.
