# S.A.M.U.D.E.R.A. — Implementation Plan

> **Status:** Planning document only. **Nothing in this document has been implemented yet.**
> This plan records the gated implementation sequence defined in `ARCHITECTURE.md`. Each phase requires explicit approval before work begins, and must end with tests/build checks, a reviewed diff, and a checkpoint/commit.

---

## 1. Source of Truth & Phase Gating

| Document | Role |
| :--- | :--- |
| `PRD.md` | **Primary source of truth** — requirements, math, policy thresholds, authority semantics. |
| `ARCHITECTURE.md` | Required project structure, module boundaries, security boundaries, implementation sequence. |
| `README.md` | Operator-facing context; must remain consistent with `PRD.md` and `ARCHITECTURE.md`. |
| `docs/IMPLEMENTATION_PLAN.md` | This file — approved per-phase execution plan. |
| `.clinerules/00-samudera.md` | Persistent coding-agent guardrails that bind every future coding session. |

**Phase-gating rule**

- Implement the system **one approved phase at a time** (Phase 0 → Phase 7).
- Do not skip ahead to a later phase before the current phase is approved.
- Each phase ends with tests/build checks, a reviewed diff, and a checkpoint/commit.
- If a documentation conflict is discovered, **stop and report it**. Do not guess. The PRD wins and the affected docs must be updated.
- `DESIGN.md` is **not** created until the UI design workflow is approved (added only after Phase 0 approval discussions; it is not a prerequisite of Phase 1 start unless explicitly approved).

---

## 2. Documentation Notes & Known Discrepancies

Recorded so future sessions do not treat these as PRD conflicts requiring a PRD rewrite:

1. **Repo-root naming (minor):** `ARCHITECTURE.md` and `README.md` render the root folder as `SAMUDERA-App/`; the actual repository root is `SAMUDERA`. All file paths in this plan refer to the actual repo root.
2. **Phase-numbering ambiguity (documented, not a contradiction):** `PRD.md` §11 defines a *product roadmap* ("Phase 1: Virtual Hackathon MVP", "Phase 2: Post-Hackathon", "Phase 3: Enterprise Integration"). `ARCHITECTURE.md` defines the *implementation sequence* (Phase 0–7). These are distinct and must not be conflated in plans, commits, or status reports.
3. **Backend deployment emphasis (minor):** `PRD.md`/`README.md` allow "Vercel Python/FastAPI runtime or an equivalent ASGI host if deployment constraints require it"; `ARCHITECTURE.md` states Vercel as the baseline. The API contract must remain host-independent; deployment specifics are resolved in Phase 7 based on constraints at that time.

No PRD-level conflicts were found. Policy thresholds (0.60 / 0.85), physical bands (0.50 / 0.80 / 1.00), agent authority rules, security boundaries, route contract, and data-honesty rules are consistent across all three documents.

---

## 3. Cross-Cutting Requirements (Apply to Every Phase)

### 3.1 Security & Secrets

- Browser clients receive **only** browser-safe Supabase publishable credentials.
- Supabase secret/service-role credentials must never appear in `NEXT_PUBLIC_*`, browser bundles, client-side JavaScript, or source control.
- Server-only credentials are restricted to trusted administrative/ingestion operations.
- The runtime LLM/agent never receives Supabase service-role credentials, provider API secrets, or unrelated tenant data.
- All public traffic uses HTTPS with TLS 1.2 or newer.

### 3.2 Data Honesty

- Shifted Kattegat AIS = **simulated/historical**; never presented as live telemetry.
- ERA5 baseline = **static**; UI wind/wave sliders are what-if simulation overrides.
- GEBCO = **bathymetry / water depth only**; never derive mud/sand substrate from GEBCO.
- `K_soil` = separate configured/mocked corridor profile, not GEBCO-derived.
- Copernicus currents = downloaded snapshot for the MVP unless automated ingestion is implemented.
- Telco topology/SLA = mocked unless connected to a real tenant dataset.

### 3.3 Architecture Discipline

