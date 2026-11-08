# AI-Based Resume Screening and Candidate Shortlisting System

A production-grade, AI-powered recruitment platform that helps recruiters upload resumes, analyze candidate skills, compare resumes against job descriptions, calculate transparent matching scores, rank applicants, and generate shortlisting recommendations.

## Features

- **AI-Powered Screening** – Gemini AI integration with local fallback (TF-IDF + keyword matching)
- **Resume Parsing** – PDF and DOCX support with auto-extraction of skills, education, experience, and projects
- **Transparent Scoring** – Configurable weights for required skills, preferred skills, experience, education, and projects
- **Candidate Ranking** – Evidence-backed scores with matched/missing skills breakdown
- **Shortlisting** – Manual recruiter decisions with audit trail and confirmation dialogs
- **Analytics Dashboard** – Score distribution, skill frequency, screening funnels, and CSV export
- **Candidate Profiles** – 8-tab detailed view (overview, skills, experience, education, projects, resume, AI analysis, notes)
- **Job Management** – Full CRUD with skill tags, search, and filtering
- **Dark/Light Mode** – Persistent theme toggle with premium SaaS design
- **Authentication** – JWT-based login/register with secure password hashing
- **Duplicate Detection** – SHA-256 file hash prevents duplicate resume uploads
- **Demo Mode** – Seed script with 5 synthetic candidates and 3 job descriptions

## Architecture

```
┌──────────────────────────────────────────────────┐
│                   Frontend (React + Vite)         │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │ Sidebar  │ │ TopNav   │ │ Pages (Dashboard, │  │
│  │ Nav      │ │ Search,  │ │ Jobs, Upload,     │  │
│  │ 8 items  │ │ Theme,   │ │ Screening, etc.)  │  │
│  │          │ │ Profile  │ │                    │  │
│  └──────────┘ └──────────┘ └──────────────────┘  │
│  Tailwind CSS │ Recharts  │ Lucide React          │
└────────────────────┬─────────────────────────────┘
                     │ HTTP/REST (Vite Proxy)
┌────────────────────┴─────────────────────────────┐
│                Backend (FastAPI + Python)          │
│  ┌──────────────┐ ┌────────────┐ ┌────────────┐  │
│  │ Auth (JWT,   │ │ Resume     │ │ Screening  │  │
│  │ bcrypt)      │ │ Parser     │ │ Engine     │  │
│  ├──────────────┤ │ (PyMuPDF,  │ │ (Local +   │  │
│  │ Job Routes   │ │ python-    │ │ Gemini AI) │  │
│  │ Candidate    │ │ docx)      │ │            │  │
│  │ Routes       │ │            │ │ TF-IDF,    │  │
│  │ Screening    │ └────────────┘ │ Cosine     │  │
│  │ Routes       │                │ Similarity │  │
│  │ Analytics    │                └────────────┘  │
│  └──────┬───────┘                                │
│         │ SQLite (WAL mode)                      │
│  ┌──────┴───────┐                                │
│  │  Database    │ Users, Jobs, Candidates,       │
│  │  (SQLite)    │ Screening Results, Actions     │
│  └──────────────┘                                │
└──────────────────────────────────────────────────┘
```

## Technology Stack

| Layer      | Technology                                         |
|------------|-----------------------------------------------------|
| Frontend   | React 18, Vite 5, Tailwind CSS 3, Recharts, Lucide |
| Backend    | Python, FastAPI, Pydantic, Uvicorn                  |
| Database   | SQLite (WAL mode, ready for PostgreSQL migration)   |
| Parsing    | PyMuPDF (PDF), python-docx (DOCX)                   |
| AI         | Google Gemini API (optional), TF-IDF fallback       |
| Auth       | JWT (python-jose), bcrypt (passlib)                 |

## Prerequisites

- **Python 3.10+**
- **Node.js 18+** and npm
- (Optional) Google Gemini API key for AI-powered screening

## Installation

### 1. Clone the repository

```bash
git clone <repository-url>
cd "AI resume"
```

### 2. Backend Setup

#### Windows PowerShell
```powershell
cd backend
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env
```

#### macOS / Linux
```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

### 3. Frontend Setup

```bash
cd frontend
npm install
```

## Environment Variables

Edit `backend/.env` to configure:

| Variable                     | Required | Description                                 |
|------------------------------|----------|---------------------------------------------|
| `SECRET_KEY`                 | Yes      | JWT signing key (change for production)     |
| `GEMINI_API_KEY`             | No       | Google Gemini API key for AI screening      |
| `DATABASE_URL`               | No       | SQLite path (defaults to ./data/)           |
| `MAX_UPLOAD_SIZE_MB`         | No       | Max file upload size (default: 10)          |
| `CORS_ORIGINS`               | No       | Allowed origins (default: localhost)        |
| `ACCESS_TOKEN_EXPIRE_MINUTES`| No       | Token expiry (default: 480 = 8 hours)       |

**Gemini API is optional.** Without it, the system uses local screening (TF-IDF + keyword matching). All other features work normally.

## Database Initialization

The database is auto-initialized on first backend startup. No migration commands needed.

## Loading Demo Data

```bash
cd backend

# Windows PowerShell
.\venv\Scripts\Activate.ps1
python -m app.seed_data

