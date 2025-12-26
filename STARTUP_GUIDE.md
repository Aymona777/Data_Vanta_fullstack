# Vanta Platform - Startup Guide

## Quick Start

Run all 4 services in order:

### 1. PostgreSQL (for user-auth)
Ensure PostgreSQL is running locally with:
- Database: `user_auth`
- User: `postgres`
- Password: `password`

### 2. Start user-auth (Port 5000)
```bash
cd back_end/user-auth-main
npm install
npm start
```
Wait for: `Server is running on http://localhost:5000`

### 3. Start datalakehouse (Port 8080)
```bash
cd back_end/datalakehouse-main
# Copy .env.example to .env first time
cp .env.example .env
docker compose up -d --build
```
Wait for all containers to be healthy.

### 4. Start Chart-API (Port 8000)
```bash
cd back_end/Chart-API-main
pip install -r requirements.txt
python main.py
```
Wait for: `Uvicorn running on http://127.0.0.1:8000`

### 5. Start Frontend (Port 3000)
```bash
cd Front_end/vanta-auth-ui
npm install
npm run dev
```
Open: http://localhost:3000

---

## Environment Files

### Frontend: Front_end/vanta-auth-ui/.env.local
```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_LAKEHOUSE_URL=http://localhost:8080/api/v1
NEXT_PUBLIC_CHART_API_URL=http://127.0.0.1:8000
```

### user-auth: back_end/user-auth-main/.env
```env
PORT=5000
URL=http://localhost:5000
NODE_ENV=development
DB_STRING=postgres://postgres:password@localhost:5432/user_auth
JWT_SECRET=your-secure-secret-key-here
```

### Chart-API: back_end/Chart-API-main/.env
```env
OPENROUTER_API_KEY=your-openrouter-key
OPENROUTER_BASE_URL=https://openrouter.ai/api/v1
OPENROUTER_MODEL=mistralai/mistral-7b-instruct:free
DATALAKE_BASE_URL=http://localhost:8080/api/v1
```

### datalakehouse: back_end/datalakehouse-main/.env
```env
# Copy from .env.example - defaults work for local dev
API_SERVICE_PORT=8080
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=password123
```

---

## Smoke Tests

After starting all services, verify each:

```bash
# user-auth
curl http://localhost:5000/api/v1/auth/me
# Expected: 401 (no token) - confirms service is up

# datalakehouse
curl http://localhost:8080/actuator/health
# Expected: {"status":"UP"}

# Chart-API
curl http://127.0.0.1:8000/charts-config
# Expected: JSON with chart configs

# Frontend
curl http://localhost:3000
# Expected: HTML response
```

---

## Ports Summary

| Service | Port |
|---------|------|
| Frontend | 3000 |
| user-auth | 5000 |
| datalakehouse | 8080 |
| Chart-API | 8000 |