- Frontend must never reimplement authoritative physics, anomaly, consequence, or policy logic in TypeScript.
- Backend owns all authoritative risk computation and policy evaluation.
- Agentic Incident Response remains post-policy, read-only, advisory, and auditable.
- PostGIS is the production spatial-query layer; GeoPandas/Shapely are ETL/preprocessing tools.
- Large source datasets and secrets must not be committed to Git.

### 3.4 Checkpoint Policy

Every phase closes with:

1. All acceptance criteria met.
2. Tests/build checks passing (per-phase checks listed below).
3. A reviewed diff.
4. A checkpoint/commit before the next phase begins.

---

## 4. Phase 0 — Project Control and Contracts

**Goal:** Lock the repository structure, documentation hierarchy, and persistent guardrails so every subsequent phase operates under the same contracts. No application code is created.

**Files/folders involved**

- `.clinerules/00-samudera.md` — persistent coding-agent guardrails (created; see section 1).
- `docs/IMPLEMENTATION_PLAN.md` — this plan (created; reviewed per ARCHITECTURE Phase 0).
- `PRD.md`, `ARCHITECTURE.md`, `README.md` — authoritative docs; kept consistent.
- Planned (future) repository structure to be scaffolded only in the phase where it is needed:
  - `frontend/` — Next.js NOC application (Phase 1)
  - `backend/` — FastAPI authoritative engine (Phase 2)
  - `supabase/migrations/`, `supabase/seed.sql` (Phase 4)
  - `data/` — local/sample data directory (Phase 5; present earlier only as empty, gitignored placeholder if approved)
  - `.gitignore`, `vercel.json` — repo hygiene / optional deployment routing
- `DESIGN.md` — **NOT created** in Phase 0. Delegated until the UI design workflow is approved.

**Dependencies**

- `PRD.md`, `ARCHITECTURE.md`, `README.md` (already present and read in full).
- Explicit user approval of this implementation plan.

**Tests/checks**

- Documentation cross-read: verify `PRD.md`, `ARCHITECTURE.md`, `README.md`, `.clinerules/00-samudera.md`, and this plan are mutually consistent.
- Diff review of the two new files.
- Git status clean except the intended new files.

**Acceptance criteria**

- `.clinerules/00-samudera.md` exists and encodes the guardrails in section 3.
- `docs/IMPLEMENTATION_PLAN.md` exists and is approved.
- `DESIGN.md` is absent.
- No frontend, backend, Supabase, MCP, or external API artifacts exist.

**What must NOT be done yet**

- No `frontend/` or `backend/` scaffolding, no `npm create`/`uvicorn` setup, no package installation.
- No Supabase project, MCP connection, or credentials (even development).
- No external API calls (Submarine Cable Map, Copernicus, GEBCO, ERA5).
- No `DESIGN.md` creation.
- No physics, policy, or ML code.

---

## 5. Phase 1 — Navigable Frontend Shell with Typed Mocks

**Goal:** Build the persistent, navigable Next.js NOC application shell with all MVP routes rendering typed mock fixtures, so navigation and information architecture are validated before any backend exists.

**Files/folders involved**

- `frontend/package.json`, `frontend/tailwind.config.js`, Next.js App Router config
- `frontend/src/app/` — routes:
  - `layout.tsx`, `page.tsx` (redirect/entry to `/dashboard`)
  - `dashboard/page.tsx` — 3D Mersing map, KPI cards, active-threat queue, what-if wind/wave controls (UI only)
  - `incidents/page.tsx` — filterable `MONITOR` / `PREPARE BACKUP` / `ESCALATE` queue
  - `incidents/[id]/page.tsx` — timeline, physics/anomaly evidence, cable/network consequence, policy state, agent brief placeholder, approval controls
  - `vessels/page.tsx`, `cables/page.tsx`, `policies/page.tsx`, `observer/page.tsx`
- `frontend/src/components/` — `shell/`, `map/`, `incidents/`, `risk/`, `agent/`, `policy/`, `alerts/`, `ui/`
- `frontend/src/lib/mock/` — typed mock fixtures mirroring the backend contract
- `frontend/src/types/` — shared frontend TypeScript types (aligned to backend schemas later)
- `frontend/src/lib/supabase/` — **browser-safe** client placeholder only (no credentials required for mocks)

