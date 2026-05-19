# 🧠 learnmind — AI-Assisted E-Learning Platform

> *Watch. Learn. Be understood.*

learnmind is a full-stack e-learning platform that goes beyond passive video watching. Students watch YouTube-embedded course videos, take auto-generated quizzes, and receive deeply personalised AI-powered diagnostic feedback. The platform doesn't just grade you — it builds a cognitive fingerprint of *how* you think about each concept, then adapts final exams accordingly.

---

## 📸 Screenshots

### Landing Page
> <img width="1903" height="992" alt="1" src="https://github.com/user-attachments/assets/2619a5e9-3382-47f6-b989-338a40ec896b" />
> <img width="1572" height="985" alt="2" src="https://github.com/user-attachments/assets/a77782d1-5527-4144-b1e5-70d6bcc21015" />

### Course Catalogue
> !<img width="1918" height="991" alt="3" src="https://github.com/user-attachments/assets/8c175d89-cfe2-4e38-b053-ad56ceb124a6" />
> <img width="1696" height="1012" alt="4" src="https://github.com/user-attachments/assets/068245ac-3d12-464c-9d72-faf7a2f4e210" />

### Course Detail & Video Player
> <img width="1919" height="997" alt="5" src="https://github.com/user-attachments/assets/7761a1c7-e3aa-4dcf-8b9b-75276eeb279e" />
> <img width="843" height="811" alt="6" src="https://github.com/user-attachments/assets/6b91c189-0899-400c-9baf-42d2590550f2" />

### Student Dashboard
> <img width="1919" height="991" alt="7" src="https://github.com/user-attachments/assets/7f1c7d9e-6754-4c01-b9a8-1029a887b429" />
> <img width="759" height="1017" alt="8" src="https://github.com/user-attachments/assets/dc39d527-41e9-4a0b-ba43-1587ff7a0471" />

### Understanding Depth Panel (Fingerprint Insight)
> <img width="1900" height="805" alt="9" src="https://github.com/user-attachments/assets/5f7e0997-5111-4696-b76a-5209e61cfa44" />
> <img width="850" height="1013" alt="10" src="https://github.com/user-attachments/assets/2dd8eb91-4312-400e-a505-1aaf0e678df0" />

### Quiz / Test Panel
> <img width="1894" height="990" alt="11" src="https://github.com/user-attachments/assets/e0dbd65b-38c4-430d-aef5-bce3d9daaebf" />
> <img width="759" height="1013" alt="12" src="https://github.com/user-attachments/assets/2d347689-a2d3-45e1-a703-9f4e810a2b2d" />

### Capstone Exam
> <img width="1896" height="633" alt="13" src="https://github.com/user-attachments/assets/111c5a3c-5c49-412e-9a60-e10e8626e2e5" />
> <img width="1899" height="997" alt="14" src="https://github.com/user-attachments/assets/f4f7add3-2a3f-49b7-9881-c8cca4f22bf8" />
> <img width="776" height="1014" alt="15" src="https://github.com/user-attachments/assets/ddf58bb8-1a2a-47e8-b413-94fbbc81e875" />

---

## ✨ Core Features

**For Students:**
- Watch YouTube-embedded course videos with automatic progress tracking (marks watched at 90% completion)
- Take auto-generated quizzes after each video — MCQ, short answer, and essay formats
- Receive AI-powered subjective answer grading via Google Gemini
- Track personal performance through an interactive dashboard with score charts and trend lines
- Unlock a personalised final capstone exam, where questions targeting your *specific* weak concepts are AI-generated on the fly
- Real-time notifications via Socket.IO when the capstone exam unlocks

**The Answer Trajectory Fingerprinting System:**
The platform's signature feature. Every time you answer a question, the system doesn't just record right/wrong — it tracks *how* you're wrong. Each concept gets a `fingerprintScore` that weighs four dimensions:
- **Wrong rate** — baseline accuracy on the concept
- **Phrasing variance** — do you fail this concept regardless of how the question is worded?
- **Fast-wrong ratio** — are errors careless (answered too quickly) or genuine gaps?
- **Recovery rate** — after feedback, do you get it right the next time?

