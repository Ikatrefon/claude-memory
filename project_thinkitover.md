---
name: project-thinkitover
description: "Projekt ThinkItOver — whiteboard LLM app z mini-chatami na canvasie, wdrożona na thinkitover.mdmresearch.com"
metadata: 
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt ThinkItOver

## Cel
Whiteboard-style interfejs do pracy z LLM — wiele równoległych mini-chatów na canvasie (React Flow). Rozwiązuje problem "dygresji" w głównym wątku — każda dygresja = osobny mini-chat obok głównego. Użytkownik ręcznie kopiuje kontekst między chatami.

**Why:** narzędzie osobiste Michała + do 15 współpracowników, nie jest projektem Lindt.
**How to apply:** desktop-first, nie optymalizować pod mobile.

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/thinkitover/` — Next.js App Router, all-in-one

## GitHub / Deploy
- Repo: `github.com/Ikatrefon/thinkitover` (prywatne) ✅
- Subdomena: `https://thinkitover.mdmresearch.com` — działa z SSL ✅
- VPS: kontener `thinkitover` na porcie 8093
- SQLite volume: `/opt/thinkitover/data/prod.db`
- Obrazy upload: `/opt/thinkitover/data/uploads/` (persistent volume)
- Repo na VPS (do buildów): `/tmp/thinkitover-build/`

## Konto admina
- Email: `ika.trefon@gmail.com`
- Hasło: `Ksaweryk1!`
- AUTH_SECRET: `0d52fbb5c70bcec118c5d47350ae575b`
- ANTHROPIC_API_KEY: w env kontenera na VPS — odczyt: `docker inspect thinkitover | grep ANTHROPIC` (nie zapisywać klucza w tym repo — GitHub push protection blokuje)

## Stack techniczny
- Next.js 16.2.6 (App Router, standalone output)
- React Flow (@xyflow/react) — canvas z węzłami
- Prisma 5 + SQLite — prostota, brak Postgresa
- Anthropic SDK — streaming chat (model: claude-sonnet-4-6)
- Własna sesja (cookie httpOnly + SHA-256 password hash: `SHA256(password + AUTH_SECRET)`)
- Tailwind 4
- @excalidraw/excalidraw 0.18.x — rysowanie (dynamic import, SSR: false)
- mammoth/mammoth.browser — Word → text (dynamic import)

## Kluczowe decyzje architektoniczne
- **Prisma 5** (nie 7) — v7 wymaga adaptera nawet dla SQLite
- **SQLite** zamiast Postgres — 15 userów, backup = 1 plik
- **Własna sesja** zamiast NextAuth — uproszczenie
- **binaryTargets = ["native", "linux-musl-openssl-3.0.x"]** w schema.prisma
- **openssl** w obu stagach Dockerfile (builder + runner)
- **entrypoint.sh** — uruchamia `node ./node_modules/prisma/build/index.js migrate deploy` przed `node server.js`
- **Build na VPS natywnie** (nie cross-compile z Mac) — Mac = arm64, VPS = amd64

## Schemat danych (aktualny)
```
User → Session[]
     → Board[] (bgColor, drawing)
         → Thread[] (headerBg, nodeBg, posX, posY)
             → Message[]
         → Note[] (title, headerBg, noteBg, posX, posY)
         → BoardImage[] (filename, posX, posY, width, height)
```

