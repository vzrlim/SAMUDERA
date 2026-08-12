# SAMUDERA — DESIGN.md

> **Status:** Phase 1 frontend design contract  
> **Product:** S.A.M.U.D.E.R.A. — Spatial Awareness for Maritime Understanding, Decision Enablement & Response Assistance  
> **Scope:** Seven MVP routes only  
> **Implementation owner:** Production Next.js implementation by Cline  
> **Visual reference:** Locked Google Stitch screens  
> **Important:** Stitch HTML is a visual prototype, not production source code.

---

## 1. Purpose

This document is the production design source of truth for SAMUDERA Phase 1.

It translates the approved Stitch screens into a stable implementation contract while correcting known prototype drift. Cline should reproduce the approved visual hierarchy and interaction model without copying generated HTML literally.

Source-of-truth order when conflicts occur:

1. `PRD.md`
2. `ARCHITECTURE.md`
3. `.clinerules/00-samudera.md`
4. `docs/IMPLEMENTATION_PLAN.md`
5. This `DESIGN.md`
6. Locked Stitch screens as visual reference
7. Raw Stitch HTML only as a last-resort visual clue

The project has exactly seven MVP routes:

- `/dashboard`
- `/incidents`
- `/incidents/[id]`
- `/vessels`
- `/cables`
- `/policies`
- `/observer`

Do not create additional MVP routes for System Status, Settings, a standalone Agent, or other generated Stitch controls.

---

## 2. Phase 1 Boundary

Phase 1 is a frontend shell using typed mock data.

Phase 1 must NOT:

- connect real AIS ingestion
- claim live vessel tracking
- connect Supabase/PostGIS
- execute authoritative physics calculations in the browser
- execute policy decisions in React/TypeScript
- execute a real agent
- transmit external communications
- persist policy edits
- fabricate missing vessel, cable, coordinate, approval, or authority data

Phase 1 may show mock UI states that represent later backend behavior, but every such state must be clearly labelled as mock/simulated where necessary.

---

## 3. Product Visual Identity

SAMUDERA is a dense enterprise maritime/telco NOC product.

### 3.1 Core visual character

Internal NOC routes should feel:

- technical
- map-first where appropriate
- information-dense
- restrained
- operational
- dark navy / graphite
- cyan-led rather than neon-heavy
- red reserved for ESCALATE
- amber reserved for PREPARE / pending
- green reserved for healthy/normal status

Do not turn the interface into a military command-center aesthetic.

### 3.2 Core palette

Use semantic design tokens in production rather than page-specific copied colors.

```text
--bg:                  #10141a
--surface-lowest:      #0a0e14
--surface-low:         #181c22
--surface:             #1c2026
--surface-high:        #262a31
--surface-highest:     #31353c

--border-subtle:       #3e4850
--border-strong:       #88929b

--text-primary:        #dfe2eb
--text-secondary:      #bec8d2

--cyan-primary:        #89ceff
--cyan-strong:         #0ea5e9

--red-escalate:        #ffb4ab
--red-container:       #93000a

--amber-warning:       #f59e0b
--amber-warning-dark:  #b45309

--green-normal:        #10b981
```

Opacity variants should be derived from these semantic colors rather than added as new hardcoded colors.

### 3.3 Typography

Primary UI font:

- `Inter`

Technical/data font:

- `JetBrains Mono`

Recommended production hierarchy:

- Page title: 32px / 40px, Inter, 700
- Section heading: 20px / 28px, Inter, 600
- Body: 13–14px minimum where space allows
- Table/data: ~12px
- Labels: ~11px
- Tiny metadata: avoid below 10px unless absolutely necessary

Do not reproduce Stitch's most compressed 8–9px metadata as a default production pattern.

### 3.4 Shape and spacing

- Sidebar: **64px**
- Topbar: **56px**
- Base spacing unit: **4px**
- Default content padding: **16px**
- Standard gutter: **12px**
- Default radius: **4px**
- Larger panels: **8px**
- Avoid excessive pills and oversized rounded cards.

---

## 4. Shell Architecture

SAMUDERA uses two role-aware shell variants.

### 4.1 Internal NOC Shell

Used by:

- NOC Monitoring Operator
- NOC Policy Administrator
- authorized internal roles

Common structure:

- 64px left sidebar
- 56px topbar
- SAMUDERA radar branding
- dark operational theme
- current tenant + role visible
- route-aware active navigation
- status indicators where appropriate

