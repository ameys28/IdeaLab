# Augenblick2026

Collaborative idea workspace with:
- FastAPI backend (REST + WebSocket)
- React + Vite frontend
- MongoDB Atlas for persistence
- Groq integration for AI features

## Tech Stack

- Frontend: React, TypeScript, Vite, Tailwind
- Backend: FastAPI, Uvicorn, Motor (MongoDB)
- Realtime: WebSocket (`/ws/{session_id}`)

## Project Structure

- `frontend/` — React app
- `backend/` — FastAPI app
- `backend/.env.example` — backend env template
- `frontend/.env.example` — frontend env template

## Prerequisites

- Python 3.10+
- Node.js 20+
- npm
- A MongoDB Atlas connection string
- A Groq API key

## Environment Setup

### 1) Backend env

Create `backend/.env` from `backend/.env.example`:

```bash
cp backend/.env.example backend/.env
```

Required values in `.env`:
- `MONGODB_URL`
- `JWT_SECRET_KEY`
- `GROQ_API_KEY`

Optional defaults are already provided in `.env.example`.

### 2) Frontend env

Create frontend env file:

```bash
cp frontend/.env.example frontend/.env
```

Default local values:
- `VITE_API_URL=http://localhost:8000/api/v1`
- `VITE_WS_URL=ws://localhost:8000/ws`

## Run Locally (No Docker)

Open two terminals.

### Terminal 1 — Backend

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Backend runs at `http://localhost:8000`

Health check:
- `GET http://localhost:8000/health`

### Terminal 2 — Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs at:
- `http://localhost:3000`

## Useful Commands

### Backend tests

```bash
cd backend
source .venv/bin/activate
pytest -q
```

### Frontend production build check

```bash
cd frontend
npm run build
```

## API Base Paths

- REST API base: `http://localhost:8000/api/v1`
- WebSocket base: `ws://localhost:8000/ws`

## Troubleshooting

- `401 Unauthorized` on protected routes:
	- Ensure login is successful and tokens are present in local storage.
- CORS errors:
	- Verify `CORS_ORIGINS` in root `.env` includes `http://localhost:3000`.
- Mongo connection issues:
	- Check `MONGODB_URL` and whitelist your IP in MongoDB Atlas.

## Docker

Docker support has been removed from this repository.

## Deployment Guide

### Backend on Render

Use `backend` as the root directory for the Render service. The error in your screenshot happens when Render runs `pip install -r requirements.txt` from the repository root instead of the `backend/` folder.

Set these values in Render:
- Root Directory: `backend`
- Build Command: `pip install -r requirements.txt`
- Start Command: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
- Python version: `3.10+`

Add these environment variables in Render:
- `MONGODB_URL`
- `JWT_SECRET_KEY`
- `GROQ_API_KEY`
- `CORS_ORIGINS` should include your frontend domain and local dev URLs

For the current deployment, `CORS_ORIGINS` must include `https://idealab-two-pi.vercel.app`.

After deploy, verify:
- `GET /health`
- `GET /api/v1/hello`

### Frontend on Vercel

Use `frontend` as the project root in Vercel.

Set these environment variables in Vercel:
- `VITE_API_URL=https://your-backend-domain/api/v1`
- `VITE_WS_URL=wss://your-backend-domain/ws`

If `VITE_API_URL` is not set during the Vercel build, the app falls back to `http://localhost:8000/api/v1`, which will fail from a deployed browser.

Use these build settings:
- Build Command: `npm run build`
- Output Directory: `dist`

The SPA rewrite is already configured in `frontend/vercel.json`, so direct routes like `/login` and `/sessions/:id` should work after refresh.

### Post-deploy checks

1. Open the frontend URL and confirm it loads without console errors.
2. Register a new account.
3. Log in.
4. Create a session.
5. Join the session from another browser or incognito window.
6. Create an idea and confirm it appears for both users.
7. Add a comment and verify websocket updates.

If a production build fails with `sendMessage is declared but its value is never read`, make sure the updated frontend code is deployed. That prop has been removed from the workspace code path.
