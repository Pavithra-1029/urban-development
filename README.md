# VaaniConnect – AI-Assisted Rural Grievance Intelligence System

VaaniConnect is an intelligent, full-stack civic technology platform engineered for rural public grievance management. It solves the widespread problem of complaint duplication in rural civic administration by consolidating multiple citizen complaints about the same physical issue into a single **Master Issue**, dynamically prioritizing real-world urgency, and coordinating departmental resolution.

---

## 1. Project Overview & Architecture

### System Architecture
The application is structured into a clean decoupled full-stack architecture:
- **Frontend (`frontend/`):** React 18, Vite, Tailwind CSS, Lucide icons, and Leaflet with OpenStreetMap for interactive geospatial visualization.
- **Backend (`backend/`):** Python FastAPI REST framework with asynchronous request handling and Pydantic schema validation.
- **Persistence Layer (`backend/database.py` & `backend/models.py`):** SQLite with SQLAlchemy ORM, fully normalized into 9 relational entities.
- **AI/ML & Analytical Engines (`backend/`):**
  - `nlp_engine.py`: Category classification, linguistic keyword extraction, severity estimation, and text similarity (TF-IDF + cosine similarity + token overlap).
  - `geo_engine.py`: Geospatial distance calculation using the Haversine formula and radius proximity matching.
  - `duplicate_engine.py`: Multi-factor composite confidence calculation ($\ge 85\%$ auto-merge, $60\% - 84\%$ review queue, $< 60\%$ independent ticket).
  - `priority_engine.py`: Dynamic multi-factor prioritization scoring with transparent human-readable explanations.
  - `analytics_engine.py`: Recurrence detection against historical resolved tickets and proximity-based cross-category correlation.
  - `recommendation_engine.py`: Actionable departmental dispatch suggestions.

---

## 2. Technology Stack

- **Frontend:** React.js, Tailwind CSS, Vite, Leaflet, React-Leaflet, Lucide React, HTML5 Web Speech API
- **Backend:** Python 3.12, FastAPI, Uvicorn, SQLAlchemy, Pydantic
- **AI/ML & Math:** Scikit-learn (TfidfVectorizer, cosine_similarity), Haversine formula implementation
- **Database:** SQLite with SQLAlchemy ORM
- **Testing:** Pytest, Fastapi TestClient, Playwright

---

## 3. Implemented Modules

1. **Citizen Complaint Registration:** Voice-to-text input (Web Speech API) with manual typing fallback; GPS coordinate location picker with manual village/landmark entry; category selection; image upload simulation.
2. **AI Complaint Understanding:** Offline NLP classifier extracting categories, keywords, and severity weights without external paid APIs.
3. **Geospatial Location Management:** Accurate latitude/longitude storage, Haversine formula calculations, and interactive OpenStreetMap markers.
4. **Intelligent Duplicate Detection:** Multi-factor confidence scoring combining geospatial distance ($40\%$), TF-IDF text similarity ($35\%$), category match ($15\%$), and village/landmark match ($10\%$).
5. **Master Issue Engine:** Consolidates multiple reports under an identifier (e.g. `MI-001`), aggregates supporting citizen reports, maintains full report history, and dynamically updates report counts.
6. **Dynamic Smart Priority Engine:** Transparent priority scoring (Severity $35\%$, Reports $25\%$, Age $15\%$, Safety $15\%$, Recurrence $10\%$) categorized into *Critical, High, Medium, Low* with full algorithmic rationale.
7. **Complaint Evolution Tracking:** Complete timestamped audit history tracking severity escalations, status transitions, and supporting complaint attachments.
8. **Recurring Problem Detection:** Identifies repeated infrastructure failures at the same location to recommend permanent structural inspections.
9. **Related Issue Detection:** Discovers correlated issues in the same vicinity (e.g., water leakage eroding road asphalt).
10. **AI-Assisted Action Recommendation:** Recommends the responsible department (PWD, Water Board, Electricity Board, Sanitation), inspection requirements, and emergency interventions.
11. **Admin Dashboard:** Real-time KPI counters, interactive Leaflet map, Master Issue filters, Master Issue detailed drawer, category/village charts, and an AI Duplicate Review Queue.
12. **Field Worker Dashboard:** Dedicated interface displaying assigned tasks, location details, status transitions (*In Progress*, *Resolved*), and resolution notes.
13. **Citizen Resolution Verification:** Citizen feedback loop allowing verification or dispute of resolved issues, auto-flagging disputed tickets for administrator review.

