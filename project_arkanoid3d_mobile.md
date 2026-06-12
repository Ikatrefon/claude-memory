---
name: project-arkanoid3d-mobile
description: "Lindt Arkanoid 3D Mobile — gra breakout na telefon, Three.js r160, 9 GLB pralinki, wdrożona lds-lindt-arkanoid3d.mdmresearch.com"
metadata:
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt: Lindt Arkanoid 3D Mobile

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/ARKANOID3d-mobile/index.html` — single-file HTML5

## Deploy
- URL: `https://lds-lindt-arkanoid3d.mdmresearch.com`
- VPS: kontener `lindt-arkanoid3d` na porcie **8095**
- Dockerfile: `nginx:alpine` + `nginx.conf` (no-cache dla HTML, cache 1h dla assets)
- Deploy: `rsync -az --exclude='.git' ... root@195.35.56.37:/tmp/arkanoid3d-build/` + `docker build --no-cache` na VPS

## Tech Stack
- Three.js r160, `<script type="module">`, importmap z CDN
- GLTFLoader, RoomEnvironment, RoundedBoxGeometry
- 9 GLB modeli praliniek: `ELEMENTS/a.glb` – `i.glb`
- Video tekstury: `ELEMENTS/tlo3.mp4` (taca gameplay, 4.6MB), `ELEMENTS/tlo_4b.mp4` (ekran wygranej, 585KB)

## Mechanika gry
- Grid 3×3 pralinki (9 sztuk), każda HP 1–3 (losowane)
- BALL_SPEED=9.0, WALL_L=-6, WALL_R=6, WALL_TOP=14.5
- Sterowanie: tilt telefonu (gamma), wygładzony przez lerp 0.09
- Czułość: `smoothGamma / 90 * 20` → krawędź przy ~22° przechyłu
- Fazy: `loading → start-screen → ready → playing → won/lost`
- START button → odliczanie 3-2-1 → `launchBall()`

## Tilt — KLUCZOWE (jak Maze Mobile)
```javascript
const _tiltHandler = e => { rawGamma = e.gamma ?? 0 }
window.addEventListener('deviceorientation',         _tiltHandler, true)
window.addEventListener('deviceorientationabsolute', _tiltHandler, true)
```
**Musi być na POZIOMIE MODUŁU** (nie w funkcji, nie po loadModels). `useCapture=true`, `?? 0`. Samsung Android nie dostarcza eventów jeśli listener zarejestrowany za późno.

## Oświetlenie (baked)
- `toneMappingExposure = 0.5`
- AmbientLight: `0xfff0d8`, int=1.2
- DirectionalLight (sun): `0xfff5e0`, int=2.2, pos=(4.5, 6, 20)
- DirectionalLight (fill): `0x7090cc`, int=0.45
- PointLight (rim): `0xffe080`, int=0.7, dist=25, pos=(0,16,-2)
- Ball: color=0xF5D060, metalness=0.55, roughness=0.18, envMapIntensity=0.4
- Paddle: color=0x5a2800, roughness=0.28, metalness=0.22

## Efekty wizualne (stan 2026-06-09)
- **Float animation**: pralinki delikatnie falują (±0.1 jednostki, sin) + kołyszą (±4°, cos), losowa faza
- **Hit ripple**: przy uderzeniu (HP>0) — złoty pierścień (RingGeometry) rozszerza się i zanika 0.3s
- **Destroy ripple**: pomarańczowy duży pierścień (endScale=4.0, dur=0.55s) + 22 cząsteczki
- **Flash**: emissive glow na trafionej pralince (flashTimer=0.22s)
- **Wear**: roughness rośnie, metalness spada z każdym trafieniem

## Ekran wygranej (stan 2026-06-09)
- Brak nakładki CSS — zostajemy w scenie 3D po zniszczeniu ostatniej pralinki
- Taca przełącza się na `tlo_4b.mp4` (odtworzony raz, no loop)
- Kulka ukryta (`ball.visible = false`) w `endGame()`, przywrócona w `startGame()`
- Po zdarzeniu `ended` (fallback: 6s) pojawia się `#btn-win-replay` (bottom: 18%)
- Game Over: ciemna nakładka `#result-screen` z tytułem i wynikiem

## GLB cache
`gridCache` — pierwsze `buildGrid()` klonuje modele i zapisuje do cache. Restart resetuje tylko state (hp, alive, flashTimer, floatPhase), nie klonuje ponownie.

## Shader stall — PUŁAPKA ROZWIĄZANA
Po `buildGrid()` w `init()` wywołujemy `renderer.render(scene, camera)` — kompiluje shadery PBR podczas loadingu, nie przy pierwszym kliknięciu PLAY (co powodowało 5-sekundowe zamrożenie).

## Audio (Web Audio API)
- **Muzyka**: oryginalna 8-sekundowa melodia w C-dur, triangle wave + bass sine, pętla przez gameplay
- Fade-out przy końcu gry (`musicBus.gain.setTargetAtTime(0, t, 0.6)`)
- **SFX**: `sfxPaddle()`, `sfxWall()`, `sfxHit()`, `sfxDestroy()` (noise burst+tones), `sfxLoseLife()`, `sfxTick()` (countdown), `sfxLaunch()`, `sfxWin()`, `sfxGameOver()`
- `initAudio()` wywoływane z user gesture (PLAY / PLAY AGAIN)
- Przycisk 🔊/🔇 prawy dolny róg (`#audio-btn`)

## Pułapki
- `renderer.compile()` → freeze loadingu na mobile → usunięte; zamiast tego 1× `renderer.render()` w `init()`
- Tilt musi być zarejestrowany na poziomie modułu (patrz wyżej)
- `display:none` na `<video>` blokuje `.play()` na Android Chrome → używać `position:fixed;left:-9999px`
- Build zawsze `--no-cache` na VPS (Mac=arm64, VPS=amd64)
- Kolizje praliniek uwzględniają float delta: `cy = (wb.min.y+wb.max.y)/2 + (p.group.position.y - p.baseY)`
- nginx `no-cache` dla HTML konieczny — bez tego Chrome mobile cache'uje JS agresywnie

## Usunięte elementy (cleanup 2026-06-09)
- Panel świateł (💡) i cały jego CSS/JS — niepotrzebny w produkcji
- Wskaźnik γ (tilt debug) — niepotrzebny w produkcji

## Zasoby
- `ELEMENTS/a.glb` – `i.glb` — 9 modeli praliniek
- `ELEMENTS/tlo3.mp4` — video tekstura tacy (portrait 1040×1280, 4.6MB, loop)
- `ELEMENTS/tlo_4b.mp4` — video ekranu wygranej (585KB, once)
- `ELEMENTS/bcg.png`, `ELEMENTS/bcg-b.png` — stare PNG tła (nieużywane)