Internal MVP navigation:

- Dashboard
- Incidents
- Vessels
- Cables
- Policies — only when the role is authorized

Do NOT implement generated Stitch entries:

- System Status
- Settings

Those are not MVP routes.

#### Operator identity

Operator routes should visibly show:

`DEMO TELCO • NOC OPERATOR`

#### Policy administrator identity

`/policies` should visibly show:

`DEMO TELCO • NOC POLICY ADMIN`

### 4.2 External Observer Shell

`/observer` belongs to the same SAMUDERA system but is a different restricted role experience.

Role:

`EXTERNAL MARITIME OBSERVER`

Required cues:

- dark SAMUDERA theme
- 64px restricted rail
- 56px topbar
- one permitted navigation surface only
- explicit `READ ONLY`
- explicit `EXTERNAL READ-ONLY ACCESS`
- clear statement that internal NOC controls/evidence are restricted
- no internal NOC navigation
- no policy editing
- no acknowledgement
- no dispatch approval
- no agent execution
- no internal private evidence

Observer navigation should intentionally contain:

- SAMUDERA radar logo
- selected Verified Threats icon
- non-clickable restricted/lock cue
- Observer Profile

Do not show disabled copies of the internal NOC routes.

---

## 5. Data-Honesty Contract

These rules are mandatory across all routes.

### 5.1 AIS

AIS is a shifted historical Kattegat dataset displayed over the Mersing area.

Use labels such as:

- `Historical AIS`
- `Historical AIS Replay`
- `MVP SIMULATION`

Never label it:

- `Live AIS`
- `Live Vessel Tracking`
- `Real-Time AIS`

### 5.2 Copernicus

Copernicus is a downloaded/current snapshot from:

`GLOBAL_ANALYSISFORECAST_PHY_001_024`

Do not imply a continuous live feed in Phase 1.

### 5.3 ERA5

ERA5 wind/wave is a static baseline.

Dashboard what-if controls may override wind/wave for simulation only.

Clearly separate:

- static baseline
- what-if override

### 5.4 GEBCO

GEBCO 2026 provides:

- bathymetry
- water depth

GEBCO does NOT provide:

- seabed substrate
- sediment type
- `K_soil`

### 5.5 K_soil

`K_soil` is a configured/mocked MVP holding profile.

Display where relevant:

`Configured/mocked for the MVP — not derived from GEBCO bathymetry.`

### 5.6 Telco network context

Topology, redundancy, criticality, and SLA data are mocked unless real tenant data is connected.

Do not present these as verified production telco data.

---

## 6. Physics and Policy Semantics

### 6.1 Environmental forcing

```text
F_env = F_wind + F_current + F_wave
```

### 6.2 Anchor-drag susceptibility ratio

```text
R_drag = |F_env| / F_hold,crit
```

`R_drag` is:

- a physical load / susceptibility ratio

`R_drag` is NOT:

- a probability
- a percentage
- a confidence score

Display `0.91`, not `91%`.

### 6.3 Physical interpretation bands

These describe engineering interpretation only:

```text
R_drag < 0.50
Low environmental loading

0.50 <= R_drag < 0.80
Elevated demand

0.80 <= R_drag < 1.00
Critical approach to estimated holding limit

R_drag >= 1.00
Estimated environmental load equals or exceeds holding capacity
```

### 6.4 Operational policy

Operational policy is separate from physical interpretation.

Default MVP policy:

```text
MONITOR
R_drag < 0.60
OR normal transit

PREPARE BACKUP
0.60 <= R_drag < 0.85
AND vessel slowing inside configured cable buffer

ESCALATE
R_drag >= 0.85
OR trajectory anomaly over high-criticality cable
```

ESCALATE may occur before `R_drag = 1.00`.

The Configurable Policy Layer owns the authoritative operational state.

The frontend must never calculate or override that state.

---

## 7. Network Consequence

Authoritative backend formula:

```text
Network Consequence =
R_drag * Criticality Tier * (1 / Effective Redundancy)
```

Effective Redundancy has a minimum of 1.

For `SIM-ESC-001`:

```text
R_drag = 0.91
Criticality Tier = 3
Effective Redundancy = 1

Consequence = 0.91 * 3 * (1 / 1) = 2.73
```

React/TypeScript must display backend/mock results only.

Do not reproduce the calculation as browser-owned business logic.

---

