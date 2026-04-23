# CrimeLens – Forensic Homicide Investigation Suite

## Project Overview
CrimeLens is a premium, glass‑morphic web‑application designed for **real‑time forensic investigation of homicide cases**.  It provides investigators with a modern UI, a powerful backend, and a seamless workflow for:
- **Creating new investigations** with multiple pieces of forensic evidence (photos, descriptions).
- **Storing and visualising evidence** on a map that automatically re‑centers to the investigator’s location.
- **Analyzing evidence** via an AI‑driven pipeline that produces forensic reports and threat assessments.
- **Managing cases** (status, priority, notes) and visual dashboards that highlight active homicide zones.

The UI follows a **glass‑morphism** design language: subtle translucency, vibrant accent colours, animated interactions, and responsive layouts that feel like a command‑center for forensic analysts.

---

## Technology Stack
| Layer | Tech | Reason |
|-------|------|--------|
| Front‑end | **React** (Vite), **Lucide‑React** icons, **CSS** (custom design system) | Fast SPA, modern component model, easy theming |
| Styling | Vanilla CSS + CSS variables (threat levels, glass borders) | Full control over the premium UI without heavyweight frameworks |
| Backend | **Node.js** + **Express** | Simple REST API, easy integration with MongoDB |
| Database | **MongoDB** (Mongoose schema) | Flexible document model for cases, analyses, users |
| File Upload | **Multer** (50 MB limit) | Handles forensic image uploads and stores them in `uploads/` |
| Authentication | **JWT** (Bearer token) | Stateless, role‑based access control |
| AI / Vision (placeholder) | **analysisService** (mock) | In a real deployment you would plug‑in YOLO / Gemini Vision for forensic analysis |

---

## Repository Structure
```
CrimeLens/
├─ client/                    # React front‑end
│  ├─ src/
│  │  ├─ components/          # UI widgets (CaseCard, StatusDropdown, NewCaseModal, …)
│  │  ├─ pages/               # Pages (CasesPage, CaseDetailPage, DashboardPage, CrimeMapPage)
│  │  ├─ services/            # API wrapper (axios instance & caseService, analysisService, …)
│  │  └─ index.css            # Global design system (CSS variables, glass‑morphism utils)
│  └─ vite.config.js
├─ server/                    # Express back‑end
│  ├─ src/
│  │  ├─ controllers/        # Business logic (caseController, authController, …)
│  │  ├─ models/             # Mongoose schemas (User, Case, Analysis)
│  │  ├─ routes/             # API routes (caseRoutes, analysisRoutes, authRoutes)
│  │  ├─ middleware/         # auth middleware (jwt verification)
│  │  ├─ services/           # (future) AI vision services
│  │  └─ server.js            # Express bootstrap, static /uploads serving
│  └─ uploads/                # Persisted forensic images (auto‑created on start)
└─ README.md                  # ***This file***
```

---

## Getting Started
### Prerequisites
- **Node.js ≥ 18**
- **npm** (or **yarn**)  
- **MongoDB** (local or remote URI)

### Installation
```bash
# Clone the repository
git clone https://github.com/yourusername/CrimeLens.git
cd CrimeLens

# Install dependencies for both client and server
npm install   # installs root level deps (none)
cd client && npm install && cd ..
cd server && npm install && cd ..
```
### Environment variables
Create a `.env` in `server/`:
```
MONGODB_URI=mongodb://localhost:27017/crimelens
JWT_SECRET=your_secret_key
PORT=5000
```
### Running the app (development)
```bash
# In one terminal – start the backend
cd server
npm run dev   # nodemon watches server files

# In another terminal – start the frontend
cd client
npm run dev   # Vite dev server (http://localhost:5173)
```
Open the URL, log in (or register), and you’ll land on the **Dashboard**.

---

## API Overview
All endpoints are prefixed with `/api`.

### Authentication (`/api/auth`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/login` | Returns JWT token on valid credentials |
| `POST` | `/register` | Creates a new user and returns token |
| `GET`  | `/me` | Returns current user details (requires Bearer token) |

### Cases (`/api/cases`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET`  | `/` | List cases (optional `status`, `priority`, `search` query) |
| `POST` | `/` | **Create a new investigation** – multipart/form‑data. Accepts multiple `images` and `evidenceDescriptions` (same order). |
| `GET`  | `/:id` | Retrieve a case with populated analyses |
| `PUT`  | `/:id` | Update case fields (optional `image` upload) |
| `DELETE`| `/:id`| Delete a case |
| `POST` | `/:id/evidence` | Link an existing analysis to the case |
| `POST` | `/:id/notes` | Add a forensic note to the case |

### Analyses (`/api/analysis`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/analyze` | Upload a single image for AI analysis (multipart) |
| `GET`  | `/` | List all analyses |
| `GET`  | `/:id` | Get a specific analysis |
| `GET`  | `/stats` | Summary statistics (e.g., threat distribution) |
| `GET`  | `/patterns` | Detect clusters of homicide hot‑spots |

