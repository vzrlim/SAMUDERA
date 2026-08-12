# S.A.M.U.D.E.R.A.

**Spatial Awareness for Maritime Understanding, Decision Enablement & Response Assistance**

S.A.M.U.D.E.R.A. is an enterprise **GeoAI + physics-informed + bounded agentic AI spatial decision-support platform** for Telecommunication Network Operations Centers (NOCs). It acts as a Translation and Decision Layer above raw vessel/cable detection by combining vessel telemetry, subsea cable geometry, bathymetry, metocean forces, anomaly detection, network consequence, tenant-configurable policy rules, and a tool-calling Incident Response Agent that investigates flagged events and prepares an evidence-grounded response plan.

The MVP produces human-in-the-loop recommendations:

* `MONITOR`
* `PREPARE BACKUP`
* `ESCALATE`

> **Important:** The MVP contains a real **agentic AI orchestration layer**, but it is deliberately bounded. The agent may autonomously choose and sequence approved read-only investigation tools; it cannot override deterministic risk/policy outputs, contact vessels/authorities, acknowledge alerts, or reroute network traffic. `ESCALATE` still requires authorized human approval for external action.

---

## 📖 Project Brief

Submarine telecommunications cables carry the majority of international data traffic, while dragged anchors are a major cause of subsea cable faults. S.A.M.U.D.E.R.A. is designed to reduce alarm fatigue by moving beyond simple vessel-to-cable proximity alerts.

The system fuses:

1. **Shifted historical AIS telemetry** for simulated vessel movement over the Mersing corridor.
2. **Subsea cable geometries** from the Submarine Cable Map / TeleGeography-derived v3 endpoints.
3. **GEBCO 2026 bathymetry** for water depth.
4. A separate **configured/mocked substrate holding profile** for the MVP (`K_soil`); GEBCO is not used as a mud/sand map.
5. **Copernicus Marine current vectors** from `GLOBAL_ANALYSISFORECAST_PHY_001_024`.
6. **Static ERA5 wind/wave inputs** plus interactive what-if override sliders.
7. An **IsolationForest** trajectory-anomaly model.
8. Mocked **telco criticality, redundancy, and SLA inputs**.
9. A **Configurable Policy Layer** that converts technical risk into operational actions.
10. A bounded **Agentic Incident Response Layer** that autonomously gathers relevant system evidence through allowlisted tools and prepares an incident brief/recommended NOC playbook for human review.

For the full requirements and equations, see `PRD.md`. For module boundaries and code organization, see `ARCHITECTURE.md`.

---

## 🧠 Decision Pipeline

```text
Data Ingestion / Normalization
          │
          v
Supabase PostgreSQL + PostGIS
          │
     ┌────┴────┐
     v         v
Physics     IsolationForest
R_drag      Anomaly Signal
     └────┬────┘
          v
Cable Interaction & Threat Fusion
          │
          v
Network Consequence Engine
          │
          v
Configurable Policy Layer  ← authoritative operational state
          │
   ┌──────┼───────────────┐
   v      v               v
MONITOR  PREPARE        ESCALATE
         BACKUP
           │               │
           └──────┬────────┘
                  v
       Agentic Incident Response
       (read-only tool selection +
          evidence gathering)
                  │
                  v
       Incident Brief + Playbook
                  │
                  v
          Supabase Realtime
                  │
                  v
         Next.js NOC Dashboard
     ┌────────────┼─────────────┐
     v            v             v
3D Map /      Incident      Agent Brief /
Monitoring    Workspace     Approval Controls
                  │
                  v
          Human Review / Approval
```

`R_drag` is an engineering **load/susceptibility ratio**, not a failure probability. The configurable operational policy may escalate before `R_drag = 1.0` as a precaution. The agent **does not determine or modify that state**; it investigates the event using authoritative backend evidence and recommends the next human-reviewed response. The **Next.js NOC Dashboard is the operator-facing application layer** through which users navigate, inspect evidence, review agent briefs, and approve allowed actions.