## 8. Canonical Phase 1 Fixtures

Use centralized typed fixtures. Do not scatter these literals through components.

### 8.1 Incident fixtures

| Incident | Vessel | MMSI | Cable | R_drag | GeoAI | Policy State |
|---|---|---:|---|---:|---|---|
| `SIM-ESC-001` | MT VALIANT | 533123456 | SEAX-1 | 0.91 | ANOMALY — loitering/drift | ESCALATE |
| `SIM-PREP-002` | PACIFIC M. | — | AAG | 0.62 | NORMAL — slowing in buffer | PREPARE BACKUP |
| `SIM-PREP-003` | SEA STAR | — | ASE / Cahaya Malaysia | 0.67 | NORMAL — low-speed transit/dwell in buffer | PREPARE BACKUP |
| `SIM-MON-004` | GLOBAL T. | — | SEAX-1 | 0.31 | NORMAL — normal transit | MONITOR |

### 8.2 MT VALIANT fixture

```text
Vessel: MT VALIANT
MMSI: 533123456
Vessel Type: Cargo
AIS Class: B

SOG: 0.8 kn
COG: 145.2°
Heading: 146.0°

Cable: SEAX-1
Distance to Cable: 12 m

R_drag: 0.91

F_wind: 12.4 kN
F_current: 8.2 kN
F_wave: 5.1 kN

Depth: 42 m
K_soil: 0.75

GeoAI: IsolationForest ANOMALY
Behavior: loitering / drift

Criticality Tier: 3
Effective Redundancy: 1
Network Consequence: 2.73
SLA Exposure: High / 4h Recovery

Authoritative Policy State: ESCALATE
```

### 8.3 Dashboard what-if defaults

```text
Wind Speed: 12.3 m/s
Wave Height: 1.8 m
```

What-if controls must state that they override the static baseline for simulation only.

### 8.4 Cable systems

Use only:

- AAG
- ASE / Cahaya Malaysia
- East-West
- SEAX-1
- SKR1M

Do not fabricate:

- owners
- capacity
- activation dates
- landing-station details
- health state
- repair status
- precise legal protection zones

unless a later approved source provides them.

---

## 9. Status Language

### MONITOR

Use restrained neutral/slate styling.

### PREPARE BACKUP

Use amber.

### ESCALATE

Use red.

### Normal / healthy

Use green only for genuinely normal system state.

Do not use red decoratively.

---

## 10. Route Contract — `/dashboard`

### Purpose

Primary NOC operational overview and spatial threat-monitoring surface.

### Role

NOC Monitoring Operator.

### Shell

Internal NOC shell.

Must visibly include:

`DEMO TELCO • NOC OPERATOR`

Must also preserve appropriate simulation/provenance context:

`MVP SIMULATION`

`Historical AIS • Copernicus Snapshot • Static ERA5`

### Layout

Map-first full workspace:

- left: operational queue
- center: map
- right: selected threat inspector
- bottom: what-if physics simulation

### Dashboard KPI corrections

The Stitch export contains prototype drift. Production MUST use:

```text
TRACKED: 4
THREATS: 3
PREPARE BACKUP: 2
ESCALATE: 1
REPLAY-LINKED INCIDENTS: 4
```

Do NOT use:

```text
TRACKED: 142
CABLES @ RISK: 4
```

The current fixture has four tracked vessels, and four replay-linked incidents across three cable systems.

### Threat queue

1. MT VALIANT — SEAX-1 — ESCALATE — 0.91
2. PACIFIC M. — AAG — PREPARE BACKUP — 0.62
3. SEA STAR — ASE / Cahaya Malaysia — PREPARE BACKUP — 0.67
4. GLOBAL T. — SEAX-1 — MONITOR — 0.31

### Selected inspector

For MT VALIANT show:

- vessel telemetry
- physics evidence
- GeoAI evidence
- network consequence
- authoritative policy state

Actions:

- `OPEN INCIDENT WORKSPACE`
- `REVIEW INCIDENT BRIEF`

Final response approval occurs inside the incident workspace.

### Production cleanup

Do not carry forward the generated `Simulation Step: 30s` unless it becomes a real configured simulation field.

---

## 11. Route Contract — `/incidents`

### Purpose

Operational incident queue.

### Role

NOC Monitoring Operator.

### Layout

Dense table/list with:

- summary counts
- search
- filters
- sorting
- incident rows
- action column