This produces three classifications: **ConceptualGap** (deep misunderstanding), **CarelessError** (knows it, just slips), and **Uncertain** (insufficient data). The capstone exam uses ConceptualGap concepts to generate targeted AI questions.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (React + Vite)                    │
│                                                                 │
│  ┌──────────┐  ┌─────────────┐  ┌──────────────┐  ┌────────┐  │
│  │ AuthStore│  │ContextStore │  │  TestStore   │  │CapStore│  │
│  │ (Zustand)│  │  (Zustand)  │  │  (Zustand)   │  │(Zustand│  │
│  └──────────┘  └─────────────┘  └──────────────┘  └────────┘  │
│                                                                 │
│  Pages: HomePage / LoginPage / SignupPage / CoursePage /        │
│         CourseDetailPage / DashboardPage / CapstonePage         │
│                                                                 │
│  HTTP: Axios (withCredentials: true) → localhost:3001/api       │
│  Realtime: socket.io-client → localhost:3001                    │
└────────────────────────────┬────────────────────────────────────┘
                             │ JWT cookie (httpOnly)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   SERVER (Express + Node.js)                    │
│                                                                 │
│  Middleware: cookie-parser → cors → protectRoute (JWT verify)  │
│                                                                 │
│  Routes:                                                        │
│   /api/auth          → auth.controller.js                      │
│   /api/courses       → course.controller.js                    │
│   /api/videos        → video.controller.js                     │
│   /api/tests         → test.controller.js                      │
│   /api/progress      → progress.controller.js                  │
│   /api/student-context → studentContext.controller.js          │
│   /api/dashboard     → dashboard.controller.js                 │
│   /api/capstone      → capstone.controller.js                  │
│                                                                 │
│  Services:                                                      │
│   studentContext.service.js  — MongoDB aggregation pipeline    │
│   fingerprintEngine.service.js — pure fingerprint algorithm    │
│                                                                 │
│  AI Layer (aiEvaluator.js):                                     │
│   evaluateSubjectiveAnswer()  — Gemini grading                 │
│   generateConceptTag()        — auto-tag questions             │
│   generateAiAnalysis()        — personalised feedback          │
│   generateCapstoneMCQ()       — dynamic exam question gen      │
│                                                                 │
│  Realtime: Socket.IO server                                     │
│   emits → context:updated (after test grading)                 │
│   emits → capstone:unlocked (after course completion)          │
└────────────────────────────┬────────────────────────────────────┘
                             │ Mongoose ODM
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                         MongoDB Atlas                           │
│                                                                 │
│  users  students  tutors  courses  videos  tests               │
│  progress  testresults  studentfingerprints  capstonesessions  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗂️ Project Structure

```
E_Learning/
├── backend/
│   └── src/
│       ├── index.js                   # Express entry point + Socket.IO
│       ├── controllers/               # Route handlers
│       ├── routes/                    # Express routers
│       ├── models/                    # Mongoose schemas
│       ├── services/
│       │   ├── studentContext.service.js     # Dashboard data aggregation
│       │   └── fingerprintEngine.service.js  # ATF algorithm
│       ├── lib/
│       │   ├── aiEvaluator.js         # Gemini API integration
│       │   ├── utils.js               # JWT generation
│       │   └── cascadeHooks.js        # GDPR cascade deletes
│       └── seed/                      # Database seed scripts
│
└── frontend/
    └── src/
        ├── App.jsx                    # Routes + auth guards
        ├── store/                     # Zustand state stores
        ├── lib/axios.js               # Preconfigured Axios instance
        ├── components/                # All UI components and pages
        └── pages/                     # Full-page views (Capstone, Results)
```

---

## 🔄 Key Data Flows

### Test Submission Pipeline

```
Student submits answers (TestPanel.jsx)
  │
  ▼
POST /api/tests/:testId/submit
  │
  ├── MCQ answers → graded immediately (server-side)
  │
  ├── Subjective answers → queued for async Gemini evaluation
  │       └── evaluateSubjectiveAnswer() × per question
  │               └── calculateWeightedScore() → totalScore
  │
  ├── TestResult document created in MongoDB
  │
  ├── Progress document upserted
  │
  ├── updateFingerprintsFromResult() — fingerprint engine runs
  │       └── computeFingerprint() per conceptTag
  │               └── StudentFingerprint upserted with new classification
  │
  ├── generateAiAnalysis() — personalised feedback written to Progress
  │
  └── io.emit("context:updated", { studentId }) → Dashboard re-fetches
```

### Capstone Generation Pipeline

```
POST /api/capstone/generate/:courseId
  │
  ├── Fetch StudentFingerprint docs for student + course
  │
  ├── Step 1 — ConceptualGap concepts
  │       └── generateCapstoneMCQ(conceptTag) via Gemini (AI-first)
  │               → scenario-based, code-contextual questions
  │
  ├── Step 2 — Uncertain / CarelessError concepts
  │       └── Pull from seeded isReusable: true question bank
  │
  ├── Step 3 — Seed backfill (if AI call falls short)
  │
  └── CapstoneSession document created (correctIndex server-only)
        └── Client receives session without correctIndex
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 19, Vite 7, TailwindCSS v4, DaisyUI |
| **State** | Zustand |
| **HTTP** | Axios |
| **Routing** | React Router v6 |
| **Charts** | Recharts |
| **Realtime** | Socket.IO (client) |
| **Backend** | Node.js (ESM), Express 5 |
| **Database** | MongoDB via Mongoose 9 |
| **Auth** | JWT in httpOnly cookies |
| **AI** | Google Gemini (`gemini-2.5-flash-lite`) |
| **Realtime** | Socket.IO (server) |
| **Testing** | Jest (unit tests for capstone cooldown logic) |

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- A MongoDB connection string (MongoDB Atlas or local)
- A Google Gemini API key

### 1. Clone the repository

```bash
git clone https://github.com/your-username/learnmind.git
cd learnmind
```

### 2. Configure the backend

```bash
cd backend
cp .env.example .env
```

Edit `backend/.env`:

```env
PORT=3001
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret_here
GEMINI_API_KEY=your_gemini_api_key
NODE_ENV=development
```

Install dependencies and seed the database:

```bash
npm install
node src/seed/seedCourse.js     # Seeds the JavaScript course + videos
node src/seed/seedTests.js      # Seeds per-video tests + capstone question bank
```

### 3. Configure the frontend

```bash
cd ../frontend
npm install
```

The frontend is pre-configured to connect to `http://localhost:3001`. No changes needed for local development.

### 4. Run the application

In two separate terminals:

```bash
# Terminal 1 — Backend
cd backend && npm run dev

# Terminal 2 — Frontend
cd frontend && npm run dev
```

Open `http://localhost:5173` in your browser.

### 5. Run backend tests

```bash
cd backend && npm test
```

---

## 📡 API Reference

All protected routes require a valid `jwt` cookie (set automatically by the browser after login).

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/signup` | — | Register a new student or instructor |
| POST | `/api/auth/login` | — | Login and receive JWT cookie |
| POST | `/api/auth/logout` | — | Clear JWT cookie |
| GET | `/api/auth/check` | 🔒 | Get current authenticated user |
| GET | `/api/courses` | 🔒 | List all courses |
| GET | `/api/videos/course/:courseId` | 🔒 | Get all videos for a course |
| PUT | `/api/videos/:videoId/watch` | 🔒 | Mark a video as watched |
| GET | `/api/progress/course/:courseId` | 🔒 | Get course completion progress |
| GET | `/api/tests/video/:videoId` | 🔒 | Get test for a video (no correct answers) |
| POST | `/api/tests/:testId/submit` | 🔒 | Submit answers for grading |
| GET | `/api/tests/result/:resultId` | 🔒 | Poll for graded result |
| GET | `/api/student-context/:studentId` | 🔒 | Full performance context (dashboard) |
| GET | `/api/dashboard/scores` | 🔒 | Score data for bar chart |
| GET | `/api/dashboard/trends` | 🔒 | Time-series trend data |
| GET | `/api/dashboard/fingerprints` | 🔒 | Concept fingerprint classifications |
| GET | `/api/dashboard/summary` | 🔒 | KPI summary stats |
| GET | `/api/capstone/status/:courseId` | 🔒 | Capstone gate status |
| POST | `/api/capstone/generate/:courseId` | 🔒 | Generate a capstone exam session |
| POST | `/api/capstone/submit/:sessionId` | 🔒 | Submit and grade capstone |
| GET | `/api/capstone/result/:sessionId` | 🔒 | Get graded capstone result |

---

## 🗄️ Data Models

```
User ──────────────────────────────────────────────────────────┐
  │                                                            │
  ├──→ Student (1:1)                                          │
  └──→ Tutor (1:1)                                            │
                                                               │
Course ──→ Video[] ──→ Test (questions[])                      │
                │                                              │
                └──→ Progress (per User per Video)             │
                        └──→ TestResult (answers[], aiScore)   │
                                │                              │
                                └──→ StudentFingerprint ←──────┘
                                      (per User per conceptTag
                                       per Course)
                                            │
                                            ▼
                                     CapstoneSession
                                     (questions generated from
                                      fingerprint classifications)
```

---

## 🧪 Answer Trajectory Fingerprinting — How It Works

The `fingerprintEngine.service.js` maintains a `StudentFingerprint` document for every `(student, conceptTag, course)` combination. After each test submission, four counters are updated:

| Counter | What it tracks |
|---|---|
| `wrongCount / attempts` | Base error rate |
| `phrasingsFailed / phrasingsTotal` | Concept confusion across question variants |
| `fastWrongCount / wrongCount` | Ratio of rushed wrong answers |
| `conceptsRecovered / conceptsFailed` | Learning recovery after feedback |

The `computeFingerprint()` pure function (no DB dependencies — fully unit-testable) computes a weighted `fingerprintScore` from these counters:

```
score = (W_WRONG × wrongRate)
      + (W_PHRASING × phrasingsFailedRate)
      + (W_FAST × fastWrongRate)
      - (W_RECOVERY × recoveryRate)
```

**Classification thresholds** (min 3 attempts required):
- `score >= 0.60` → **ConceptualGap** — shown as "Needs Work" in UI
- `score < 0.30` → **CarelessError** — shown as "Minor Slips"
- Otherwise → **Uncertain** — shown as "Tracking"

---

## 🗺️ Roadmap
- [ ] Student enrollment system (currently hardcoded single course)
- [ ] Admin dashboard and role management
- [ ] Tutor dashboard (currently a placeholder)
- [ ] Forgot password / email verification flow
- [ ] `conceptsRecovered` / `conceptsFailed` counters (recovery dimension fully active)
- [ ] `difficulty` field joined into test history for accurate dashboard filtering
- [ ] AI chatbot tutoring feature (CuriosityLog model + chat routes in progress)

---

## 📄 License

MIT

---

## 🙏 Acknowledgements

- Course content: [Apna College JavaScript Playlist](https://www.youtube.com/playlist?list=PLGjplNEQ1it_oTvuLRNqXfz_v_0pq6unW)
- AI grading & generation: [Google Gemini](https://ai.google.dev/) (`gemini-2.5-flash-lite`)
- Charts: [Recharts](https://recharts.org/)
- UI components: [DaisyUI](https://daisyui.com/)