---

## 4. Demo Login & Roles

The system provides an instant role switcher in the top navigation bar for testing and demonstration:

| Role | Demo Identity | Description & Access |
|---|---|---|
| **Citizen** | Ramesh Kumar (`citizen_demo`) | File complaints, use voice input, verify/dispute resolutions |
| **Administrator** | Officer Rajesh Sharma (`admin_demo`) | Monitor KPIs, review duplicates, assign departments, inspect evolution |
| **Field Worker** | Suresh Verma - Lineman / PWD (`worker_demo`) | View assigned tasks, mark In Progress/Resolved, enter field repair notes |

---

## 5. Setup & Execution Instructions

### Prerequisites
- Python 3.10+
- Node.js 18+ and npm

### Backend Setup
```bash
# 1. Install Python dependencies
pip install -r requirements.txt

# 2. Run backend test suite
PYTHONPATH=. pytest -v

# 3. Start backend API server (runs on port 8000)
uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

### Frontend Setup
```bash
# 1. Navigate to frontend directory and install dependencies
cd frontend
npm install

# 2. Start the development server (runs on port 5173)
npm run dev
```

The application will be accessible at `http://localhost:5173` with backend API documentation available at `http://localhost:8000/docs`.

---

## 6. AI/ML Algorithms & Formulas

### A. Geospatial Distance: Haversine Formula
Given coordinates $(\phi_1, \lambda_1)$ and $(\phi_2, \lambda_2)$ in radians and Earth radius $R = 6,371,000\text{ m}$:
$$a = \sin^2\left(\frac{\Delta \phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta \lambda}{2}\right)$$
$$c = 2 \cdot \text{atan2}\left(\sqrt{a}, \sqrt{1 - a}\right)$$
$$\text{Distance} = R \cdot c$$

### B. Text Similarity: TF-IDF & Cosine Similarity
1. Vectorize complaint texts using Scikit-Learn `TfidfVectorizer` (unigrams and bigrams, English stopword elimination).
2. Compute cosine angle between text vectors:
$$\text{Cosine Similarity} = \frac{\mathbf{u} \cdot \mathbf{v}}{\|\mathbf{u}\|_2 \|\mathbf{v}\|_2}$$
3. Supplemented with token-level Dice coefficient and domain synonym expansion (e.g. *pothole* $\leftrightarrow$ *crater* $\leftrightarrow$ *caved road*).

### C. Multi-Factor Duplicate Confidence Score
$$\text{Confidence} = 0.40 \cdot S_{\text{geo}} + 0.35 \cdot S_{\text{text}} + 0.15 \cdot S_{\text{category}} + 0.10 \cdot S_{\text{landmark}}$$
- **Geospatial Score ($S_{\text{geo}}$):** Linear decay within 300 meters ($S_{\text{geo}} = \max(0, 1 - \frac{d}{300})$). If distance exceeds 500 meters, geospatial score is 0.
- **Threshold Actions:**
  - $\ge 85\%$: **Likely Duplicate** $\rightarrow$ Automatically merged into active Master Issue.
  - $60\% - 84\%$: **Borderline Match** $\rightarrow$ Sent to Administrator Review Queue.
  - $< 60\%$: **Independent Issue** $\rightarrow$ New Master Issue ticket created.