Summary:

```text
TOTAL: 4
ESCALATE: 1
PREPARE BACKUP: 2
MONITOR: 1
```

Required fields:

- state
- incident ID
- vessel / MMSI
- cable / corridor
- R_drag
- GeoAI analysis
- consequence
- replay age
- review status
- action

MONITOR must not imply automatic agent invocation.

---

## 12. Route Contract — `/incidents/[id]`

### Purpose

Detailed investigation and human decision workspace.

### Role

NOC Monitoring Operator.

### Layout

Three-column desktop workspace:

#### Left

- historical spatial replay
- cable-buffer context
- incident timeline

#### Center

- authoritative policy decision
- deterministic physics evidence
- deterministic GeoAI evidence
- deterministic network evidence

#### Right

- bounded agent advisory preview
- external dispatch draft
- human approve/reject controls

### Timeline

Use eight replay events:

1. historical AIS enters configured SEAX-1 cable buffer
2. low-speed / loitering behavior develops
3. IsolationForest anomaly evidence produced
4. `R_drag` reaches 0.91
5. network consequence evaluates to 2.73
6. Configurable Policy Layer produces ESCALATE
7. simulated bounded-agent investigation preview prepared
8. awaiting human operator review

### Coordinates

Do not invent numeric coordinates.

Use:

`Simulated Coordinates`

or:

`Simulated coordinates per threat snapshot`

### Dispatch draft

Must include the required contract fields when available:

- vessel MMSI
- GPS coordinates
- corridor
- threat basis
- safe-clearance / coordination request

For the Phase 1 fixture, do not invent numeric GPS coordinates.

Prefer wording:

`VESSEL: MT VALIANT (MMSI 533123456)`

rather than generic hostile-sounding target language.

State:

`NOT TRANSMITTED`

Buttons may represent the mock decision flow:

- `APPROVE DRAFT`
- `REJECT / REQUEST REVIEW`

But show:

`Phase 1 mock only — no external transmission is connected.`

---

## 13. Bounded Agent UI Contract

The agent is embedded in incident detail.

Do NOT create a standalone chatbot route.

Agent properties:

- post-policy
- read-only
- advisory
- auditable
- bounded by tenant
- cannot override deterministic outputs
- cannot modify policy
- cannot acknowledge alerts
- cannot reroute
- cannot transmit external communication

Automatic trigger:

- PREPARE BACKUP
- ESCALATE

MONITOR is not automatically invoked.

Authorized operator request may also trigger it.

Approved read-only tools:

1. `get_threat_snapshot`
2. `get_recent_trajectory`
3. `get_physics_breakdown`
4. `get_anomaly_result`
5. `get_cable_context`
6. `get_network_consequence`
7. `get_tenant_policy`

A real agent chooses the minimum required subset/order.

Do not implement a UI assumption that every future run must call all seven tools.

Phase 1 may display the existing simulated trace as one fixture run.

If evidence is stale, conflicting, or insufficient:

`NEEDS HUMAN REVIEW`

Audit display should support:

- trigger
- tools used
- evidence references
- model/provider metadata
- recommendation
- human outcome

---

## 14. Route Contract — `/vessels`

### Purpose

Historical AIS vessel explorer and selected-vessel inspector.

### Role

NOC Monitoring Operator.

### Layout

Approximate desktop split:

- 65% vessel table
- 35% selected vessel inspector

### Summary

```text
Tracked: 4
Buffer-Related: 3
GeoAI Anomalies: 1
ESCALATE: 1
```

Unknown telemetry for PACIFIC M., SEA STAR, and GLOBAL T. must remain:

`—`

Do not invent MMSIs, SOG, COG, heading, or distance.

### Required wording correction

Do NOT use:

`Target is within SEAX-1 protected corridor.`

Use:

`Vessel is within the configured SEAX-1 cable buffer.`

Do not imply hostile intent or a legal protection-zone designation.

### Map

The Stitch map is a visual placeholder only.

Production map implementation rules are in Section 19.

---

## 15. Route Contract — `/cables`

### Purpose

Subsea cable asset and network-context explorer.

It is not a live cable-health dashboard.

### Role

NOC Monitoring Operator.

### Layout

Approximate split:

- 60% cable table
- 40% cable inspector

### Required data-source label

Show:

`Source: Submarine Cable Map / TeleGeography-derived v3 endpoints`

### Required correction

The Stitch export contains one invalid label:

`Live AIS`

Production MUST replace it with:

`Historical AIS Replay`

Never ship `Live AIS`.

### Table

Use the five approved cable systems only.

For SEAX-1:

```text
Criticality Tier: 3
Effective Redundancy: 1
SLA Exposure: High / 4h Recovery
```

Other unverified network fields should be `—`.

### Incident context

SEAX-1 has two replay-linked incidents in the fixture:

- SIM-ESC-001
- SIM-MON-004

AAG:

- SIM-PREP-002

ASE / Cahaya Malaysia:

- SIM-PREP-003

East-West:

- none

SKR1M:

- none

### Mock-data note

Show:

`MVP telco topology / SLA context is mocked unless real tenant data is connected.`

### Do not implement unsupported generated controls

The Stitch export includes generated affordances such as `Export Data`.

Do not implement them in Phase 1 unless separately required by the product contract.

---

## 16. Route Contract — `/policies`

### Purpose

Tenant operational-policy configuration view.

### Role

NOC Policy Administrator.

### Shell

Internal NOC shell, but role must be:

`DEMO TELCO • NOC POLICY ADMIN`

The long telemetry-provenance sentence is not required in this route's topbar.

### Metadata

Show:

```text
Tenant: DEMO TELCO
Corridor: Mersing Subsea Corridor
Policy: Default MVP Operational Policy
Configuration Status: Phase 1 Mock — Not Persisted
Authority: Configurable Policy Layer
```

### Main sections

- operational policy rule set
- physical R_drag interpretation
- architecture/guardrails
- mock editing status

### Phase 1 interaction

Policy persistence is not connected.

Any mock edit affordance must not imply that changes are saved to a backend.

`SAVE MOCK DRAFT` should remain disabled or clearly non-persistent.

### Required guardrails

Display:

- authoritative evaluation remains server-side
- frontend does not calculate incident state
- physical interpretation and operational policy remain separate
- agent cannot override policy state
- human approval required before external communication

---

## 17. Route Contract — `/observer`

### Purpose

Restricted read-only external observer projection.

### Role

External Maritime Observer.

### Shell

Use the dedicated Observer shell.

Header identity:

`SAMUDERA EXTERNAL OBSERVER PORTAL`

Role:

`EXTERNAL MARITIME OBSERVER • READ ONLY`

Access strip:

`EXTERNAL READ-ONLY ACCESS`

Supporting text:

`Verified ESCALATE context only • Internal NOC controls and evidence are restricted`

### Layout

Approximate split:

- 58% verified incident list
- 42% selected threat detail

### Incident visibility

Observer sees verified ESCALATE only.

Current Phase 1 fixture:

```text
SIM-ESC-001
MT VALIANT
MMSI 533123456
SEAX-1
Mersing Subsea Corridor
ESCALATE
```

Do not show PREPARE BACKUP or MONITOR incidents.

### Threat detail

Use high-level external context only.

Do not expose:

- raw force components
- K_soil
- water depth
- IsolationForest score
- full network consequence details
- SLA exposure
- tenant policy configuration
- internal notes
- agent tool trace
- model/provider metadata

### Dispatch visibility

Current Phase 1 state:

`AWAITING HUMAN APPROVAL`

`NOT TRANSMITTED`

Message:

`External dispatch information is withheld until an authorized NOC operator approves the communication.`

No:

- approve
- reject
- send
- transmit
- acknowledge
- reroute
- run-agent controls

### Coordinates

Do not invent coordinates.

Show:

`Simulated Coordinates`

with:

`Available from approved dispatch context when authorized.`

### Observer warning tokens

Use:

```text
warning: #f59e0b
warning-dark: #b45309
```

for the existing amber pending/approval-waiting treatment.

---

## 18. Role and Access Matrix

| Capability | NOC Monitoring Operator | NOC Policy Admin | External Maritime Observer |
|---|---:|---:|---:|
| Dashboard | Yes | As authorized | No |
| Incident queue | Yes | As authorized | No |
| Incident workspace | Yes | As authorized | No |
| Vessels | Yes | As authorized | No |
| Cables | Yes | As authorized | No |
| Policies | No by default | Yes | No |
| Observer projection | No need | No need | Yes |
| Acknowledge/review NOC incident | Yes | No by default | No |
| Approve/reject dispatch draft | Yes | No by default | No |
| Modify policy | No | Yes, later backend phase | No |
| Run/request bounded agent | Authorized operator flow | As authorized | No |
| See internal deterministic evidence | Yes | As authorized | No |
| See approved external dispatch context | Internal | Internal | Yes, after approval |