**Dependencies**

- Phase 0 approval (guardrails + plan).
- Next.js 14+, React, TypeScript, Tailwind CSS, Lucide icons, Deck.gl + React Map GL / Mapbox-compatible basemap.
- Mock fixtures must be typed to match the intended backend API contract (designed in Phase 2 schemas, mocked now).

**Tests/checks**

- `npm run lint`, `npm run build`, `npm run test` (or equivalent) pass.
- `npx next build` succeeds with all 7 routes.
- Manual/automated navigation check: shell persists across routes; deep links map → incident work.
- Type checks pass for all mock fixtures.

**Acceptance criteria**

- All MVP routes exist and render with typed mock data; no backend calls required.
- Persistent shell shows tenant identity, role, alert-count indicators, sidebar/topbar nav.
- What-if wind/wave sliders are present in the UI **as simulation overrides only** (clearly labeled; no live-weather claim).
- Agent brief area in `/incidents/[id]` is a placeholder rendering mock/empty-evidence state — never presented as executed action.
- `/observer` renders restricted read-only `ESCALATE` context.
- Frontend contains zero authoritative physics/anomaly/consequence/policy calculations in TypeScript.

**What must NOT be done yet**

- No backend code or API client wiring to a live service.
- No Supabase Auth/Realtime/RLS wiring (client placeholders only).
- No real data ingestion; fixtures only.
- No authoritative calculations in the browser.
- No agentic tool execution.

---

## 6. Phase 2 — Deterministic Backend

**Goal:** Implement the authoritative deterministic engine — schemas, physics, IsolationForest, threat fusion, network consequence, and policy — with replaceable data/repository interfaces so it runs and is tested without Supabase.

**Files/folders involved**

- `backend/requirements.txt`, FastAPI app structure
- `backend/app/main.py` — entrypoint
- `backend/app/api/` — HTTP route handlers
- `backend/app/core/` — configuration, auth/JWT helpers (placeholders)
- `backend/app/schemas/` — Pydantic request/response schemas (source for Phase 1 type alignment)
- `backend/app/services/`
  - `spatial.py` — PostGIS query + GeoPandas preprocessing layer (repository interface)
  - `physics_engine.py` — wind/current/wave vectors, `F_hold,crit`, `R_drag`
  - `ml_anomaly.py` — IsolationForest trajectory features (SOG, COG, dwell)
  - `threat_fusion.py` — physics + anomaly + cable interaction
  - `consequence.py` — criticality / effective redundancy (≥1) / SLA scoring
  - `policy_engine.py` — tenant-configurable thresholds (0.60 / 0.85 defaults)
  - `dispatch.py` — draft dispatch generation (no transmission)
- `backend/app/db/` — repository helpers; in-memory/file fixtures so tests run without Supabase
- `backend/tests/` — unit + integration tests

**Dependencies**

- Phase 1 approval (mock contract alignment).
- Python 3.12+, FastAPI, Pydantic, scikit-learn, GeoPandas/Shapely (for preprocessing), `xarray`/NetCDF parsing (for ingestion-readiness; not required to run deterministic tests).
- No Supabase required; repositories are replaceable.

**Tests/checks**

- `pytest` suite for: physics vectors, `R_drag` bands, IsolationForest anomaly flag, threat fusion, consequence scoring, policy thresholds.
- Verify the documented default policy thresholds: `MONITOR` `<0.60`, `PREPARE BACKUP` `0.60–0.85` with vessel slowing in buffer, `ESCALATE` `≥0.85` or anomaly over high-criticality cable.
- Confirm `MONITOR` / `PREPARE BACKUP` / `ESCALATE` are produced by the policy engine (not the ML model alone).
- API contract tests for schemas.

**Acceptance criteria**