## Wdrożone funkcje (stan 2026-06-11)
1. Login / lista boardów / tworzenie / usuwanie
2. Canvas: drag, zoom/pan, resize węzłów, strzałki między węzłami
3. **Chaty** — mini-chat z Claude (streaming), tytuł (dbl-click), zwijanie, kolor nagłówka (8 presetów), kolor tła (color wheel), załączniki (PNG/JPG/WebP/GIF/PDF/Word/.txt i inne)
4. **Notatki** — sticky note, tytuł, zwijanie, kolor nagłówka, kolor tła (color wheel)
5. **Obrazy na canvasie** — Ctrl+V lub drag&drop pliku na canvas, resizable, delete
6. **Rysowanie (Excalidraw)** — przycisk ✏️ Draw otwiera Excalidraw jako transparent overlay nad chatami/notatkami. Przycisk "↓ Add to canvas" eksportuje PNG z przezroczystym tłem w dokładnej pozycji rysunku (Excalidraw scene→screen→RF coords), ląduje jako ImageNode React Flow. ✕ anuluje.
7. **Kolor tablicy** — color wheel w toolbarze (obok avatara), zapisywany do DB
8. Tryb dark/light (localStorage), ikony z /icons/
9. Settings: zmiana hasła i nicku
10. User avatar w toolbarze

## Rysowanie — szczegóły techniczne (WAŻNE)
- `DrawingOverlay` montowany tylko gdy `showDrawing=true` (nie persistent overlay)
- Excalidraw z `viewBackgroundColor: 'transparent'` — chaty/notatki widoczne pod spodem
- Konwersja pozycji: `screenX = (sceneX + scrollX) * zoom + offsetLeft` (z appState Excalidraw), potem `rf.screenToFlowPosition({x, y})` (React Flow instance via `onInit` ref)
- Rozmiar node: `width = screenW / rfZoom`, `height = screenH / rfZoom`
- Po "Add to canvas": czyści scenę Excalidraw, czyści DB `board.drawing`, zamyka draw mode
- CSS Excalidraw: `import '@excalidraw/excalidraw/index.css'` w `app/layout.tsx` (NIE w globals.css — Tailwind 4 nie obsługuje `@import` CSS z node_modules bez `style` export condition)
- Overlay: `position: fixed; inset: 0; z-index: 50` — powyżej React Flow (który ma lokalny stacking context)

## Upload API — aktualne
- `POST /api/upload` — multipart, przyjmuje opcjonalne `width` i `height` (float), zapisuje do DB
- Default: `width=320, height=240` (gdy nie podano)
- `GET /api/files/[filename]` — serwuje pliki z UPLOADS_DIR

## Pułapki (odkryte 2026-06-11)
- **Excalidraw CSS** — musi być importowany przez JS: `import '@excalidraw/excalidraw/index.css'` w layout.tsx. CSS `@import` w globals.css z Tailwind 4 = błąd build ("not exported under condition style")
- **Excalidraw z-index** — `fixed inset-0 z-50` daje własny stacking context powyżej React Flow. NIE używać `absolute z-[1001]` — to zakrywa toolbar Excalidraw (portale renderują się niżej)
- **Reset hasła** — `SHA256(password + AUTH_SECRET)`, można przez `docker exec thinkitover node /app/resetpw.js`
- **sqlite3** nie jest zainstalowany na hoście VPS — resetować DB przez `docker exec` z Node + Prisma client

## Deployment (aktualny flow)
```bash
# Na VPS (native build dla amd64):
ssh -i ~/.ssh/id_ed25519 root@195.35.56.37
cd /tmp/thinkitover-build && git pull
docker build -t thinkitover:latest .
docker stop thinkitover && docker rm thinkitover
docker run -d --name thinkitover --restart unless-stopped \
  -p 8093:3000 \
  -v /opt/thinkitover/data:/data \
  -e DATABASE_URL='file:/data/prod.db' \
  -e UPLOADS_DIR='/data/uploads' \
  -e ANTHROPIC_API_KEY='sk-ant-...' \
  -e AUTH_SECRET='0d52fbb5c70bcec118c5d47350ae575b' \
  -e NEXTAUTH_URL='https://thinkitover.mdmresearch.com' \
  thinkitover:latest
```

## Pułapki Dockerfile (ważne!)
```dockerfile
COPY package*.json ./
COPY prisma ./prisma   # ← MUSI być przed npm ci
RUN npm ci
RUN apk add --no-cache openssl  # ← w obu stagach builder i runner
```