Production auth/RBAC must ultimately be enforced server-side and with Supabase RLS when that phase is reached.

---

## 19. Map Implementation Contract

Do NOT copy Stitch's handcrafted maps into production.

Do NOT copy:

- hardcoded SVG land masses
- arbitrary pixel-positioned vessel markers
- public placeholder background images
- fake fixed cable paths
- map-like decorative divs as authoritative spatial data

Production stack:

- React Map GL
- Deck.gl

Map layers should be fed from typed data structures and replaceable data-access functions.

Expected layer families:

- historical AIS positions / trajectories
- vessel markers
- cable paths
- configured cable buffers
- incident markers
- GEBCO bathymetry context where appropriate

Backend spatial authority in later phases:

- PostGIS
- `ST_DWithin`
- `ST_Intersects`
- GiST indexes

GeoPandas/Shapely are ETL/local-processing tools only, not production spatial-query authority.

---

## 20. Component Architecture

Cline should create reusable components rather than route-specific copies.

Suggested structure:

```text
components/
  shell/
    NocAppShell
    ObserverAppShell
    SideNav
    TopBar
    RoleBadge
    SimulationBadge

  status/
    PolicyStateBadge
    GeoAIStatusBadge
    PlatformStatus
    ApprovalStatusBadge

  incidents/
    IncidentTable
    IncidentRow
    IncidentSummary
    IncidentTimeline
    PolicyDecisionPanel
    PhysicsEvidencePanel
    GeoAIEvidencePanel
    NetworkEvidencePanel
    AgentAdvisoryPanel
    DispatchDraftPanel

  vessels/
    VesselTable
    VesselInspector
    VesselTrajectoryPreview

  cables/
    CableTable
    CableInspector
    CableIncidentContext

  policies/
    PolicyRuleTable
    PhysicalBandPanel
    PolicyGuardrails

  observer/
    ObserverIncidentList
    ObserverThreatDetail
    ObserverDispatchContext

  map/
    MaritimeMap
    VesselLayer
    TrajectoryLayer
    CableLayer
    CableBufferLayer
    IncidentLayer
```

Exact folder naming can vary, but duplication should remain low.

---

## 21. Typed Domain Objects

Use centralized domain types.

Example direction:

```ts
type PolicyState = "MONITOR" | "PREPARE_BACKUP" | "ESCALATE";

interface Incident {
  id: string;
  vesselName: string;
  mmsi: string | null;
  cableName: string | null;
  corridorName: string;
  rDrag: number | null;
  geoAiStatus: "NORMAL" | "ANOMALY" | null;
  behaviorSummary: string | null;
  consequenceScore: number | null;
  policyState: PolicyState;
}

interface Vessel {
  name: string;
  mmsi: string | null;
  vesselType: string | null;
  aisClass: string | null;
  sog: number | null;
  cog: number | null;
  heading: number | null;
  cableName: string | null;
  distanceToCableM: number | null;
  rDrag: number | null;
  geoAiStatus: "NORMAL" | "ANOMALY" | null;
  policyState: PolicyState | null;
  incidentId: string | null;
}

interface Cable {
  name: string;
  source: string;
  criticalityTier: number | null;
  effectiveRedundancy: number | null;
  slaExposure: string | null;
  linkedIncidentIds: string[];
}

interface ObserverIncident {
  id: string;
  vesselName: string;
  mmsi: string;
  cableName: string;
  corridorName: string;
  authoritativeState: "ESCALATE";
  behaviorSummary: string;
  threatSummary: string;
}

interface ObserverDispatchContext {
  status: "AWAITING_HUMAN_APPROVAL" | "APPROVED";
  approved: boolean;
  vesselMmsi: string | null;
  coordinates: string | null;
  corridorIdentifier: string | null;
  threatBasis: string | null;
  clearanceRequest: string | null;
}
```

Nullable fields are required.

Never invent a value just because a component expects one.

---

## 22. Data Access Boundary

Pages/components should not import raw fixture literals directly.

Use replaceable functions such as:

```text
getDashboardSnapshot()
getIncidents()
getIncidentById(id)
getVessels()
getVesselByMmsi(mmsi)
getCables()
getCableByName(name)
getPolicyConfig()
getObserverIncidents()
getObserverDispatchContext(id)
```