# macOS / Linux
source venv/bin/activate
python -m app.seed_data
```

This creates:
- **Demo user**: `demo_recruiter` / `demo123`
- **5 synthetic candidates** with parsed resume data
- **3 job descriptions** (Senior Software Engineer, Data Analyst, Frontend Developer)
- **15 screening results** (all candidate-job combinations)
- **Sample notifications**

### Reset Demo Data

```bash
python -m app.seed_data --reset
```

This safely restores the demo dataset without deleting other users' data.

## Starting the Application

### Start Backend (Terminal 1)

#### Windows PowerShell
```powershell
cd backend
.\venv\Scripts\Activate.ps1
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

#### macOS / Linux
```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Start Frontend (Terminal 2)

```bash
cd frontend
npm run dev
```

### Access the Application

| Service          | URL                              |
|-----------------|----------------------------------|
| **Frontend**    | http://localhost:5173             |
| **Backend API** | http://localhost:8000             |
| **API Docs**    | http://localhost:8000/docs        |
| **ReDoc**       | http://localhost:8000/redoc       |
| **Health Check**| http://localhost:8000/api/health  |

## Running Tests

```bash
cd backend

# Windows PowerShell
.\venv\Scripts\Activate.ps1
pytest tests/ -v

# macOS / Linux
source venv/bin/activate
pytest tests/ -v
```

## API Documentation

FastAPI automatically generates interactive Swagger documentation at `/docs`.

### Key Endpoints

| Method | Endpoint                         | Description                    |
|--------|----------------------------------|--------------------------------|
| POST   | `/api/auth/register`             | Register new recruiter         |
| POST   | `/api/auth/login`                | Login                          |
| GET    | `/api/auth/me`                   | Get current user profile       |
| GET    | `/api/jobs/`                     | List job descriptions          |
| POST   | `/api/jobs/`                     | Create job description         |
| POST   | `/api/candidates/upload`         | Upload resumes (PDF/DOCX)      |
| GET    | `/api/candidates/`               | List candidates                |
| POST   | `/api/screening/screen`          | Run AI/local screening         |
| GET    | `/api/screening/results`         | Get screening results          |
| PUT    | `/api/screening/results/{id}/decision` | Update recruiter decision |
| GET    | `/api/screening/shortlisted`     | Get shortlisted candidates     |
| GET    | `/api/analytics/dashboard`       | Get analytics data             |
| GET    | `/api/analytics/export/csv`      | Export results as CSV          |
| GET    | `/api/health`                    | Health check                   |

## User Workflow

1. **Login** → Use demo credentials or register
2. **Create Job** → Define title, skills, requirements
3. **Upload Resumes** → Drag & drop PDF/DOCX files
4. **AI Screening** → Select job + candidates → Run screening
5. **Review Results** → View scores, matched/missing skills, evidence
6. **Shortlist** → Accept/reject candidates with confirmation
7. **Analytics** → View charts and export CSV reports

## Scoring Model

Default configurable weights (adjustable in Settings):

| Component        | Weight | Description                      |
|-----------------|--------|----------------------------------|
| Required Skills  | 40%    | Skills explicitly required       |
| Preferred Skills | 15%    | Nice-to-have skills              |
| Experience       | 20%    | Years and relevance              |
| Education        | 10%    | Degree level match               |
| Projects         | 15%    | Relevant project evidence        |

Plus a TF-IDF cosine similarity bonus (up to ~10 points).

### Match Categories

| Category        | Default Threshold | Description         |
|----------------|-------------------|---------------------|
| Strong Match   | ≥ 75%             | Highly recommended  |
| Potential Match| ≥ 50%             | Worth reviewing     |
| Needs Review   | < 50%             | Below threshold     |

## Common Errors & Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError` | Activate the virtual environment first |
| `Port 8000 already in use` | Kill the existing process or use `--port 8001` |
| `CORS error in browser` | Ensure backend is running on port 8000 |
| `Gemini API error` | Check your API key in `.env` or leave empty for local fallback |
| `File upload fails` | Check file is PDF/DOCX and under 10MB |
| `Database locked` | Restart the backend server |
| `npm install fails` | Delete `node_modules` and `package-lock.json`, then retry |

## Known Limitations

- **Resume parsing** relies on regex patterns and may miss unconventional formats. Recruiters can manually correct extracted data.
- **Local screening** uses keyword matching and TF-IDF, which is less sophisticated than AI-based analysis.
- **SQLite** is suitable for local development. For production multi-user deployments, migrate to PostgreSQL.
- **Profile editing** (changing password/email) is not yet implemented.
- **Candidate comparison** side-by-side view is available via the screening results table.
- **File storage** is local; for production, use cloud object storage (S3, GCS).

## Security Notes

- API keys are stored in backend `.env` only, never exposed to the frontend
- Passwords are hashed with bcrypt
- JWT tokens expire after 8 hours (configurable)
- All database queries are scoped by `user_id` (tenant isolation)
- File uploads are validated for type, size, and stored with randomized names
- Resume content is treated as untrusted input
- No protected characteristics are used in scoring

## License

This project is for educational and demonstration purposes.
#   A I - r e u m s e - s c r e e n i n g  
 