# Data Vanta - Full Stack Data Visualization Platform

> **AI-Powered Data Analytics & Visualization Platform**

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-production--ready-brightgreen)

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Tech Stack](#tech-stack)
4. [Frontend Documentation](#frontend-documentation)
5. [Backend Documentation](#backend-documentation)
6. [API Reference](#api-reference)
7. [Installation Guide](#installation-guide)
8. [Environment Variables](#environment-variables)
9. [Usage Guide](#usage-guide)
10. [Troubleshooting](#troubleshooting)

---

## Overview

Data Vanta is a comprehensive data analytics platform that enables users to:

- **Upload** CSV/Excel files to a cloud data lakehouse
- **Analyze** data using natural language queries
- **Visualize** results with intelligent chart suggestions powered by LLM
- **Collaborate** through team workspaces and shared sessions

### Key Features

| Feature | Description |
|---------|-------------|
| 🚀 **Smart Upload** | Drag-and-drop file upload with automatic schema detection |
| 🤖 **AI Suggestions** | LLM-powered chart recommendations based on data types |
| 📊 **Multi-Chart** | Support for 10+ chart types (bar, line, pie, scatter, heatmap, etc.) |
| 💬 **Chat Interface** | Natural language queries against your data |
| 👥 **Team Collaboration** | Multi-user workspaces with role-based access |
| 🔐 **Secure Auth** | JWT-based authentication with session management |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           FRONTEND                                   │
│                    Next.js 15 + React 19                            │
│                       (Port 3000)                                   │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   User Auth     │   │  Data Lakehouse │   │   Chart API     │
│   (Node.js)     │   │  (Java/Spark)   │   │   (Python)      │
│   Port 5000     │   │   Port 8080     │   │   Port 8000     │
├─────────────────┤   ├─────────────────┤   ├─────────────────┤
│ • Authentication│   │ • File Upload   │   │ • LLM Suggest   │
│ • Sessions      │   │ • Iceberg Store │   │ • Query Build   │
│ • Chat History  │   │ • SQL Queries   │   │ • Visualization │
│ • Teams         │   │ • Schema Mgmt   │   │ • Validation    │
└────────┬────────┘   └────────┬────────┘   └────────┬────────┘
         │                     │                     │
         └──────────┬──────────┴──────────┬──────────┘
                    │                     │
              ┌─────▼─────┐         ┌─────▼─────┐
              │  MongoDB  │         │   MinIO   │
              │ (Users)   │         │  (Files)  │
              └───────────┘         └───────────┘
```

### Data Flow

```
User Upload → Lakehouse (Iceberg) → Chart API (LLM) → Frontend (Charts)
     │              │                    │                  │
     │              ▼                    ▼                  ▼
     │         MinIO Storage      Schema Analysis     Multi-Type Render
     │              │                    │                  │
     └──────────────┴────────────────────┴──────────────────┘
                            ▼
                     Real-time Dashboard
```

---

## Tech Stack

### Frontend
| Technology | Purpose |
|------------|---------|
| **Next.js 15** | React framework with App Router |
| **React 19** | UI components |
| **TypeScript** | Type safety |
| **Tailwind CSS** | Styling |
| **Turbopack** | Fast bundling |

### Backend Services

| Service | Technology | Purpose |
|---------|------------|---------|
| **User Auth** | Node.js, Express, MongoDB | Authentication, sessions, teams |
| **Data Lakehouse** | Java, Spring Boot, Spark, Iceberg | File storage, SQL queries |
| **Chart API** | Python, FastAPI, OpenRouter LLM | AI suggestions, query building |

### Infrastructure
| Component | Technology |
|-----------|------------|
| **Object Storage** | MinIO (S3-compatible) |
| **Data Format** | Apache Iceberg |
| **Query Engine** | Apache Spark |
| **Message Queue** | Redis |
| **Containerization** | Docker Compose |

---

## Frontend Documentation

### 📁 Project Structure

```
Front_end/vanta-auth-ui/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth pages (login, signup)
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   ├── (dashboard)/              # Protected dashboard pages
│   │   └── dashboard/
│   │       └── page.tsx          # Main chat + visualization UI
│   ├── api/                      # API Route Handlers (proxies)
│   │   ├── chart/execute-prompt/ # → Chart API
│   │   ├── lakehouse/upload/     # → Lakehouse upload
│   │   ├── lakehouse/jobs/       # → Lakehouse job status
│   │   └── chat/preview/         # → Chat service
│   ├── layout.tsx                # Root layout
│   └── globals.css               # Global styles
├── components/
│   ├── dashboard/
│   │   ├── DashboardLayout.tsx   # Main layout wrapper
│   │   ├── Header.tsx            # Top navigation
│   │   ├── Sidebar.tsx           # Chat history + navigation
│   │   ├── ImportModal.tsx       # File upload + polling
│   │   ├── SettingsModal.tsx     # User settings
│   │   └── TeamsModal.tsx        # Team management
│   └── ui/                       # Reusable UI components
├── lib/
│   ├── api-client.ts             # Typed fetch wrapper
│   ├── auth.ts                   # Auth utilities
│   └── types.ts                  # TypeScript interfaces
├── middleware.ts                 # Route protection
└── package.json
```

### 🔑 Key Components

#### 1. Dashboard Page (`app/(dashboard)/dashboard/page.tsx`)

**Main Features:**
- Chat interface with message history
- Multi-type chart rendering (bar, line, pie, scatter, heatmap)
- Data table preview
- Insights panel

**Key State Variables:**
```typescript
const [messages, setMessages] = useState<Message[]>([]);      // Chat history
const [chartSpecs, setChartSpecs] = useState<ChartSpec[]>([]); // Charts
const [tablePreview, setTablePreview] = useState(null);        // Data preview
const [sessionId, setSessionId] = useState<string | null>(null); // Session
```

#### 2. ImportModal (`components/dashboard/ImportModal.tsx`)

**Upload Flow:**
```
File Select → Validate → Upload (FormData) → Poll Job Status → Success
```

**Polling Logic:**
- Interval: 1 second
- Timeout: 60 seconds
- States: `queued` → `processing` → `completed`/`failed`

#### 3. API Proxy Routes

**Why Proxies?**
- Eliminate CORS issues (same-origin requests)
- Hide backend URLs from browser
- Enable server-side token injection

| Route | Target |
|-------|--------|
| `/api/lakehouse/upload` | `http://localhost:8080/api/v1/upload` |
| `/api/lakehouse/jobs/[id]` | `http://localhost:8080/api/v1/jobs/[id]` |
| `/api/chart/execute-prompt` | `http://localhost:8000/execute-prompt` |

#### 4. Middleware (`middleware.ts`)

**Route Protection:**
- Validates JWT token from cookie
- Redirects to login if invalid
- Protects `/dashboard/*` routes

### 📊 Chart Rendering

**Supported Chart Types:**

| Type | Use Case |
|------|----------|
| `bar_chart` | Category comparison |
| `line_chart` | Time series trends |
| `area_chart` | Volume over time |
| `pie_chart` | Distribution/share |
| `scatter_plot` | Correlation analysis |
| `heatmap` | Matrix visualization |
| `histogram` | Value distribution |
| `big_number` | KPI display |

**Chart Selection Logic:**
```
date + numeric    → line_chart, area_chart
category + numeric → bar_chart, pie_chart
numeric + numeric → scatter_plot
single numeric    → big_number, histogram
```

---

## Backend Documentation

### 📁 Project Structure

```
back_end/
├── Chart-API-main/          # Python FastAPI service
│   ├── main.py              # Main app + endpoints
│   ├── charts_config.py     # Chart type definitions
│   └── requirements.txt     # Python dependencies
├── datalakehouse-main/      # Java Spring Boot service
│   ├── api-service/         # REST API layer
│   ├── spark/               # Spark job processor
│   └── docker-compose.yml   # Full stack setup
└── user-auth-main/          # Node.js Express service
    ├── src/
    │   ├── controllers/     # Request handlers
    │   ├── models/          # MongoDB schemas
    │   ├── routes/          # API routes
    │   └── middleware/      # Auth middleware
    └── package.json
```

### 🔐 User Auth Service (Port 5000)

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/auth/register` | User registration |
| POST | `/api/v1/auth/login` | User login |
| GET | `/api/v1/auth/me` | Get current user |
| POST | `/api/v1/chat` | Send chat message |
| GET | `/api/v1/chat/sessions` | Get chat sessions |
| POST | `/api/v1/teams` | Create team |
| GET | `/api/v1/teams` | List teams |

**Authentication:**
- JWT tokens stored in cookies
- Header: `x-auth-token: <token>`
- Token expiry: 24 hours

### 💾 Data Lakehouse (Port 8080)

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/upload` | Upload CSV/Excel file |
| GET | `/api/v1/jobs/{id}` | Get job status |
| POST | `/api/v1/query` | Execute SQL query |
| GET | `/api/v1/query/{id}` | Get query result |

**Upload Processing Flow:**
```
1. Receive file (multipart/form-data)
2. Create job record (Redis)
3. Store file in MinIO
4. Spark reads file, creates Iceberg table
5. Update job status to 'completed'
```

**Query Execution:**
```json
{
  "source": "projectId.tableName",
  "select": [
    {"column": "Region", "as": "Region"},
    {"column": "Revenue", "aggregation": "sum", "as": "Total"}
  ],
  "groupBy": ["Region"],
  "orderBy": [{"column": "Total", "direction": "desc"}],
  "limit": 20
}
```

### 🤖 Chart API (Port 8000)

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/execute-prompt` | Full flow: suggest + query + execute |
| POST | `/suggest-charts` | Get chart suggestions |
| POST | `/build-queries` | Build queries from suggestions |
| GET | `/charts-config` | Get chart type definitions |

**LLM Integration:**
- Provider: OpenRouter
- Default Model: `google/gemini-flash-1.5-8b`
- Temperature: 0 (deterministic)

**Schema-Grounded Suggestions:**
```json
{
  "user_prompts": ["show revenue by region"],
  "project_id": "default_project",
  "table_name": "sales_data"
}
```

**Response:**
```json
{
  "intent": "visualization",
  "charts": [
    {
      "chart_id": 1,
      "chart_type": "bar_chart",
      "encoding": {"x": "Region", "y": "Revenue"},
      "query": {...},
      "data": {
        "labels": ["Asia", "EU", "US"],
        "datasets": [{"data": [770, 3400, 4590]}]
      }
    }
  ]
}
```

---

## API Reference

### Authentication

```bash
# Register
curl -X POST http://localhost:5000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"User","email":"user@example.com","password":"pass123"}'

# Login
curl -X POST http://localhost:5000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"pass123"}'
```

### File Upload

```bash
# Upload CSV
curl -X POST http://localhost:8080/api/v1/upload \
  -F "file=@data.csv" \
  -F "userId=123" \
  -F "projectId=my_project" \
  -F "tableName=sales_data"

# Check Status
curl http://localhost:8080/api/v1/jobs/{jobId}
```

### Chart Generation

```bash
# Generate Charts
curl -X POST http://localhost:8000/execute-prompt \
  -H "Content-Type: application/json" \
  -d '{
    "user_prompts": ["show revenue by region"],
    "project_id": "my_project",
    "table_name": "sales_data"
  }'
```

---

## Installation Guide

### Prerequisites

- Node.js 18+
- Python 3.9+
- Java 17+
- Docker & Docker Compose
- MongoDB
- Redis

### Quick Start

```bash
# 1. Clone repository
git clone https://github.com/Aymona777/Data_Vanta_fullstack.git
cd Data_Vanta_fullstack

# 2. Start Lakehouse (Docker)
cd back_end/datalakehouse-main
docker-compose up -d

# 3. Start User Auth
cd ../user-auth-main
npm install
npm run dev

# 4. Start Chart API
cd ../Chart-API-main
pip install -r requirements.txt
python main.py

# 5. Start Frontend
cd ../../Front_end/vanta-auth-ui
npm install
npm run dev
```

### Service URLs

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| User Auth API | http://localhost:5000 |
| Lakehouse API | http://localhost:8080 |
| Chart API | http://localhost:8000 |
| MinIO Console | http://localhost:9001 |

---

## Environment Variables

### Frontend (`.env.local`)
```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_LAKEHOUSE_URL=http://localhost:8080/api/v1
CHART_API_URL=http://localhost:8000
```

### User Auth (`.env`)
```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/datavanta
JWT_SECRET=your-secret-key
```

### Chart API (`.env`)
```env
OPENROUTER_API_KEY=your-openrouter-key
OPENROUTER_MODEL=google/gemini-flash-1.5-8b
DATALAKE_BASE_URL=http://localhost:8080/api/v1
```

---

## Usage Guide

### 1. User Registration
Navigate to `/signup` and create an account.

### 2. Upload Data
1. Click "Import Data" in dashboard
2. Drag & drop CSV file
3. Enter table name
4. Wait for processing (30-60 seconds)

### 3. Ask Questions
Type natural language queries:
- "Show total revenue by region"
- "Compare sales across categories"
- "What's the trend over time?"

### 4. View Visualizations
Charts appear automatically in the Charts tab.

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| CORS Error | Use proxy routes, not direct URLs |
| "Failed to fetch" | Check backend services are running |
| "No token provided" | Ensure logged in, check cookie |
| Upload timeout | Increase polling timeout, check Spark logs |
| Empty charts | Verify data types match chart requirements |

### Debug Commands

```bash
# Check service health
curl http://localhost:8000/health
curl http://localhost:8080/api/v1/health
curl http://localhost:5000/api/v1/health

# View logs
docker-compose logs -f  # Lakehouse
npm run dev             # User Auth (stdout)
python main.py          # Chart API (stdout)
```

---

## License

MIT License - See LICENSE file for details.

---

## Contributors

- **Ayman** - Full Stack Developer

---

*Built with ❤️ for data-driven decision making*
