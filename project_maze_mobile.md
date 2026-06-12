---
name: project-maze-mobile
description: "Projekt Lindt Maze Mobile — gra 2D labirynt na telefon, sterowanie przechylaniem, Lindorki jako nagrody, wdrożona na VPS"
metadata: 
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt Lindt Maze Mobile

## Cel
Gra mobilna PWA (bez App Store) — gracz przechyla telefon żeby przetaczać złotą kulkę przez labirynt. Zbiera Lindorki → kod rabatowy na zakupy Lindt.

**Why:** kolejna gra-widget do portfolio B2B Lindt, po breakout i puzzle. Dystrybucja przez QR kod.

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/MAZE-MOBILE/maze.html` — all-in-one
Assets: `ELEMENTY/` — background.jpg (440×1400px), Maze.svg (440×1400px), lindor-1.png … Lindor-7.png (140×150px)

## Lokalny serwer
`python3 -m http.server 8092` w katalogu MAZE-MOBILE

## GitHub / Deploy
- Repo: `github.com/Ikatrefon/lindt-maze-mobile` ✅
- Subdomena: `https://lds-lindt-maze-mobile.mdmresearch.com` — działa z SSL ✅
- VPS: kontener `lindt-maze` na porcie 8091, pliki w `/opt/lindt-maze/`
- Index: `index.html` = kopia `maze.html` (nginx serwuje index)
- QR kod: `/Users/michal/CLAUDE PROJECTS/MAZE-MOBILE/qr-maze.png`

## Architektura — kluczowe decyzje

### Sterowanie (KRYTYCZNE)
- `betaBase = 0, gammaBase = 0` — ZAWSZE hardcoded, nie z lastBeta/lastGamma
- Płaski telefon (beta≈0, gamma≈0) = kulka stoi w miejscu
- Przechylenie = ruch kulki
- Dead zone ±2.5° eliminuje szum sensora
- iOS 13+: `DeviceOrientationEvent.requestPermission()` na user gesture (btn-start)

```javascript
betaBase = 0; gammaBase = 0; // flat phone = neutral, always

// W orientation listener:
const DZ = 2.5;
const rawX = Math.max(-35, Math.min(35, lastGamma - gammaBase));
const rawY = Math.max(-35, Math.min(35, lastBeta  - betaBase));
tiltX = Math.abs(rawX) < DZ ? 0 : rawX - Math.sign(rawX) * DZ;
tiltY = Math.abs(rawY) < DZ ? 0 : rawY - Math.sign(rawY) * DZ;
```

### Maze collision
- SVG pre-rasteryzowany do offscreen canvas (440×1400)
- `isWall(x,y)`: pixel R < 110 = ściana (dla fizyki kulki)
- `hasRoomForLindor(x,y)`: box sweep ±14px z progiem R < 200 — uwzględnia antyaliasing SVG
- 20-sample circumference push resolution, 4 iteracje na frame

### Fizyka
```javascript
const FRICTION = 0.86, G_SCALE = 0.038, MAX_SPD = 7, BOUNCINESS = 0.22;
const START = {x:218, y:1365}, FINISH = {x:218, y:28};
const BALL_R = 10; // maze units
```

### Kamera
- `cameraY += (target - cameraY) * 0.1` — smooth follow
- Kulka centrowana pionowo w obszarze gry

### Lindorki (collectibles)
- 7 sztuk, pliki 140×150px → rysowane w stałym rozmiarze LINDOR_H=32 maze-units
- Pozycjonowanie: `findOpen()` spiral search z `hasRoomForLindor()` box sweep
- Zbieranie: promień `BALL_R + 16`
- Animacja zbierania: pop scale 1→1.4x→0 + fade + złote sparkle particles + "+1"

### Nagrody
- 3 Lindorki → LINDOR5 (5% off)
- 5 Lindorki → LINDOR10 (10% off)
- 7 Lindorki → LINDOR15 (15% off)

### Audio (Web Audio API)
- `initAudio()` na user gesture (btn-start)
- Muzyka: pentatoniczna C-dur, BEAT=0.8s, 14-nutowy loop z basem co 4. nucie
- Dźwięk zbierania: 3 wznoszące nuty (G5→C6→E6)
- Dźwięk odbicia: sine 110→55Hz, throttle 180ms
- Fanfara przy wygrywanej: 6 nut triangle

### Wake Lock
- `navigator.wakeLock.request('screen')` — ekran nie gaśnie podczas gry
- Re-request przy `visibilitychange`

### Tocząca się kulka (rolling animation)
- Seam: elipsa prostopadle do kierunku ruchu
- Szerokość = `r * cos(ballRoll)` — zwęża się do linii gdy z boku
- Dwa szwy (front/back, przesunięte o π) dla przekonującego efektu

## Stan na 2026-05-31
- Wdrożona na VPS: https://lds-lindt-maze-mobile.mdmresearch.com
- QR kod wygenerowany: qr-maze.png
- Repo GitHub: NIE STWORZONE (TODO)
- Brak CLAUDE.md w projekcie
- Problem z lindorkami na ścianach: naprawiony (box sweep + R<200 threshold)
- Problem ze sterowaniem (kalibracja): naprawiony (betaBase=0 hardcoded)