- Physics engine computes `F_wind`, `F_current`, `F_wave`, `F_hold,crit`, `R_drag` deterministically per PRD §7 equations.
- IsolationForest baseline is used; `R_drag` is a load/susceptibility ratio, never a probability.
- Policy engine is the authoritative source of operational state.
- Repository/data-access interfaces are replaceable; full test suite passes with no Supabase.
- Every endpoint returns data matching `backend/app/schemas/`.

**What must NOT be done yet**

- No Supabase connection, migrations, or RLS code.
- No real data ingestion (GEBCO, Copernicus, ERA5, AIS, cable GeoJSON).
- No agentic layer.
- No external LLM provider calls.
- No frontend integration (Phase 3).

---

## 7. Phase 3 — Local Frontend/Backend Integration

**Goal:** Connect the Next.js API client to FastAPI over stable typed contracts, replacing mock responses incrementally while keeping fixtures for tests and fallback demos.

**Files/folders involved**

- `frontend/src/lib/api/` — FastAPI client/fetchers (typed against backend schemas)
- `frontend/src/types/` — synchronized with `backend/app/schemas/`
- `frontend/src/lib/mock/` — retained as fallback/test fixtures
- `backend/app/api/` — refined route handlers consumed by the frontend
- Contract fixtures shared by both sides (schema-derived samples)

**Dependencies**

- Phase 2 approval (backend deterministic engine + schemas stable).
- Phase 1 frontend shell present.
- Local FastAPI dev server (`uvicorn`) and Next.js dev server (`npm run dev`).

**Tests/checks**

- Contract tests: response payloads match Pydantic schemas exactly.
- Route-by-route integration: `/dashboard`, `/incidents`, `/incidents/[id]`, `/vessels`, `/cables`, `/policies`, `/observer`.
- Frontend build/lint passes with live API responses.
- Fallback path verified: mocks still render if API is unavailable.

**Acceptance criteria**

- The dashboard displays backend-authoritative `R_drag`, force breakdown, anomaly, consequence, and policy state.
- What-if sliders trigger backend simulation recomputation and update UI results without the browser reimplementing calculations.
- Incident workspace shows a real (pre-agent) evidence summary from backend services.
- Typed contract drift is caught by tests (any schema change breaks a contract test).
- No secrets in frontend code; API base URL configured via `NEXT_PUBLIC_API_BASE_URL`.

**What must NOT be done yet**

- No Supabase/PostGIS usage (repositories still in-memory/file-backed).
- No real ingestion datasets.
- No agentic incident-response tool calling.
- No Realtime subscriptions.

---

## 8. Phase 4 — Supabase/PostGIS Development Integration

**Goal:** Introduce the development Supabase/PostGIS layer after schema review — migrations, GiST indexes, tenant-membership model, RLS, and generated TypeScript database types — without letting MCP redesign the schema silently.

**Files/folders involved**

- `supabase/migrations/` — tables, PostGIS extension, GiST indexes, RLS policies, tenant membership
- `supabase/seed.sql` — demo tenants/policies/topology (clearly mocked)
- `backend/app/db/` — repository implementations backed by Supabase (kept behind replaceable interfaces)
- `frontend/src/lib/supabase/` — browser-safe publishable client; use of tenanted tables via RLS
- Generated TypeScript database types (after migrations are stable)

**Dependencies**

- Phase 3 approval (contracts stable).
- A **development** Supabase project (never production).
- PostGIS enabled; migrations reviewed before any apply.
- Controlled MCP/CLI application of migrations — never silent schema redesign.

**Tests/checks**

- Migration review diff (tables, indexes, RLS, seed).
- RLS tests: tenant A cannot read tenant B data; `auth.uid()` + tenant-membership/role verified.
- PostGIS GiST index present on spatial columns.
- `ST_DWithin` benchmark check (target `< 50 ms` at the query layer for MVP dataset).
- Generated TypeScript types compile against frontend clients.

**Acceptance criteria**

- RLS protects all tenant-scoped tables; no secret/service-role key in browser code or source control.
- Repositories now use PostGIS for production spatial queries; GeoPandas/Shapely remain ETL/preprocessing only.
- Application runs against the development Supabase project with mocked seed data.
- Schema changes flow through reviewed migrations, not MCP auto-generation.