Phase 1 implementations return typed fixtures.

Later phases replace those functions with FastAPI calls.

This keeps visual components independent from the data source.

---

## 23. No Browser-Side Authority

React/TypeScript may:

- format values
- filter/sort current fixture rows
- select rows
- open/close inspectors
- manage visual form state
- render what-if input controls

React/TypeScript must NOT authoritatively compute:

- R_drag
- environmental force
- policy state
- network consequence
- anomaly classification
- agent recommendation
- dispatch approval

Those outputs belong to deterministic backend / policy / model layers in later phases.

---

## 24. Production Overrides From Final Audit

These overrides are mandatory even if the Stitch HTML differs.

### Dashboard

Replace:

```text
TRACKED 142
```

with:

```text
TRACKED 4
```

Replace:

```text
CABLES @ RISK 4
```

with:

```text
REPLAY-LINKED INCIDENTS 4
```

Ensure visible identity:

`DEMO TELCO • NOC OPERATOR`

Omit generated `Simulation Step: 30s` unless formally configured.

### Vessels

Replace:

`Target is within SEAX-1 protected corridor.`

with:

`Vessel is within the configured SEAX-1 cable buffer.`

### Cables

Replace:

`Live AIS`

with:

`Historical AIS Replay`

Normalize the topbar to 56px.

### Internal shared shell

Remove generated non-MVP navigation:

- System Status
- Settings

### Observer

Use the latest corrected dark Observer screen.

Keep the fixed amber tokens:

```text
warning: #f59e0b
warning-dark: #b45309
```

Do not resurrect the abandoned light-mode version.

---

## 25. What NOT to Copy From Stitch

Do not copy Stitch output verbatim.

Specifically avoid:

- duplicated Google Fonts imports
- duplicate Material Symbols imports
- per-screen Tailwind configs
- random generated CSS utility drift
- hardcoded `href="#"` navigation
- remote placeholder profile images
- remote placeholder map images
- handcrafted fake spatial geometry
- page-specific repeated shell markup
- duplicate height classes
- unsupported buttons/routes
- literal fixture values scattered throughout JSX
- browser-owned policy/physics calculations

The production implementation should preserve the visual result while cleaning the structure.

---

## 26. Phase 1 Acceptance Checklist

Phase 1 frontend is complete only when:

- [ ] all seven MVP routes render
- [ ] shared internal NOC shell is reusable
- [ ] Observer uses the restricted Observer shell
- [ ] topbar is 56px
- [ ] sidebar is 64px
- [ ] internal role/tenant identity is visible
- [ ] Policy route shows NOC POLICY ADMIN
- [ ] Observer shows EXTERNAL MARITIME OBSERVER • READ ONLY
- [ ] Dashboard uses tracked count 4
- [ ] Dashboard does not show `CABLES @ RISK 4`
- [ ] Cables does not contain `Live AIS`
- [ ] Vessels does not use `protected corridor`
- [ ] no System Status route
- [ ] no Settings route
- [ ] no standalone Agent route
- [ ] all core data comes from typed centralized fixtures
- [ ] missing values render as `—`
- [ ] no fabricated MMSIs or coordinates
- [ ] historical/static provenance is honest
- [ ] R_drag is never displayed as a probability/percentage
- [ ] physical interpretation and policy remain visually separate
- [ ] Configurable Policy Layer is shown as state authority
- [ ] IsolationForest is evidence, not policy authority
- [ ] agent is post-policy, read-only, advisory
- [ ] MONITOR does not auto-trigger the agent
- [ ] dispatch remains NOT TRANSMITTED until human approval
- [ ] Observer never exposes unapproved dispatch contents
- [ ] maps are production-ready component shells, not copied Stitch SVG art
- [ ] no Supabase/backend/live ingestion is added in Phase 1
- [ ] no authoritative browser-side calculations are added

---

## 27. Phase 1 Handoff Rule

Once this document is committed, Cline should implement **Phase 1 only**.

Do not allow Cline to jump ahead into:

- FastAPI deterministic backend
- Supabase/PostGIS
- real ingestion
- real agent execution
- realtime subscriptions
- deployment hardening

Those belong to later implementation phases.

The goal of Phase 1 is a clean, typed, reusable frontend implementation of the seven approved SAMUDERA screens with honest mock data and correct role boundaries.
