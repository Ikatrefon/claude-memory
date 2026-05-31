---
name: project-puzzle
description: Projekt Lindt Puzzle — gra puzzle z banerem Lindt, stan techniczny i architektura
metadata:
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt Lindt Puzzle

## Cel
Interaktywna gra puzzle w obszarze banera Lindt (1440×400px) — user układa baner z 15 kawałków. Cel prezentacyjny dla klienta.

**Why:** kolejna gra-widget do portfolio B2B Lindt, po breakout i maze.
**How to apply:** projekt statyczny HTML/JS, brak backendu, deploy jak poprzednie projekty.

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/PUZZLE/puzzle.html` — all-in-one

## Lokalny serwer
`python3 -m http.server 8091` w katalogu PUZZLE (port 8091)

## GitHub / Deploy
- Repo: jeszcze nie założone
- Subdomena docelowa: `lds-lindt-puzzle.mdmresearch.com` (planowana)
- Nie wdrożone na VPS

## Konfiguracja gry
- Siatka: **5 kolumn × 3 wiersze = 15 kawałków**
- Obszar assembly: **1440×400px** (pełna szerokość banera)
- PIECE_W = 288px, PIECE_H ≈ 133px

## Architektura — kluczowe decyzje

### Baner źródłowy
- Pobierany z: `https://www.lindt.pl/media/wysiwyg/all-30_desktop_PL_4.jpg` (3840×800px)
- Lokalnie cache'owany jako `banner.jpg`
- Skalowanie: **"cover"** — `scale = max(ASSEMBLY_W/imgW, ASSEMBLY_H/imgH)` + crop od środka
- Docelowo: cron job na VPS co 1h parsuje lindt.pl HTML, wyciąga URL `_desktop_` i nadpisuje banner.jpg

### Layout
- Bank (`#bank`) USUNIĘTY — kawałki leżą w lewej części assembly jako "sterta"
- Assembly = pełne 1440×400px, żadne elementy UI nie kradną szerokości obrazu
- Ghost sloty (docelowe miejsca) widoczne przez całą grę

### Flow gry
1. Obrazek złożony wyświetla się przez **3 sekundy** z odliczaniem ("Zapamiętaj układ")
2. Animacja shuffle — kawałki rozlatują się do losowych pozycji w lewej ~260px (stagger co 60ms)
3. Gracz układa kawałki drag & drop
4. Win screen po 10 sekundach od ułożenia ostatniego

### Geometria wypustek — KRYTYCZNE
Stara implementacja (różne formuły dla każdej krawędzi) dawała niedopasowane wypustki/wycięcia.

**Poprawna implementacja: `drawEdge(ctx, x0,y0, x1,y1, nx,ny, tabDir)`**
- Jedna funkcja dla wszystkich 4 krawędzi
- `(nx,ny)` = outward unit normal
- `tabDir`: +1=tab out, -1=blank in, 0=prosta
- Matematycznie gwarantuje że tab i blank są lustrzanymi ścieżkami
- T = TAB_T * len = 0.22 * długość krawędzi (głębokość wypustki)
- W = TAB_W * len = 0.18 * długość krawędzi (półszerokość szyjki)

```javascript
function drawEdge(ctx, x0, y0, x1, y1, nx, ny, tabDir) {
  const dx=x1-x0, dy=y1-y0, len=Math.sqrt(dx*dx+dy*dy);
  const ex=dx/len, ey=dy/len;
  const T=TAB_T*len, W=TAB_W*len;
  const a0x=x0+ex*len*0.3, a0y=y0+ey*len*0.3;
  const a1x=x0+ex*len*0.7, a1y=y0+ey*len*0.7;
  const tipX=(x0+x1)/2+nx*T*tabDir, tipY=(y0+y1)/2+ny*T*tabDir;
  ctx.lineTo(a0x,a0y);
  ctx.bezierCurveTo(a0x+nx*T*0.3*tabDir, a0y+ny*T*0.3*tabDir, tipX-ex*W, tipY-ey*W, tipX, tipY);
  ctx.bezierCurveTo(tipX+ex*W, tipY+ey*W, a1x+nx*T*0.3*tabDir, a1y+ny*T*0.3*tabDir, a1x, a1y);
  ctx.lineTo(x1,y1);
}
```

### Pozycjonowanie kawałków — KRYTYCZNE
Canvas ma `margin: -pad px` (pad = TAB_T * max(PW,PH) + 2).
Z tym marginem: `content_box_left = CSS_left - pad`.

W drag:
- `offX = e.clientX - elRect.left` (względem content box)
- `CSS_left = e.clientX - assemblyRect.left - offX + pad`
- `targetX = col * PIECE_W` (CSS left dla prawidłowego ułożenia)

### Snap
- Dystans snap: `Math.min(PIECE_W, PIECE_H) * 0.45`
- Snap check: `Math.hypot(x - piece.targetX, y - piece.targetY) < snapDist`
  gdzie `x` = CSS left kawałka podczas drag
- Po snap: `pointer-events: none` na ułożonych kawałkach (inaczej blokują kliknięcia)

## Stan na 2026-05-30
- Gra działa lokalnie na porcie 8091
- Wypustki pasują do siebie
- Preview 3s + shuffle animacja zaimplementowane
- Baner wyświetla się bez zniekształceń (cover scaling)
- Nie wdrożone na VPS
- Brak cron joba do auto-odświeżania baneru
- Brak CLAUDE.md i repo GitHub
