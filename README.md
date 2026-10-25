# AI Project Builder

A full-stack starter application for generating project code with a server-side AI provider, previewing files, downloading ZIP archives, and publishing to GitHub after OAuth authorization.

## Stack
- Frontend: React + Vite
- Backend: FastAPI
- AI: OpenAI-compatible Chat Completions API (configurable by base URL/model)
- GitHub: OAuth web flow + GitHub REST API
- Document parsing: pypdf, python-docx, Pillow (OCR is optional and requires pytesseract + Tesseract installed)

## Requirements
- Python 3.11+
- Node.js 20+
- An AI provider API key
- A GitHub OAuth App for publishing (optional until you use GitHub)

## Quick start

### 1. Backend
```bash
cd backend
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Edit .env and set AI_API_KEY. For GitHub publishing, set GITHUB_CLIENT_ID,
# GITHUB_CLIENT_SECRET, APP_SECRET_KEY and PUBLIC_BASE_URL.
uvicorn app.main:app --reload --port 8000
```

### 2. Frontend
In another terminal:
```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```
Open http://localhost:5173. The Vite dev server proxies `/api` to FastAPI.

## GitHub OAuth setup
Create an OAuth App in GitHub Developer settings. Set its callback URL to:
`http://localhost:8000/api/github/callback`
Set `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `PUBLIC_BASE_URL=http://localhost:8000` in backend `.env`.
The app requests the `public_repo` scope by default (private repository support requires `repo`; set `GITHUB_SCOPE=repo` if you need private repositories). Use the narrowest scope that supports your intended workflow.

GitHub OAuth access tokens are held in a signed, HTTP-only session cookie. For production, use HTTPS, a durable session store, a managed secret, restrictive CORS, and consider GitHub App installation tokens for finer-grained permissions.

## AI provider configuration
Set `AI_API_KEY`, `AI_BASE_URL` (defaults to `https://api.openai.com/v1`), and `AI_MODEL` in backend `.env`. The backend calls an OpenAI-compatible `/chat/completions` endpoint. If no key is configured, generation returns a clear configuration error rather than a fake result.

## API
- `POST /api/projects/analyze` — validate/normalize topic and optional upload text
- `POST /api/projects/generate` — generate files through configured AI provider
- `GET /api/projects/{project_id}/files` — list safe generated files
- `GET /api/projects/{project_id}/download` — download ZIP
- `GET /api/github/login` — start OAuth
- `GET /api/github/callback` — OAuth callback
- `GET /api/github/repositories` — list repositories for authorized account
- `POST /api/github/publish` — create or update repository contents after explicit confirmation

## Security notes
- AI and GitHub secrets are backend-only.
- Uploads are size-limited and only PDF, DOCX, TXT, PNG, JPG/JPEG/WEBP are accepted.
- Generated paths are normalized and checked to prevent path traversal.
- ZIP export excludes `.env`, credentials, dependency directories, caches, and build output.
- Publishing requires `confirm: true`; credentials and private environment files are rejected.
- The demo uses an in-memory project store and signed cookie sessions; generated projects are lost on restart. For production, use a database, durable session storage, CSRF protections for all state-changing browser requests, HTTPS, rate limiting, and per-user authorization.
- Generated code is untrusted. Review it before running or publishing it.

## Test
```bash
cd backend
pytest
```
