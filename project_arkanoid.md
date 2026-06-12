---
name: project-arkanoid
description: "Projekt Lindt Chocolate Arkanoid — gra breakout/arkanoid z czekoladkami Lindt, wdrożona na VPS"
metadata: 
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt Lindt Chocolate Arkanoid

## Cel
Gra arkanoid/breakout w klimacie Lindt — klocki to czekoladki Lindt (c1–c23). Widget prezentacyjny B2B.

**Why:** pierwsza gra w portfolio Lindt, poprzedza puzzle i maze.

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/PIN BALL/game.html` — all-in-one (29K, May 29)

## GitHub / Deploy
- Repo: `github.com/Ikatrefon/pin_ball` (razem z maze3d w jednym repo) ✅
- Subdomena: `https://lds-lindt-chocolate-arkanoid.mdmresearch.com` — działa z SSL ✅
- VPS: kontener `lindt-arkanoid` na porcie 8092, pliki w `/opt/lindt-arkanoid/`
- Index: `index.html` = kopia `game.html`

## Assety
- `c1.png` – `c23_dluga.png` — czekoladki Lindt (klocki)
- `k1.png` – `k12.png` — inne grafiki (poprzednia wersja)
- `ball2.png`, `pasek_1.png` — piłka i pasek
- `tlo_startowe.png` (1.8MB), `tlo_drugi_ekran.png`, `tlo_strony.png`, `tlo_gry.png` — tła
- `blick.gif`, `hurray.gif` — animacje

## Stan na 2026-06-01
- Wdrożone na VPS ✅
- GitHub: w repo pin_ball ✅
- Wgrana najnowsza wersja game.html (May 29, ekrany win/lose po angielsku)
