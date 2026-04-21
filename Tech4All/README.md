# ⚡ ChronoGen — Automatic Timetable Generator

**Team Tech4ALL · Watch The Code 2026 · Graphic Era Hill University**

ChronoGen generates conflict-free weekly timetables using a **Genetic Algorithm written from scratch in JavaScript** — no external GA library. It enforces 7 hard constraints and 8 soft constraints, streams live evolution progress to the browser via SSE, and exports the final timetable as CSV, JSON, or a self-contained HTML file.

---

## 🎥 Demo
https://screenrec.com/share/EZO492wrGL

---

## 🚀 Quick Start

### Prerequisites
- [Node.js](https://nodejs.org/) v18+
- [MySQL](https://dev.mysql.com/downloads/) 8.0+

### 1 — Clone and install
```bash
git clone https://github.com/your-username/chronogen.git
cd chronogen
npm install
```

### 2 — Create the database
```bash
# macOS / Linux
mysql -u root -p < Database/schema.sql

# Windows (PowerShell)
Get-Content Database/schema.sql | mysql -u root -p
```

### 3 — Configure environment
```bash
cp .env.example .env
# Open .env and set your MySQL password
```

### 4 — Build Tailwind CSS
```bash
npm run css:build
```
> This compiles `Frontend/src/input.css` → `Frontend/output.css`.  
> Run `npm run css:watch` during development to auto-rebuild on changes.

### 5 — Start the server
```bash
npm start          # production
npm run dev        # development (auto-restarts with nodemon)
```

Open **http://localhost:3000**

---

## 🗂 Project Structure

```
chronogen/
├── Backend/
│   ├── server.js            # Express entry point
│   ├── db.js                # MySQL pool + startup migrations
│   ├── routes/
│   │   ├── upload.js        # Data input: manual, CSV, demo
│   │   ├── generate.js      # GA run, SSE stream, attendance
│   │   └── export.js        # CSV / JSON / HTML export
│   └── ga/
│       ├── genetic.js       # Evolution engine (evolve function)
│       ├── fitness.js       # Fitness = 10000 − penalties
│       ├── constraints.js   # Hard (H1–H6+HU) & soft (S1–S9) checks
│       └── operators.js     # Selection, crossover, mutation
├── Frontend/
│   ├── src/
│   │   └── input.css        # Tailwind source (edit this)
│   ├── output.css           # Compiled Tailwind (auto-generated)
│   ├── index.html           # Landing page
│   ├── index.js
│   ├── input.html           # Data input page
│   ├── input.js
│   ├── generate.html        # GA run + live chart page
│   ├── generate.js
│   ├── output.html          # Timetable viewer page
│   └── output.js
├── Database/
│   └── schema.sql           # Full schema (safe to re-run)
├── uploads/                 # Temp CSV files (git-ignored)
├── .env.example
├── .gitignore
├── tailwind.config.js
└── package.json
```

---

## 📋 Constraint System

### Hard Constraints (violations = unusable timetable)

| ID  | Rule |
|-----|------|
| H1  | A teacher cannot teach two classes at the same time |
| H2  | A class cannot attend two lectures at the same time |
| H3  | A room cannot host two classes at the same time |
| H4  | A class cannot have two subjects in the same period |
| H5  | A teacher can only teach subjects they are qualified for |
| H6  | Room capacity must not be exceeded by the class |
| HU  | A teacher cannot be scheduled during their unavailable periods |

### Soft Constraints (violations reduce fitness score)

| ID  | Rule | Penalty |
|-----|------|---------|
| S1  | Each subject must reach its min lectures/week | 10 × missing |
| S2  | Teacher must not exceed max lectures/week | 5 × excess |
| S3  | Max consecutive lectures per teacher | 3 × violation |
| S4  | Same subject should not appear twice on one day | 2 × violation |
| S5  | Gaps in a class's day schedule (compact mornings) | 4 × gap |
| S6  | Gaps in a teacher's day schedule | 1 × gap |
| S7  | Labs/practical subjects must use lab rooms | 8 × violation |
| S8  | Morning-preference teachers should not get afternoon slots | 2 × violation |
| S9  | Lectures in afternoon when morning slots are free | 5 × violation |

---

## 🔌 API Reference

| Method | Route | Description |
|--------|-------|-------------|
| `POST` | `/api/upload/institution` | Save institution settings |
| `POST` | `/api/upload/teacher` | Add a teacher |
| `POST` | `/api/upload/room` | Add a room |
| `POST` | `/api/upload/subject` | Add a subject |
| `POST` | `/api/upload/class` | Add a class |
| `POST` | `/api/upload/curriculum-entry` | Add a curriculum entry |
| `POST` | `/api/upload/demo` | Load full demo dataset |
| `GET`  | `/api/upload/data` | Fetch all entities |
| `DELETE` | `/api/upload/clear` | Clear all data |
| `POST` | `/api/generate` | Start a GA run → returns `sessionId` |
| `GET`  | `/api/generate/progress/:sid` | SSE live progress stream |
| `GET`  | `/api/generate/timetable` | Fetch saved timetable |
| `POST` | `/api/generate/reschedule` | Drag-drop move a gene |
| `POST` | `/api/generate/substitute` | Find and assign a substitute teacher |
| `POST` | `/api/generate/mark-absent` | Log teacher absence |
| `DELETE` | `/api/generate/undo-absent` | Undo today's absence |
| `GET`  | `/api/generate/attendance` | Attendance log (filterable) |
| `GET`  | `/api/generate/violations` | Re-check constraint violations |
| `GET`  | `/api/export/csv` | Download timetable as CSV |
| `GET`  | `/api/export/json` | Download timetable as JSON |
| `GET`  | `/api/export/html` | Download self-contained HTML |
| `GET`  | `/api/health` | Server health check |

---

## 🐛 Bug Fixes in This Version

1. **Attendance log not showing date** — MySQL returned `absent_date` as a `Date` object instead of a string. The backend now uses `DATE_FORMAT(absent_date, '%Y-%m-%d')` in the SELECT so the frontend always receives a plain `YYYY-MM-DD` string for reliable filtering and display.

2. **Wrong date stored for mark-absent (IST timezone)** — `new Date().toISOString().split('T')[0]` returns UTC date, which gives *yesterday* in India before 05:30 AM. Fixed to use local date parts (`getFullYear / getMonth / getDate`).

3. **Drag-drop border flicker** — `dragleave` fired when the cursor entered a child element inside the drop cell, causing the blue border to flicker. Fixed by checking `td.contains(e.relatedTarget)` before clearing the highlight.

4. **`package.json` wrong `main` path** — pointed to `backend/server.js` (lowercase) but folder is `Backend/` (capital B), breaking `npm start` on Linux. Fixed.

5. **`.gitignore` missing `uploads/`** — temp CSV files from bulk imports would be committed to git. Fixed.

---

## 🛠 Development

```bash
# Watch Tailwind and auto-rebuild CSS on every HTML/JS change
npm run css:watch

# In a separate terminal, run the server with auto-restart
npm run dev
```

---

## 👥 Team

Built with ❤️ by **Team Tech4ALL** for Watch The Code 2026 @ GEHU
