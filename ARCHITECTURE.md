# System Architecture & Project Framework

## 📂 Project Directory Structure
*(Note: This is the enforced monorepo structure for the Antigravity agent to follow for clean separation of concerns).*

\`\`\`text
SAMUDERA-App/
│
├── frontend/                  # Next.js 14 App Router
│   ├── src/
│   │   ├── app/               # UI Pages & Routing
│   │   ├── components/        # Deck.gl Maps, Sliders, Alert Modals
│   │   └── lib/               # Supabase Client & API Fetchers
│   ├── package.json
│   └── tailwind.config.js
│
├── backend/                   # Python FastAPI Server
│   ├── main.py                # API Entrypoint
│   ├── physics_engine.py      # Wind/Wave/Current Vector Math
│   ├── ml_anomaly.py          # scikit-learn Isolation Forest logic
│   ├── spatial_router.py      # GeoPandas R-Tree Proximity checks
│   └── requirements.txt
│
└── vercel.json                # Cloud routing configuration
\`\`\`

## 🧩 Core Modules Explanation

*   **Module A: Geospatial Data Ingestion:** Dynamically fetches subsea cables from TeleGeography, ingests GEBCO bathymetric grids, and converts Copernicus NetCDF current velocities into JSON vector grids using Python `xarray`[cite: 6]. Executes $O(1)$ spatial bounding-box checks using PostGIS or GeoPandas[cite: 6].
*   **Module B: Dual-Engine Risk Modeling:** Calculates total environmental drag force vectors ($\vec{F}_{\text{env}}$) against holding capacity ($F_{\text{hold,crit}}$)[cite: 6]. Runs an `IsolationForest` model on AIS trajectory features (Speed, Course, Dwell time) to flag abnormal loitering[cite: 6].
*   **Module C: Network Consequence & Policy Layer:** Calculates SLA business risk using: $R_{\text{drag}} \cdot \text{Criticality Tier} \cdot (1 / \text{Backup Redundancy})$[cite: 6]. Evaluates scores against a modular rules engine to output `MONITOR`, `PREPARE BACKUP`, or `ESCALATE` statuses[cite: 6].
*   **Module D: NOC Command Dashboard:** GPU-accelerated 3D Mapbox/Deck.gl frontend ($\ge 50 \text{ FPS}$) providing explainable physics inspection and automated VHF radio dispatch generation[cite: 6].

## 🗄️ Data Model Design (Minimum Dictionary)

| Category | Variable | Source | Purpose |
| :--- | :--- | :--- | :--- |
| **Vessel Telemetry** | Lat, Lon, SOG, COG, Heading, MMSI | Shifted Kattegat AIS CSV | Tracks dynamic vessel motion & position[cite: 6]. |
| **Cable Infrastructure**| Line Coordinates, Landing Nodes | TeleGeography Live API | Maps spatial location of subsea assets[cite: 6]. |
| **Seabed Profiling** | Water Depth ($D_{\text{water}}$), Substrate ($K_{\text{soil}}$)| GEBCO 2026 Grid | Determines bathymetry and friction capacity[cite: 6]. |
| **Metocean Currents** | $U$-Velocity, $V$-Velocity | Copernicus (`PHY_001_024`) | Supplies live ocean current vectors[cite: 6]. |
| **Metocean Weather** | Wind Speed ($V_{\text{wind}}$), Wave Height ($H_s$) | ERA5 + UI Sliders | Provides wind and wave force inputs[cite: 6]. |
| **Network Topology** | Segment ID, Criticality Tier, Redundancy | Mocked Telco Data | Calculates business impact & SLA risk[cite: 6]. |

## 🛡️ Coding Requirements & Restrictions (NFRs)

1.  **Security & Multi-Tenant Isolation:**
    *   All API endpoints MUST enforce TLS 1.3 in transit[cite: 6].
    *   Database storage MUST enforce AES-256 encryption at rest[cite: 6].
    *   **Row-Level Security (RLS):** Supabase RLS policies MUST restrict database table access to `auth.uid()`. Tenant A cannot access Tenant B's internal network topology[cite: 6].
2.  **Performance:**
    *   Frontend mapping engine MUST render up to 2,000 simultaneous vessel vectors at $\ge 50 \text{ FPS}$[cite: 6].
    *   Spatial proximity checks (`ST_DWithin`) MUST resolve in $< 50 \text{ ms}$[cite: 6].
3.  **Real-Time Sync:**
    *   Multi-user state updates (e.g., alert acknowledgment) MUST sync across connected sessions via Supabase Realtime within $< 300 \text{ ms}$[cite: 6].