**What must NOT be done yet**

- No production credentials or production project.
- No real external data ingestion (Phase 5).
- No agentic layer.
- No Realtime wiring (Phase 7) beyond what dev integration requires.

---

## 9. Phase 5 — MVP Data Ingestion

**Goal:** Integrate the approved MVP datasets with their documented semantics preserved and data-honesty labels enforced.

**Files/folders involved**

- `backend/app/services/ingestion/` — cable, AIS, GEBCO, Copernicus, ERA5 loaders/normalizers
- `data/` — local/sample data only; large/private files gitignored:
  - `data/ais/` — shifted Kattegat AIS (coordinate-shifted, labeled simulated/historical)
  - `data/bathymetry/` — GEBCO 2026 bounded slice (Mersing corridor)
  - `data/metocean/` — Copernicus current snapshot + static ERA5 baseline
  - `data/fixtures/` — configured `K_soil` profile, mocked telco topology/SLA
- `supabase/migrations/` — any new tables for normalized ingested data (reviewed)
- `backend/app/services/spatial.py` — proximity checks via PostGIS `ST_DWithin` / `ST_Intersects`

**Dependencies**

- Phase 4 approval (Supabase/PostGIS available for normalized data).
- Approved source artifacts: Submarine Cable Map v3 GeoJSON (Mersing systems: AAG, ASE, East-West, SEAX-1, SKR1M); shifted historical AIS; GEBCO 2026 slice; configured/mocked `K_soil`; Copernicus NetCDF snapshot; static ERA5; mocked telco data.

**Tests/checks**

- Ingestion tests per source: normalization, schema validation, caching.
- Data-honesty assertions: labels ("simulated/historical", "static") present in UI-facing metadata.
- GEBCO-derived values are depth-only; no mud/sand classes generated.
- `K_soil` loads from the configured/mocked corridor profile, not GEBCO.
- Spatial query benchmark on real MVP dataset (target `< 50 ms`).
- AIS shift verified against Mersing corridor bounds.

**Acceptance criteria**

- All MVP data sources load and normalize into the internal schema.
- No source is mislabeled; labels are consistently enforced.
- Physics/anomaly/consequence runs against ingested data end-to-end.
- Cached artifacts stored in Supabase Storage where appropriate.
- Data pipeline produces the same deterministic results on re-runs for the same inputs.

**What must NOT be done yet**

- No live AIS streaming, no automated weather ingestion, no validated sediment source (post-MVP per PRD §11).
- No agentic layer.
- No Realtime/approval workflow wiring (Phase 7).
- No changes to documented data semantics.

---

## 10. Phase 6 — Bounded Agentic Incident Response

**Goal:** Add the server-side, post-policy, read-only, auditable Incident Response Agent that investigates `PREPARE BACKUP` / `ESCALATE` events or operator requests and produces evidence-grounded briefs + recommended playbooks.

**Files/folders involved**

- `backend/app/agent/`
  - `orchestrator.py` — multi-step tool-calling incident loop
  - `tools.py` — allowlisted read-only tools: `get_threat_snapshot`, `get_recent_trajectory`, `get_physics_breakdown`, `get_anomaly_result`, `get_cable_context`, `get_network_consequence`, `get_tenant_policy`
  - `schemas.py` — structured input/output contracts (authoritative state, evidence refs, uncertainty flags, recommendation)
  - `provider.py` — server-side tool-capable LLM provider adapter (config-driven, credentials server-only)
  - `guardrails.py` — authority, tenant-scope, evidence-completeness checks
- `backend/app/db/` — persistence of agent runs, tool-call traces, evidence references, model/provider metadata, human outcome (audit records)
- `supabase/migrations/` — agent audit tables (reviewed)

**Dependencies**

- Phase 5 approval (deterministic outputs tested and stable — the PRD requires this ordering).
- Server-side LLM provider adapter + credentials (server-only).
- Deterministic services remain the sole source of risk/policy state.

**Tests/checks**

