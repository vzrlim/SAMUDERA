# S.A.M.U.D.E.R.A. 
**Spatial Analytics for Maritime Utilities & Drag Early-warning Risk Automation**

## 📖 Project Brief
Submarine telecommunications cables carry over 99% of international data traffic, yet dragged ship anchors account for roughly 30% of all subsea faults, costing $1.5M+ USD per incident in repair mobilization[cite: 6]. 

S.A.M.U.D.E.R.A. is an enterprise spatial intelligence and decision-support platform designed for Telecommunication Network Operations Centers (NOCs)[cite: 6]. It acts as a Translation and Decision Layer[cite: 6]. By fusing seafloor geophysics, hydrodynamic drag vectors, and vessel trajectory telemetry, the system quantifies business impact and runs risk scores through a Configurable Policy Layer[cite: 6]. It delivers proactive, human-in-the-loop escalation recommendations (`MONITOR`, `PREPARE BACKUP`, `ESCALATE`) before physical strikes occur[cite: 6].

## 🛠 Tech Stack
* **Frontend:** Next.js 14+ (React, TypeScript), Tailwind CSS, React Map GL / Deck.gl[cite: 6]
* **Backend:** Python 3.11+, FastAPI, GeoPandas, scikit-learn (Isolation Forest), Xarray[cite: 6]
* **Database & Auth:** Supabase (PostgreSQL, PostGIS Extension, JWT Auth, Row-Level Security)[cite: 6]
* **Deployment:** Vercel (Frontend & Serverless Functions)[cite: 6]

## 🚀 How to Operate and Run Locally

*(Note: The following local dev commands are standard requirements for the Next.js/FastAPI stack).*

### Prerequisites
* Node.js (v18+)
* Python (v3.11+)
* Supabase Account & Database URL

### 1. Start the Python Backend (FastAPI)
Navigate to the backend folder, install dependencies, and run the server:
\`\`\`bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
\`\`\`
*The API will be available at http://localhost:8000*

### 2. Start the Frontend (Next.js)
Open a new terminal, navigate to the frontend folder, install packages, and start the dev server:
\`\`\`bash
cd frontend
npm install
npm run dev
\`\`\`
*The dashboard will be available at http://localhost:3000*

## ☁️ Deployment (Vercel)
This project is configured for serverless deployment on Vercel[cite: 6]. 
1. Connect this repository to Vercel.
2. Vercel will automatically detect the Next.js frontend framework.
3. Add your `NEXT_PUBLIC_SUPABASE_URL` and `SUPABASE_SERVICE_KEY` to the Vercel Environment Variables.
4. Deploy. (If using Vercel to host the Python backend, ensure `vercel.json` is configured to route `/api` calls to the FastAPI app).