---

## 🧭 NOC UI Navigation

The MVP is a navigable NOC web application, not a single dashboard page.

| Route | What the operator sees |
| :--- | :--- |
| `/dashboard` | Main 3D Mersing map, KPI cards, current alerts, and what-if weather controls |
| `/incidents` | Filterable `MONITOR` / `PREPARE BACKUP` / `ESCALATE` incident queue |
| `/incidents/[id]` | Threat timeline, physics/anomaly evidence, cable consequence, policy state, agent brief, and approval controls |
| `/vessels` | Searchable vessel list and trajectory/context inspection |
| `/cables` | Cable corridor/segment context, criticality, and redundancy |
| `/policies` | Role-gated tenant escalation-rule editor |
| `/observer` | Restricted read-only verified `ESCALATE` view for External Maritime Observers |

Authenticated NOC pages share a persistent sidebar/topbar. The frontend displays backend-authoritative risk/policy values; it does not reimplement those calculations client-side.

---

## 🛠 Tech Stack

* **Frontend:** Next.js 14+ (React, TypeScript), Tailwind CSS, React Map GL, Deck.gl, Lucide Icons
* **Backend:** FastAPI, Python 3.12+ for Vercel deployment, GeoPandas, Shapely, scikit-learn (`IsolationForest`), `xarray` / NetCDF parsing
* **Agentic AI:** Server-side tool-calling LLM adapter + bounded incident-response orchestrator + allowlisted read-only tools + structured output/guardrails
* **Spatial Database:** Supabase PostgreSQL + PostGIS with GiST spatial indexes
* **Auth & Tenant Isolation:** Supabase Auth (JWT) + PostgreSQL Row-Level Security (RLS)
* **Realtime:** Supabase Realtime
* **Object Storage:** Supabase Storage for cached geospatial/metocean artifacts when appropriate
* **Deployment:** Vercel for the Next.js frontend; FastAPI is deployable using Vercel's supported Python/FastAPI runtime for the MVP

---

## 📂 Repository Layout

```text
SAMUDERA-App/
├── frontend/                 # Next.js navigable NOC application
│   └── src/
│       ├── app/              # dashboard, incidents, vessels, cables, policies, observer
│       ├── components/       # shell, map, risk, incidents, agent, policy, UI primitives
│       ├── lib/              # API client, Supabase browser client, typed mocks
│       └── types/
├── backend/                  # FastAPI risk/analytics service
│   └── app/
│       ├── api/
│       ├── core/
│       ├── schemas/
│       ├── services/
│       ├── agent/            # Runtime Incident Response Agent, tools, guardrails
│       └── db/
├── supabase/                 # SQL migrations, PostGIS indexes, RLS policies
├── data/                     # Local/sample data; large/private files gitignored
├── .clinerules/
│   └── 00-samudera.md        # Persistent agent guardrails
├── docs/
│   └── IMPLEMENTATION_PLAN.md # Reviewed Cline deep-planning output
├── DESIGN.md                 # Approved UI/design-system contract
├── PRD.md                    # Requirements source of truth
├── ARCHITECTURE.md           # Implementation/module boundaries
└── README.md
```

---

## 🏗 Recommended Build Order

Do **not** ask a coding agent to build the whole system in one pass.

1. **Plan and lock structure:** read `PRD.md` → `ARCHITECTURE.md` → `README.md`; create `.clinerules/00-samudera.md`; run deep planning; review and commit `docs/IMPLEMENTATION_PLAN.md`.
2. **Design and static UI shell:** finalize `DESIGN.md`, then implement the navigable Next.js routes with typed mock data.
3. **Deterministic backend:** implement and test physics, IsolationForest, threat fusion, network consequence, and policy logic.
4. **Local integration:** connect the frontend to FastAPI route-by-route.
5. **Supabase/PostGIS development integration:** review migrations first, then connect a development project/MCP and apply controlled schema/RLS changes.
6. **Real data ingestion:** integrate the approved MVP data sources without changing their documented semantics.
7. **Agentic layer:** add the bounded read-only Incident Response Agent only after deterministic outputs are stable.
8. **Realtime + approvals + deployment:** connect relevant Supabase Realtime events, verify human approval paths, test, and deploy.