- Tool allowlist enforcement: no write-capable tool exposed to the agent.
- Tenant-scope tests: agent tool calls inherit the operator's tenant and cannot read unrelated tenant data.
- Trigger tests: agent invoked for `PREPARE BACKUP`/`ESCALATE` or explicit operator request; not for `MONITOR` unless requested.
- `NEEDS HUMAN REVIEW` path tested with incomplete/contradictory/stale evidence.
- Auditability: every run records trigger, tool calls, evidence refs, model/provider metadata, recommendation, and human outcome.
- Guardrail tests: agent cannot modify `R_drag`, anomaly scores, consequence, policy state, tenant rules, alert acknowledgements, routing, or external comms.

**Acceptance criteria**

- Agent produces structured, evidence-grounded briefs distinguishing retrieved/calculated evidence from model narrative.
- Randomization/parameterization of provider choice does not affect deterministic engine outputs.
- For `ESCALATE`, agent may prepare/refine a draft dispatch for human review only.
- Audit trail persisted and reproducible from source evidence.
- No external side effects (no transmissions, no rerouting, no acknowledgements).

**What must NOT be done yet**

- No agent authority over deterministic state or external actions (hard boundary maintained).
- No autonomous dispatch transmission.
- No production LLM credentials in frontend code or source control.
- No Realtime wiring of agent events (Phase 7).

---

## 11. Phase 7 — Realtime, Approval Workflow, Testing and Deployment

**Goal:** Connect Supabase Realtime for alert/brief/approval events, validate role-based navigation and human-approval paths, run performance/security tests, and deploy the demo.

**Files/folders involved**

- `supabase/migrations/` — Realtime publication configuration (only needed tables/events), RLS refinement if required
- `frontend/src/lib/supabase/` — Realtime subscriptions for alert acknowledgements, brief updates, approval outcomes
- `frontend/src/components/alerts/`, `frontend/src/components/agent/` — approval/rejection controls wired to authoritative backend
- `frontend/vercel.json`, `backend` deployment config — deployment routing
- `.env` / `.env.local` guidance (README §"How to Operate") — browser-safe keys only in `NEXT_PUBLIC_*`
- Performance/security test artifacts and test suite

**Dependencies**

- Phase 6 approval (agent layer tested).
- Supabase Realtime enabled for the required tables/events.
- Deployment target decision: Vercel FastAPI/Python runtime **or** equivalent ASGI host — resolved here per constraints (see section 2, note 3). API contract remains host-independent.

**Tests/checks**

- Realtime propagation benchmark (target `< 300 ms` alert-state propagation under the designated demo environment).
- Approval workflow end-to-end tests: acknowledge alert, review brief, approve/reject dispatch draft; UI never presents agent recommendations as executed actions.
- Role-based navigation enforcement on all routes, including `/observer` read-only restriction.
- Frontend rendering benchmark: up to 2,000 vessel vectors at `≥ 50 FPS` on the designated demo/reference device.
- Security checks: no secrets in bundles/source control; RLS re-verified; HTTPS/TLS 1.2+ required.
- Production build: `next build`, backend startup/health checks, migration apply on target environment.

**Acceptance criteria**

- Realtime updates propagate within the documented target; multi-user state stays consistent.
- Human approval is the only path to external dispatch; no autonomous transmission anywhere.
- All routes respect roles; `/observer` is verified read-only.
- Performance targets met under documented benchmark conditions.
- Demo deployable and reproducible from the checked-in artifacts; `.env` secrets documented without committing values.

**What must NOT be done yet after Phase 7**

- No expansion beyond the Mersing corridor unless approved (roadmap scope, PRD §11).
- No live AIS streaming, automated weather ingestion, or validated sediment source (post-MVP roadmap).
- No unbounded agent tools or agent authority over operational state.
- No silent architecture changes — any evolution goes through the documentation + approval process.

---

## 12. Phase Completion Checklist (Every Phase)

- [ ] Phase approved before work began.
- [ ] All acceptance criteria for the phase met.
- [ ] Tests/build checks pass as listed for the phase.
- [ ] Diff reviewed.
- [ ] Checkpoint/commit created.
- [ ] Documentation updated if a conflict or change was discovered (PRD-wins rule).