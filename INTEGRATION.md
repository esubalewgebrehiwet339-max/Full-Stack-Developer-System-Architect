# Frontend ↔ Backend Integration Guide

## What was connected

| Feature | Before | After |
|---|---|---|
| Contact form | Simulated with a 1.8 s timeout | Real `POST /api/contact` to PHP backend |
| Projects section | Static HTML cards | Fetched from `GET /api/projects` dynamically |
| CORS | Hardcoded to port 3000/5500 | Comma-separated `FRONTEND_URL` env var |
| API base URL | Scattered / missing | Single `js/config.js` file |

---

## Quick Start

### 1. Backend — PHP built-in server

```bash
cd portfolio-php

# Install dependencies
composer install

# Create environment file
cp .env.example .env
# ✏️  Edit .env — set DB_*, JWT_SECRET, MAIL_*, etc.

# Create the database
mysql -u root -p -e "CREATE DATABASE portfolio CHARACTER SET utf8mb4"
mysql -u root -p portfolio < config/schema.sql

# Seed admin user + sample projects
php scripts/seed.php

# Start the development server on port 8000
php -S localhost:8000 -t public
```

### 2. Frontend — Live Server (VS Code) or any static server

Open `index.html` with VS Code Live Server (runs on port 5500 by default), or:

```bash
cd portfolio
npx serve .         # serves on http://localhost:3000
```

The `js/config.js` file already points `API_BASE` to `http://localhost:8000`.
Change it if you run the backend on a different port.

---

## Production Deployment

### Same domain (recommended)
Place the PHP backend at `/api` on the same domain as the frontend.
In `js/config.js`, set:
```js
window.PORTFOLIO_API_BASE = '';   // same origin — no CORS needed
```

### Separate domains / subdomains
In `portfolio-php/.env`, set:
```
FRONTEND_URL=https://yourdomain.com
```
In `js/config.js`, set:
```js
window.PORTFOLIO_API_BASE = 'https://api.yourdomain.com';
```

---

## API Endpoints Used by the Frontend

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/projects?limit=50` | Load all projects for the grid |
| `POST` | `/api/contact` | Submit contact form |
| `GET` | `/api/health` | Health check |

### Contact form JSON body

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "project_type": "Web App",
  "budget": "$2,000",
  "message": "I would like to discuss a project..."
}
```

### Projects API response shape

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "My Project",
      "slug": "my-project",
      "description": "Short description",
      "category": "web",
      "tags": ["React", "Node.js"],
      "demo_url": "https://demo.example.com",
      "github_url": "https://github.com/user/repo",
      "featured": true,
      "icon": "🚀"
    }
  ]
}
```

> **Note:** If the API is unreachable (e.g. backend not running), the frontend
> gracefully falls back to the static project cards already in `index.html`.
> The contact form will show a user-friendly error instead of silently failing.