---

## Data Models (Mongoose)
### User
```js
{
  name: String,
  email: { type: String, unique: true },
  password: String, // hashed via bcrypt
  role: { type: String, enum: ['investigator','admin'], default: 'investigator' }
}
```
### Case
```js
{
  title: { type: String, required: true },
  description: String,
  status: { type: String, enum: ['open','investigating','closed'], default: 'open' },
  priority: { type: String, enum: ['critical','high','medium','low'], default: 'medium' },
  imageUrl: String,                // featured image (first evidence)
  analyses: [{ type: ObjectId, ref: 'Analysis' }],
  notes: [{ content: String, createdAt: Date }],
  createdBy: { type: ObjectId, ref: 'User' }
}
```
### Analysis (Evidence)
```js
{
  imageUrl: { type: String, required: true },
  originalFilename: String,
  detections: [{ class: String, confidence: Number, category: String, bbox: { x, y, w, h } }],
  forensicReport: {
    sceneOverview: String,
    detectedElements: [String],
    anomalyAnalysis: [String],
    forensicInterpretation: String,
    threatAssessment: { level: String, score: Number, factors: [String] },
    recommendedActions: [String],
    crimeType: String,
    confidence: Number
  },
  threatScore: { type: Number, default: 0 },
  threatLevel: { type: String, enum: ['CRITICAL','HIGH','MEDIUM','LOW','MINIMAL'], default: 'MINIMAL' },
  location: { type: { type: String, enum: ['Point'], default: 'Point' }, coordinates: [Number] },
  analyzedBy: { type: ObjectId, ref: 'User' },
  caseId: { type: ObjectId, ref: 'Case' }
}
```
---

## Front‑End Component Map
| Component | Purpose |
|-----------|---------|
| **CasesPage** | List, filter, and launch the *NewCaseModal*; displays case cards |
| **NewCaseModal** | Multi‑evidence intake – upload images, add per‑image description, set priority & title |
| **CaseDetailPage** | Detailed view of a case, evidence gallery, forensic notes |
| **DashboardPage** | Overview cards, threat‑level pie chart, active homicide zone counters |
| **CrimeMapPage** | Leaflet map – auto‑centers to user location, shows evidence pins, cluster alerts |
| **StatusDropdown** | Glass‑morphic custom dropdown for case status filtering |
| **CaseCard** | Visual summary of a case (title, priority badge, thumbnail) |

---

## Workflow: Creating a New Investigation

```mermaid
flowchart TD
    A[Click "New Case" button] --> B[Open NewCaseModal]
    B --> C[Enter Title, Priority, Brief Description]
    C --> D[Add Evidence Items]
    D --> D1[Upload Image]
    D1 --> D2[Write per-image forensic description]
    D2 --> D3[Add More Evidence?]
    D3 -->|Yes| D1
    D3 -->|No| E[Submit]
    E --> F[Client builds FormData]
    F --> G[POST /api/cases (multipart)]
    G --> H{Auth & Validation}
    H -->|Success| I[Create Case document]
    I --> J[Create Analysis documents for each image]
    J --> K[Link Analyses to Case & set featured image]
    K --> L[Return 201 + case payload]
    L --> M[UI updates Cases list]
    M --> N[Investigation created!]
    H -->|Fail| O[Return error (e.g., missing title, auth expired)]
    O --> P[Show alert with server message]
```
**Key points**
- Each uploaded image becomes an `Analysis` record with its own forensic description (`evidenceDescriptions`).
- The first image (if any) is stored on the `Case.imageUrl` for thumbnail display.
- The back‑end validates the JWT; a missing/expired token returns **401 – Investigator session expired**.

---

## Design System (CSS Variables)
```css
:root {
  --bg-primary: #0a0a0f;
  --glass-border: rgba(255,255,255,0.12);
  --radius-md: 12px;

  --threat-critical: #ef4444;
  --threat-high: #f97316;
  --threat-medium: #eab308;
  --threat-low: #22c55e;
  --threat-minimal: #06b6d4;

  --accent-cyan: #06b6d4;
  --text-primary: #f5f5f5;
  --text-muted: #999;
}
```
All components reference these variables to guarantee a cohesive, premium experience.

---

## Security & Permissions
- **JWT** is verified on every request by `middleware/auth.js`.
- Routes are protected with `router.use(authMiddleware);`.
- Only the `createdBy` user can edit/delete their own cases (future RBAC can be added).
- Uploaded files are stored outside the source tree (`../uploads`) and served **read‑only** via Express static middleware.

---

## Future Enhancements
- Replace the mock `analysisService` with **YOLO‑v8** + **Gemini Vision** for automatic object detection.
- Real‑time WebSocket updates for new evidence arriving while investigators are on the map.
- Role‑based dashboards (admin view, field‑investigator view).
- Export case bundles (PDF + images) for courtroom presentation.
- Unit / integration test suite (Jest, Supertest).

---

## License
This project is open‑source and available under the **MIT License**.

---

*Happy investigating!*
