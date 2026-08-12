# S.A.M.U.D.E.R.A. — Persistent Coding-Agent Guardrails

> **Source of truth:** `PRD.md` is the primary source of truth for this project. `ARCHITECTURE.md` defines the required project structure and module boundaries. `README.md` provides operator-facing context. If a documentation conflict is discovered, **stop and report it** — do not guess. Resolve per the PRD-wins rule and update the affected docs.

## 1. Authority & Documentation

- Treat `PRD.md` as the **primary source of truth** for all requirements, math, policy thresholds, and authority semantics.
- Follow `ARCHITECTURE.md` for project structure, module boundaries, and the implementation sequence.
- Keep `README.md` consistent with `PRD.md` and `ARCHITECTURE.md`.
- **Never invent architecture changes silently.** Any structural, module, or contract change must be proposed, reviewed, and recorded in the documentation before implementation.
- **Stop and report documentation conflicts** instead of guessing. If a conflict is real, the PRD wins and the affected docs must be updated.

## 2. Frontend / Backend Separation

- Preserve the documented separation between the `frontend/` (Next.js presentation & human-control plane) and `backend/` (FastAPI authoritative engine).
- The **frontend displays results and requests simulations only**; it must never reimplement authoritative calculations in TypeScript.
- The **backend owns all authoritative risk computation and policy evaluation**:
  - Physics drag engine (`R_drag`, force vectors)
  - IsolationForest trajectory-anomaly model
  - Threat fusion
  - Network consequence scoring
  - Configurable policy engine (`MONITOR` / `PREPARE BACKUP` / `ESCALATE`)

## 3. Agentic Incident Response Layer

- The Agentic Incident Response layer is **post-policy, read-only, and advisory**.
- It may select and sequence **allowlisted, tenant-scoped, read-only** tools to gather authoritative evidence and produce an evidence-grounded incident brief + recommended playbook.
- It **cannot** modify `R_drag`, anomaly scores, network-consequence values, policy state, tenant rules, alert acknowledgement state, network routing, or external communications.
- If evidence is incomplete, stale, or contradictory, the agent must return `NEEDS HUMAN REVIEW` — never invent missing facts.
- `ESCALATE` outputs remain human-in-the-loop: external dispatch requires an authorized human to review and approve.
- Every agent run and tool call must be auditable (trigger, tool calls, evidence references, model/provider metadata, recommendation, human outcome).

## 4. Security & Secrets

- **Never expose Supabase secret/service-role credentials to frontend code.**
- Secret/service-role credentials must not appear in `NEXT_PUBLIC_*` variables, browser bundles, client-side JavaScript, or source control.
- Browser clients receive only browser-safe Supabase publishable credentials.
- Server-only credentials are used only for trusted administrative/ingestion operations that intentionally require elevated access.
- The runtime LLM/agent never receives Supabase service-role credentials, provider API secrets, or unrelated tenant data.

## 5. Data Honesty

- **Shifted Kattegat AIS** must be labeled **simulated/historical**; never present it as live vessel telemetry.
- **ERA5 baseline** must be labeled **static**; UI wind/wave sliders are what-if simulation overrides, not live weather ingestion.
- **GEBCO** is used for **bathymetry / water depth only**. Never derive or display mud/sand substrate classes from GEBCO.
- The substrate holding factor `K_soil` comes from a separate **configured or mocked corridor profile** for the MVP, not from GEBCO.
- Copernicus currents for the MVP use a downloaded snapshot unless automated ingestion is actually implemented.
- Telco topology/SLA inputs are mocked unless connected to a real tenant dataset.

## 6. Implementation Discipline

- Implement the system **one approved phase at a time** (Phase 0 → 7), per `ARCHITECTURE.md`.
- Do not skip ahead to a later phase before the current phase is approved.
- Each phase must end with tests/build checks, a reviewed diff, and a checkpoint/commit before the next phase begins.
- Do not connect remote APIs, Supabase MCP, or production-like credentials during Phase 0.
- `DESIGN.md` is added only after the UI design workflow is approved; do not create it early.