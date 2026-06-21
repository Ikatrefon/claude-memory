---
name: project-pickmix
description: "PICK&MIX — mobilna gra fizyczna (Three.js + Rapier3D): pralinki LINDOR toczą się/odbijają w pudełku, flick palcem + tilt. Prototyp wdrożony tymczasowo pod /pickmix/"
metadata: 
  node_type: memory
  type: project
  originSessionId: 42ecb71f-295d-47b5-b01f-77be0ea7d00f
---

# Projekt: PICK&MIX

## MECHANIKA FINALNA: FLICK PRALINEK + WYBUCH (2026-06-21, najnowsza)
Michał uprościł bilard: USUNIĘTA biała kula. Gracz pyka pralinki BEZPOŚREDNIO palcem do 2 stałych otworów (górne rogi).
- **Flick:** pointerdown→`pickBall` (raycast), pointermove zbiera punkty, pointerup→impuls z prędkości machnięcia. `FLICK_J=1.7, FLICK_MAX=20` (dostrajalne). Otwory/respawn/tilt-gentle bez zmian.
- **Kolorowy wybuch (utrudnienie):** co `EXPL.every=[12,18]`s w losowym miejscu — `armExpl()` pokazuje pulsujący pierścień-zapowiedź (`EXPL.warn=0.6`s), potem `detonate()`: radialny impuls (`EXPL.force=13`, promień `EXPL.r=4.2`, falloff z odległością, planarny=nie wyrzuca poza stół) + 3 rozchodzące się kolorowe pierścienie. Kolory z `EXPL_COLORS`. Tylko w `playing`. Hook `window.__boom()`.
- Wszystkie knoby wybuchu w obiekcie `EXPL` na górze sekcji — łatwe strojenie po teście Michała (częstotliwość/siła/promień/zapowiedź).
- OTWARTE (test na telefonie): wyczucie siły flicka (FLICK_J/MAX) i czy wybuch nie za częsty/mocny.

### Trasa decyzji mechaniki (historia)
tilt-slide + ruchomy otwór (słaba grywalność) → krótko bilard z białą kulą/slingshot (też porzucone) → **finalnie flick pralinek + 2 stałe otwory + wybuch** (sekcja wyżej). Stałe otwory: `HOLES`, `inHole()`, HOLE_R=1.0, CAPTURE_R=0.62, wcięcie HM=1.15. Tilt został jako delikatna pomoc (TILT_MAXA=3.0). Usunięte po drodze: pinball wall-kick, ruchomy otwór, biała kula/slingshot/aimStick, `wallH`. Bandy restitution 0.62, kule linearDamping 0.5.

