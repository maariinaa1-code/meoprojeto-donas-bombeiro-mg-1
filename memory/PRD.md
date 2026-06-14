# PRD – IDECAN / Donas Campina Grande (now CBMMG)

## Original problem statement
User imported the existing project from GitHub (https://github.com/mpintoi-dev/meoprojeto-donas-campinagrande-2), then asked to:
1. Restore it (clone + install deps + run).
2. Replace the home cards with TWO new cards for "CONCURSO CORPO DE BOMBEIROS MILITAR DE MINAS GERAIS" (Soldado + Tenente).
3. Create dedicated detail pages for each concurso using the official IDECAN saved HTMLs.
4. Make the browser tab title identical to the homepage IDECAN title across all pages.
5. Wire the "INSCRIÇÃO ON-LINE" buttons of the new pages to the existing login/cadastro modal (same flow as quadro-geral et al.).

## Stack
- Backend: FastAPI + MongoDB (admin panel, tracking, PIX). Routes under `/api`.
- Frontend: React (CRA) shell rendering a `<iframe src="/idecan.html">` plus a set of static HTML pages in `/app/frontend/public/`.
- Auth/cadastro: client-side (sessionStorage `idecan_cadastro`), interception by `/assets/login-modal.js`.

## Pages live in /app/frontend/public
- `idecan.html` – home with the section title and the two CBMMG cards.
- `cbmmg-soldado.html` – detail page (Edital Nº 10 – QP-BM, R$ 4.562,30, 321 vagas).
- `cbmmg-tenente.html` – detail page (Edital Nº 09 – CFO, R$ 7.506,80, 21 vagas).
- `inscricao.html` + `inscricao-passo2*.html` – cadastro flow shared across editais.
- `donaspainel*` – admin panel (login `donas` / `Seinao10@@`).

## Completed (2026-06-14)
- Imported repo into `/app`, installed missing python deps (`qrcode`, `crcmod`), supervisor running both services.
- Replaced the 4 original home cards with 2 CBMMG cards (Soldado + Tenente) and updated the section title.
- Added artwork `assets/cbmmg-soldado.jpg` and `assets/cbmmg-tenente.jpg` from user-supplied images.
- Saved the official IDECAN pages as `cbmmg-soldado.html` and `cbmmg-tenente.html`.
- Fixed empty `<title>` in both new pages.
- Added `data-testid="btn-inscricao-online"` and links `/inscricao.html?edital=cbmmg-soldado|cbmmg-tenente` on the "Inscrição On-line" buttons.
- Injected `/assets/notice.js`, `/assets/auth.js`, `/assets/login-modal.js`, `/assets/tracker.js` and relaxed the CSP (`script-src 'self' 'unsafe-inline' data:; connect-src 'self' https:`).
- Testing agent: 100% green on all 7 acceptance criteria.

## Test credentials
- See `/app/memory/test_credentials.md`.

## Backlog / Next actions
- P1: Strip the Cloudflare beacon reference from the imported HTMLs (CSP currently logs a harmless block).
- P1: Move admin credentials and JWT secret from `admin_routes.py` fallback to `backend/.env` (the user already has those in production).
- P2: Remove the `.bak` files (`idecan.html.bak`, `.bak2..5`, `meus-concursos.html.bak`) from `public/`.
- P2: Build dedicated `inscricao-passo2-cbmmg.html` variants if the user wants different city/local lists per concurso.
- P3: Hook up CTA analytics (the existing `tracker.js` is loaded but new editais aren't yet identified in the dashboard).