### D. Dynamic Smart Priority Score
$$\text{Priority Score} = 0.35 \cdot S_{\text{severity}} + 0.25 \cdot S_{\text{reports}} + 0.15 \cdot S_{\text{aging}} + 0.15 \cdot S_{\text{safety}} + 0.10 \cdot S_{\text{recurrence}}$$
- **Severity ($S_{\text{severity}}$):** Critical = 100, High = 75, Medium = 50, Low = 25.
- **Report Count ($S_{\text{reports}}$):** $\min(100, (\text{reports} - 1) \cdot 15)$.
- **Aging ($S_{\text{aging}}$):** $\min(100, \text{days\_unresolved} \cdot 10)$.
- **Safety ($S_{\text{safety}}$):** 100 if hazardous keywords detected (live wire, flood, deep crater, collapse).
- **Recurrence ($S_{\text{recurrence}}$):** 100 if prior resolved incidents occurred at coordinates.
- **Bands:** Critical ($\ge 80$), High ($60 - 79$), Medium ($40 - 59$), Low ($< 40$). Safety override guarantees at least High priority for dangerous hazards.

---

## 7. Key REST API Endpoints

- `GET /api/roles` - Retrieve demo users and credentials.
- `GET /api/dashboard/stats` - Summary KPI metrics and analytics distributions.
- `GET /api/master-issues` - Filterable list of all Master Issues.
- `GET /api/master-issues/{id}` - Full details including supporting reports, history, related issues, and feedback.
- `POST /api/complaints` - Register a new citizen complaint and trigger duplicate evaluation.
- `POST /api/master-issues/{id}/assign` - Assign responsible department.
- `POST /api/master-issues/{id}/status` - Update resolution status and add field notes.
- `GET /api/duplicate-reviews` - Retrieve complaints awaiting duplicate review triage.
- `POST /api/duplicate-reviews/{id}/action` - Administrator triage decision (`merge` or `separate`).
- `POST /api/master-issues/{id}/feedback` - Citizen resolution verification feedback (confirm/dispute).
- `POST /api/demo/reset` - Reset database to pre-populated demo seed data.

---

## 8. Verification & Testing Results

- **Automated Tests:** 15 out of 15 tests passing (`PYTHONPATH=. pytest`):
  - NLP keyword and severity extraction tests.
  - Haversine distance tests.
  - TF-IDF similarity and duplicate threshold tests.
  - Priority formula and recommendation engine tests.
  - API endpoint integration tests.
  - End-to-End Scenario Test (`tests/test_e2e_scenario.py`):
    1. Citizen A reports pothole at Rampur primary school.
    2. Citizen B reports same pothole with different wording.
    3. Engine calculates composite confidence $\ge 85\%$ and auto-merges complaint to Master Issue `MI-001`.
    4. Supporting reports count increases to 2; priority engine recalculates urgency.
    5. Administrator views ticket on map and assigns to Public Works Department.
    6. Field worker transitions status to `Resolved` with completion notes.
    7. Citizen provides resolution verification feedback.
    8. Separate complaint from distant village correctly spawns independent Master Issue.
- **Visual UI Verification:** Playwright scripts verified interactive Leaflet maps, live Web Speech API integration, Master Issue drawer, review queue, and field worker forms.

---

## 9. Known Limitations & Future Enhancements

- **Speech Recognition:** Web Speech API depends on modern browser support (Chrome, Edge, Safari); manual text entry functions seamlessly on unsupported environments.
- **Multilingual Support:** Future versions could add Indic language translation (Hindi, Tamil, Telugu, etc.) using offline IndicBERT models.
- **Image Analysis:** Future iterations can incorporate a lightweight computer vision model (YOLOv8-nano) to verify pothole depth and garbage volume from uploaded photographs.
- **SMS & IVR Integration:** Integration with open-source telephony gateways (e.g., Asterisk) for feature-phone voice reporting via automated toll-free IVR.