## Stan: GRYWALNY MOCKUP DO PREZENTACJI (2026-06-21)
Pełny flow zbudowany w `index.html` i wdrożony pod `/pickmix/`. Działa end-to-end (test Playwright, zero błędów):
- **Flow:** entry (plain START) → `page` (screenshot.jpg strony Pick&Mix + nałożony złoty przycisk „PLAY & WIN A DISCOUNT") → `intro` (3D: taca z kulkami od frontu + puste pudełko pod spodem + panel „how to play" + START) → `countdown` 3-2-1 z wjazdem kamery (INTRO_POS 0,8.5,22 → PLAY_POS 0,28,12, smoothstep, jak Arkanoid3d) → `playing` → `paused`/`result`.
- **Smaki:** 12 w `FLAVORS` (Milk, Dark 60%, White, Hazelnut, Salted Caramel, Pistachio, Stracciatella, Extra Dark, Raspberry&Cream, Tiramisu, Strawberry&Cream, Caramel), każdy ma `glb` (własny model folii), `dot` (hex koloru, siatka HUD), `ps` (packshot do pauzy/wyniku).
- **Pętla:** 12 kulek (siatka 4×3, po jednej na smak). Wpadnięcie do otworu → `potBall`: +BOX.batch sztuk tego smaku do liczników, usuń ciało, **respawn TEGO SAMEGO smaku** w górnej części tacy (zawsze 12 smaków na tacy). Koniec gdy `total>=BOX.cap`.
- **BOX:** 1,5 kg, cap 120, batch 8 (parametryzowalne `?cap=&batch=`). Cena 324,45 zł, rabat placeholder 10% (kod LINDORMIX10, kod renderowany monospace bo Marcellus myli 1/0 z I/O).
- **HUD (góra):** miniatura pudełka (pm-box-1-5kg_4.webp) + „1,5 KG" + „N out of 120" + siatka kropek kolorowanych per smak (`order[]`).
- **Pauza:** przycisk pod tacą → modal „Your box so far" z listą zebranych (kolorowa kropka + nazwa + liczba) + Resume/Exit(reload).
- **Wynik:** „Your box is full!" + 10% + kod + kwota − 32,45 zł + pełna kompozycja (lista smaków z liczbami) + Play again.
- **Puste pudełko 3D:** czekoladowy open-box (boxGroup, pozycja y=-4.4 pod tacą) + wieko oparte z boku z logo Lindt (ELEMENTS/Lindt-Logo kopia.png). Widoczne w intro od frontu.
- **PRESERVED:** silnik fizyki/tilt/flick/fullscreen/kontr-obrót z prototypu. Symulacja biegnie w intro/countdown/playing; pot+tilt+flick+wybuch tylko w `playing`.
- **Packshoty (DONE 2026-06-21):** 12 PNG w `ELEMENTS/packshots/` (transparentne tło, pionowe folie). Pauza i wynik pokazują miniaturę packshota (`.ps`, object-fit:contain) zamiast kropki. Michał nie zidentyfikował wszystkich → dopasowane po kolorze folii (makieta): Milk=MILK.png(czerwona), Dark 60%=Lindo-c(niebieska), White=Lindo-g(kremowa), Hazelnut=MILK HAZELNUT.png(ceglasta), Salted Caramel=MILK SEASALT CARAMEL.png(turkus), Pistachio=Lindo-a(zielona), Stracciatella=Lindo-b(mięta), Extra Dark=Extra Dark.png(czarna), Raspberry&Cream=Lindo-f(jasnoróż), Tiramisu=Lindo-d(miedź), Strawberry&Cream=MILK STRAWBERRY CREAM.png(malina), Caramel=Lindo-e(złota). Nazwy plików ze spacjami → `encodeURI()` w src. `dot` (siatka HUD) zaktualizowane do kolorów folii.
- **12 realnych modeli GLB (DONE 2026-06-21):** wygenerowane wszystkie 12 folii LINDOR (patrz [[project-lindor-models]]), zmniejszone do 512 (`gltf-transform resize`, ~0.45–0.69 MB każdy, łącznie ~7 MB) w `MODELS/`. Każdy smak ma własny `glb` (pole w FLAVORS) → na tacy 12 RÓŻNYCH pralinek, BEZ powtórzeń. Mapowanie smak→folia (po kolorze, zgodnie z packshotami): Milk→czerwony, Dark 60%→niebieski-ciemny, White→zielony_ciemny(⚠ brak białej folii, jedyny słaby kolor), Hazelnut→jasny_braz, Salted Caramel→niebieskawy, Pistachio→zielony, Stracciatella→mientowy, Extra Dark→czarny, Raspberry&Cream→rozowy, Tiramisu→brazowy, Strawberry&Cream→rozowy-ciemy, Caramel→zolty. `loadAll` ładuje `FLAVORS.map(f=>f.glb)`, `addBall` używa `texScenes[FLAVORS[flavor].glb]`.
- **TODO/oczekiwane:** Iteracja kadrowania kamery intro (pudełko mało widoczne za panelem). White=ciemnozielona folia (brak białej w skanach) — do ew. poprawy. Magento: hostowana gra + CMS-safe przycisk launch full-page (nie iframe), wynik→koszyk później.
- Debug hooki: `window.__balls/__phase()/__total()/__counts()/__pot(i)/__potAny()/__boom()`.

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
