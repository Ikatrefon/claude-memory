---
name: project-pickmix
description: "PICK&MIX — mobilna gra fizyczna (Three.js + Rapier3D): pralinki LINDOR toczą się/odbijają w pudełku, flick palcem + tilt. Prototyp wdrożony tymczasowo pod /pickmix/"
metadata: 
  node_type: memory
  type: project
  originSessionId: 42ecb71f-295d-47b5-b01f-77be0ea7d00f
---

# Projekt: PICK&MIX

## Cel
Gra **wyłącznie na telefon**, odpalana z QR. Pralinki LINDOR (nasze modele 3D) toczą się i odbijają w płaskim pudełku jak prawdziwe kule. Gracz pyka je palcem (flick) i przechyla telefon (tilt), by je przesuwać.

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/PICK&MIX/` (UWAGA: `&` w ścieżce — w shellu cytować całą ścieżkę).
- `index.html` — single-file gra
- `MODELS/` — 4 GLB pralinki (kopie z [[project-lindor-models]]: lindor-czerwony/czarny/brazowy/zolty)
- `PODGLAD/`, `qr-pickmix.png`

## Stack
- Three.js r160 (importmap CDN) + **Rapier3D** `@dimforge/rapier3d-compat@0.14` (WASM, `await RAPIER.init()`) + es-module-shims.
- Render: WebGLRenderer, PMREM RoomEnvironment (połysk folii), AmbientLight + DirectionalLight (cienie), kamera lekko z góry (poz. 0,12.5,5.2 patrzy 0,0,0.3).

## Fizyka (Rapier)
- `world` grawitacja (0,-9.81,0). Podłoga + 4 ścianki = `RigidBodyDesc.fixed()` + `ColliderDesc.cuboid`. Pudełko HX=3.2, HZ=5.2, ścianki WALL_H=1.2.
- Kulki: `dynamic()`, `ColliderDesc.ball(R=0.55)`, restitution 0.55, friction 0.85, density 1.2, linearDamping 0.5, angularDamping 0.6, ccd on. GLB skalowany ×(2R).
- Pętla: `world.step()`; co klatkę kopiuj `body.translation()`/`body.rotation()` → mesh.position/quaternion.
- **`world.gravity = {x,y,z}` działa dynamicznie** (setter) — używane do tiltu.

## Sterowanie
- **Flick:** pointerdown → raycast wybiera kulkę; pointermove zbiera punkty na podłodze; pointerup → impuls z prędkości machnięcia (J=0.55, clamp). `setNDC()` przelicza dotyk na NDC.
- **Tilt:** `deviceorientation`+`deviceorientationabsolute` (na poziomie modułu, useCapture). Pozioma składowa grawitacji = przechył. **Baza = telefon PŁASKO (0)** → przy typowym trzymaniu ~45° kulki zsuwają się KU userowi (potwierdzone jako pożądane). TILT_MAXA=4.5, **TILT_FULL_X=18** (lewo/prawo czułe), TILT_FULL_Z=42 (przód/tył), martwa strefa 2°, wygładzenie 0.18. Tilt **aktywny dopiero po starcie** (`firstTouch`).
- Ekran „Dotknij, aby zacząć" → `onFirstTouch()`: chowa start, prosi o zgodę na czujniki (iOS), `enterFullscreenAndLock()`.

## Deploy (TYMCZASOWY — pod ścieżką, nie własna subdomena)
- Pliki na VPS w `/opt/pickmix/`; kontener Docker **`pickmix`** (nginx:alpine, `COPY . /usr/share/nginx/html`) na porcie **8101**.
- Wystawione przez **workile_nginx** (patrz [[infrastructure]]) — `location /pickmix/ { proxy_pass http://195.35.56.37:8101/; add_header Cache-Control "no-cache"; }` w bloku 443 domeny gift-catcher.
- **URL prototypu:** `https://lds-lindt-gift-catcher.mdmresearch.com/pickmix/` (HTTPS, no-cache). QR: `qr-pickmix.png`.
- **Pętla deployu:** edytuj → `rsync index.html → /opt/pickmix/` → `cd /opt/pickmix && docker build -t pickmix:latest . && docker rm -f pickmix && docker run -d --name pickmix --restart unless-stopped -p 8101:80 pickmix:latest`.
- **Docelowo:** własna subdomena `lds-lindt-pickmix.mdmresearch.com` — wymaga: rekord DNS A (Michał, brak wildcard) + cert + blok w workload_nginx.

## OTWARTY PROBLEM (nierozwiązany) — obracanie ekranu na Samsung S24
- Przy MOCNYM przechyle w bok Android włącza autoobrót → ekran kładzie się na poziom. Przy słabym nie (próg autoobrotu).
- `screen.orientation.lock('portrait-primary')` NIE trzyma. Diagnostyka na S24: **FS:NIE** (fullscreen nie wchodzi), **kąt zawsze 0** (`screen.orientation` nie raportuje obrotu), w „Chrome" **tilt nie rusza kulek** (deviceorientation prawdopodobnie nie dochodzi).
- **Hipoteza:** aparat/skaner Samsunga otwiera link w **okrojonym in-app WebView**, gdzie fullscreen + screen.orientation + DeviceOrientation są wyłączone. To środowisko, nie kod. (Patrz [[reference-mobile-web-gotchas]].)
- Próbowane bez skutku: lock bez fullscreena (jak Arkanoid), fullscreen+lock (fullscreenchange), kontr-obrót CSS (po `screen.orientation.angle` — ale ten = 0), nakładka „Obróć do pionu", czułość tiltu na małych kątach.
- **Następny krok:** dodana linijka statusu `FS: kąt: WxH TILT:<licznik> start: Browser/WebView`. Czekam aż Michał odczyta: czy `TILT:` rośnie przy przechylaniu i czy tag to `WebView`. To rozstrzygnie, czy w tej przeglądarce tilt jest w ogóle możliwy.
- Lekcja wstępna: tilt + lock działają w PEŁNYCH przeglądarkach (Chrome/Samsung Internet); zawodzą w in-app WebView. Arkanoid „działał", bo otwierany w normalnej przeglądarce i tiltowany delikatnie (nie przekraczał progu autoobrotu).

## Narzędzia/testy
- Lokalny serwer `python3 -m http.server 8100` w katalogu projektu.
- Playwright w `/tmp` bywa czyszczony — `npm i playwright`, a chromium z cache: `executablePath` = `~/Library/Caches/ms-playwright/chromium_headless_shell-1217/chrome-headless-shell-mac-arm64/chrome-headless-shell`. Symulacja tiltu: `dispatchEvent(new DeviceOrientationEvent('deviceorientation',{gamma,beta}))`. Debug hooki: `window.__balls/__world/__cam/__flick/__state/__dbg`.
