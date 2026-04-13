# Oscorp Capstone — Jiran 

AI-powered inventory management system with stock status prediction, transfer recommendations, and restock alerts across multiple store locations.

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/M4MD-M7SN/capstone_oscorp.git
cd capstone_oscorp
```

### 2. Set your Supabase keys

```bash
cp frontend/.env.example frontend/.env
```

Open `frontend/.env` and fill in:

```env
VITE_SUPABASE_URL=https://your-project-ref.supabase.co
VITE_SUPABASE_ANON_KEY=your_anon_key_here
VITE_ML_API_URL=http://localhost:8001
```

> Find these in your Supabase dashboard under **Project Settings → API**.

### 3. Start everything

```bash
./start.sh
```

That's it. The script handles everything automatically:
- Clears macOS quarantine flags (Mac only)
- Installs Python and Node dependencies on first run
- Starts the ML backend on **http://localhost:8001**
- Starts the React frontend on **http://localhost:5173**

Press `Ctrl+C` to stop both services.

---

## Prerequisites

| Tool | Min Version | Install |
|------|-------------|---------|
| Python | 3.9+ | https://python.org |
| pip | — | comes with Python |
| Node.js | 18+ | https://nodejs.org |
| npm | 9+ | comes with Node.js |

---

## Project Structure

```
oscorp_capstone/
├── start.sh              ← Run this to start everything
├── README.md
│
├── frontend/             ← React + Vite app
│   ├── src/
│   ├── .env.example      ← Copy to .env and fill in keys
│   └── package.json
│
└── ml_backend/           ← Python FastAPI ML service
    ├── app.py            ← API server (port 8001)
    ├── supabase_watcher.py
    ├── train.py          ← Run if models need retraining
    ├── saved_models/     ← Pre-trained model artifacts
    └── requirements.txt
```

---

## ML API Reference

Base URL: `http://localhost:8001`
Interactive docs: `http://localhost:8001/docs`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Health check |
| `POST` | `/predict-stock` | Predict stock status for products |
| `POST` | `/recommend` | Predictions + transfer & restock recommendations |
| `GET` | `/feature-importance` | Top model features |

### Stock Labels

| Label | Meaning | Action |
|-------|---------|--------|
| `Low` | Running low | Receive transfers or restock |
| `Optimum` | Healthy | No action needed |
| `Excess` | Overstocked | Donate to Low stores |
| `Slow` | Slow-moving | Transfer to higher-demand stores |
| `Dead` | No recent sales | Clear out stock |

---

## Supabase Watcher

The ML backend automatically watches for prediction triggers from Supabase. To trigger a full recommendation run from the Supabase SQL editor:

```sql
INSERT INTO ml_triggers (status, trigger_type)
VALUES ('pending', 'recommend');
```

Results are saved to `ml_backend/ml_outputs/`.

---

## Troubleshooting

**`Operation not permitted` on Mac**
```bash
xattr -cr frontend/
xattr -cr ml_backend/
```

**ML models missing**
```bash
cd ml_backend && python3 train.py
```

**Port already in use**
```bash
# Kill whatever is on port 8001 or 5173
lsof -ti:8001 | xargs kill -9
lsof -ti:5173 | xargs kill -9
```

**npm install fails with EACCES**
```bash
xattr -cr frontend/
rm -rf frontend/node_modules
cd frontend && npm install
```
