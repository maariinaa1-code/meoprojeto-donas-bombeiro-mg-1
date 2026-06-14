# PRD — Portal IDECAN / Painel Admin Donas (Campina Grande – PB)

## Problema original (sessão Jan/2026)
Projeto trazido do GitHub público `mpintoi-dev/meoprojeto-donas-campinagrande-1`. Após o usuário fazer deploy em VPS usando o código do GitHub, o painel administrativo não contabilizava acessos nem atividade em tempo real. Os tracking events não apareciam no painel quando o site era acessado em outro domínio (VPS) que não o do Emergent.

## Causa raiz identificada
O bundle JavaScript do painel administrativo (`/donaspainel/static/js/main.bc9162b5.js`) tinha a URL do backend **hardcoded** no build:
```
baseURL: "https://html-builder-122.preview.emergentagent.com/api"
```
Esse URL era uma versão antiga do preview Emergent. Em qualquer outro domínio (VPS de produção ou um novo preview do Emergent), o painel continuava chamando esse host antigo — buscando dados em um backend que não recebia o tracking do site real.

## Correção aplicada (Jan/2026)
1. Reescrito `painel-build-src/src/admin/api.js` para usar `baseURL: '/api'` (URL relativa, mesma origem).
2. Adicionado `"homepage": "/donaspainel"` em `painel-build-src/package.json` para que o CRA gere os assets com prefixo correto.
3. Executado `yarn build` em `/app/painel-build-src`.
4. Copiado o resultado para `/app/frontend/public/donaspainel/` (substituindo `static/`, `index.html`, `asset-manifest.json`) e re-injetado o `<script defer src="/donaspainel/admin-extras.js"></script>` no `index.html`.
5. Backend (`backend/admin_routes.py`) já estava OK — `get_real_ip()` honra `X-Forwarded-For`/`X-Real-IP`, e `track_access` faz geolocalização via ip-api.com.

## Validação (curl + browser)
- `POST /api/track/access` com `X-Forwarded-For: 8.8.8.8` → gravou em `db.accesses` e gerou evento em `db.events` com geolocalização "Ashburn/VA".
- Login admin via `/api/admin/auth/login` retorna JWT válido.
- Painel logado em `/donaspainel/` mostra: 3 acessos, eventos em tempo real ("Novo acesso mobile - Ashburn/VA", etc.), funil, atividade 24h, top localizações. Todas as chamadas API confirmadas em `code-retrieval-19.preview.emergentagent.com/api/...` (URL relativa funcionando).

## Arquitetura
- **Backend**: FastAPI + Motor (MongoDB async). Rotas em `backend/admin_routes.py`. Auth JWT (HS256). Tracking público (sem auth) + endpoints admin protegidos.
- **Frontend portal**: HTML estáticos servidos por `frontend/public/*.html` (idecan.html, inscricao.html, etc.), com `assets/tracker.js` enviando eventos para `/api/track/*`.
- **Painel admin**: React 19 (CRA + craco) em `painel-build-src/`, build copiado para `frontend/public/donaspainel/`. Servido como SPA estática.
- **Tracker** (`frontend/public/assets/tracker.js`): usa `sendBeacon` (fallback `fetch`) com `/api/track/access`. `sessionStorage.idecan_access_logged='1'` garante 1 acesso por sessão.

## Implementado nesta sessão
- [Jan/2026] Restauração do projeto a partir do GitHub para `/app/`, instalação de dependências (qrcode, crcmod), serviços rodando via supervisor.
- [Jan/2026] Correção da URL hardcoded do painel — painel agora é portável entre Emergent preview e VPS.

## Itens em backlog / próximos passos
- P1: Documentar no README a configuração de nginx da VPS (proxy `/api` → `localhost:8001`, com `proxy_set_header X-Forwarded-For` e `X-Real-IP`).
- P2: Adicionar healthcheck endpoint dedicado.
- P2: Considerar mover `painel-build-src/` para fora do repo de produção (manter só build em `frontend/public/donaspainel/`) ou criar workflow CI para rebuild automático.
- P2: Sugestão de produto — contador regressivo de prazo nos cards de concurso (urgência → conversão).

## Personas
- **Admin (Donas)**: login em `/donaspainel`, acompanha KPIs, acessos, inscrições, PIX gerado/copiado/baixado, atividade ao vivo.
- **Candidato**: navega pelo portal (idecan.html, inscricao.html, etc.), gera PIX, faz inscrição. Cada primeiro acesso é registrado uma única vez por sessão.

## Requisitos estáticos (core)
- Tracking de primeiro acesso por sessão.
- Painel admin protegido por JWT.
- Geolocalização de IP por ip-api.com (free tier).
- Funil: acesso → inscrição → pix gerado → pix copiado → pix baixado.
- Portabilidade entre Emergent preview e VPS (URL relativa).