At the end of every phase: run tests/build checks, inspect the diff, and create a checkpoint/commit before continuing.

---

## 🚀 How to Operate and Run Locally

### Prerequisites

* Node.js compatible with the selected Next.js 14+ version
* Python 3.12 recommended for parity with the Vercel backend deployment baseline
* A Supabase project with PostgreSQL, PostGIS, Auth, Realtime, and Storage enabled as required
* Required local/sample datasets for the MVP

### 1. Configure Environment Variables

#### Frontend — `frontend/.env.local`

```bash
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your_browser_safe_publishable_key
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
```

Only browser-safe Supabase publishable credentials may use the `NEXT_PUBLIC_` prefix.

#### Backend — `backend/.env`

```bash
SUPABASE_URL=your_supabase_url
SUPABASE_SECRET_KEY=your_server_only_secret_key

# Agentic Incident Response (server-only)
AGENT_LLM_PROVIDER=your_selected_provider
AGENT_LLM_MODEL=your_tool_capable_model
AGENT_LLM_API_KEY=your_server_only_provider_key
```

`SUPABASE_SECRET_KEY` is **server-only** and may bypass RLS. Use elevated credentials only for trusted administrative/ingestion operations. Normal user/tenant requests must preserve authenticated user context and enforce tenant authorization/RLS.

`AGENT_LLM_API_KEY` is also **server-only**. The runtime agent receives no raw infrastructure secrets; it accesses system evidence only through backend-controlled, tenant-scoped, allowlisted tools.

> Older Supabase projects may expose legacy `anon` / `service_role` keys. If used, treat `anon` as browser-safe only with correct RLS, and treat `service_role` as a server-only secret.

### 2. Apply Supabase Schema / RLS Migrations

Apply the SQL migrations under:

```text
supabase/migrations/
```

The migrations should create the required application tables, PostGIS extension/indexes, tenant-membership model, and RLS policies before the application is used.

### 3. Start the Python Backend (FastAPI)

```bash
cd backend
python -m venv .venv
```

Activate the virtual environment, then:

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

The local API will be available at:

```text
http://localhost:8000
```

### 4. Start the Frontend (Next.js)

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

The dashboard will be available at:

```text
http://localhost:3000
```

---

## 🤖 Agentic AI — Incident Response Agent

S.A.M.U.D.E.R.A. includes a bounded **tool-calling Incident Response Agent**. This is the project's agentic AI component. It is automatically triggered for `PREPARE BACKUP` / `ESCALATE` events, or can be started manually by an authorized NOC operator.

The agent does not replace the physics engine, IsolationForest, threat fusion, consequence calculation, or Policy Layer. Instead, it autonomously decides which approved read-only tools it needs to call—and in what sequence—to complete the incident-analysis goal.

Typical tools include:

* `get_threat_snapshot`
* `get_recent_trajectory`
* `get_physics_breakdown`
* `get_anomaly_result`
* `get_cable_context`
* `get_network_consequence`
* `get_tenant_policy`

The final structured output contains the authoritative risk state, supporting evidence, uncertainty/missing-data flags, and a recommended NOC playbook. For `ESCALATE`, the agent may prepare/refine the human-review dispatch draft.

**Hard safety boundary:** the agent cannot change risk scores, policy thresholds, authoritative escalation state, alert acknowledgement state, network routing, or external communications. If evidence is incomplete or contradictory, it returns `NEEDS HUMAN REVIEW`. Every run and tool call is recorded for auditability.

---

## 🌊 MVP Data Rules

To avoid accidental misrepresentation during a demo:

* **AIS:** The MVP uses shifted historical Kattegat AIS trajectories. Do not label them as live vessel telemetry.
* **GEBCO:** Use GEBCO 2026 for bathymetry/water depth only. Do not derive or display mud/sand substrate classes from GEBCO.
* **Substrate factor:** Use a clearly labeled configured/mocked `K_soil` corridor profile until a validated sediment/geotechnical source is integrated.
* **Copernicus currents:** The MVP may use a downloaded NetCDF snapshot. Automated scheduled ingestion belongs to the post-hackathon phase unless actually implemented.
* **ERA5:** The baseline is static; the UI sliders perform live **what-if recomputation**, not live ERA5 ingestion.
* **Telco topology/SLA:** MVP values are mocked unless connected to a real tenant dataset.

---

## 🧪 Core Risk Logic

### Environmental Force

```text
F_env = F_wind + F_current + F_wave
```

### Anchor-Drag Susceptibility

```text
R_drag = |F_env| / F_hold,crit
```

### Default Operational Policy

* `MONITOR`: `R_drag < 0.60` or normal transit behavior
* `PREPARE BACKUP`: `0.60 <= R_drag < 0.85` with vessel slowing inside the cable buffer
* `ESCALATE`: `R_drag >= 0.85` or an anomaly over a high-criticality cable

These are configurable **operational thresholds**. They are deliberately separate from the physical interpretation that estimated environmental load reaches estimated holding capacity at approximately `R_drag = 1.0`.

---

## ☁️ Deployment

### Frontend

Deploy the `frontend/` Next.js application to Vercel and configure:

```text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
NEXT_PUBLIC_API_BASE_URL
```

### Backend

Deploy the FastAPI service using Vercel's supported Python/FastAPI runtime for the MVP, or another compatible ASGI host if required by deployment constraints.

Configure server-side secrets in the backend deployment environment. **Never expose the Supabase secret/service-role key to the frontend.**

### Supabase

Before production/demo use:

1. Enable PostGIS.
2. Apply database migrations and GiST spatial indexes.
3. Enable and test RLS on all tenant-scoped tables.
4. Configure Supabase Auth.
5. Configure Realtime only for the tables/events needed by the dashboard.
6. Configure Storage buckets and policies for cached artifacts if used.

---

## 🔐 Security Notes

* All public application/API traffic must use HTTPS with modern TLS (TLS 1.2 or newer).
* Tenant isolation is enforced through authenticated identity **plus tenant-membership/role checks**, not by `auth.uid()` alone.
* Secret/service-role Supabase credentials are server-only.
* `ESCALATE` is a recommendation state. External dispatch remains human-reviewed and human-approved.
* Agent tools are server-side, allowlisted, tenant-scoped, and read-only for the MVP.
* The LLM is never given Supabase service-role credentials or authority to write operational state.
* Agent runs must preserve an evidence/tool-call trace so recommendations are auditable.

---

## 📊 MVP Performance Targets

These are benchmark targets, not unconditional guarantees:

* Up to 2,000 vessel vectors at a target of `>= 50 FPS` on the designated demo/reference device.
* Indexed PostGIS `ST_DWithin` proximity queries targeting `< 50 ms` at the database/query layer for the MVP dataset.
* Supabase Realtime alert-state propagation targeting `< 300 ms` under the designated demo/test environment.

---

## 🗺 MVP Corridor

The initial prototype targets the **Mersing, Johor subsea cable corridor**, including the AAG, Asia Submarine-cable Express (ASE)/Cahaya Malaysia, East-West Submarine Cable System, SEAX-1, and SKR1M systems identified for the project scope.

---

## ⚠️ Scope Boundary

S.A.M.U.D.E.R.A. is an early-warning and decision-support prototype. Its physics and holding-capacity models are calibrated risk proxies for the MVP and are not a substitute for certified maritime engineering analysis, official hydrographic products, vessel-navigation systems, or maritime-authority instructions. The Agentic Incident Response layer is an advisory orchestration component and must never be presented as autonomous maritime command authority.
