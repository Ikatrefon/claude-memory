---
name: project-gift-catcher
description: "Projekt Lindt Gift Catcher — mobilna gra HTML, łapanie czekoladek do koszyka, wdrożona na lds-lindt-gift-catcher.mdmresearch.com"
metadata: 
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt: Lindt Gift Catcher

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/GIFT CATCHER/index.html` — gra single-file HTML5

## Deploy
- URL: `https://lds-lindt-gift-catcher.mdmresearch.com`
- GitHub: `github.com/Ikatrefon/lds-lindt-gift-catcher`
- VPS: kontener `lindt-gift-catcher` na porcie 8094
- Dockerfile: `nginx:alpine`, `COPY . /usr/share/nginx/html`

**Why:** gra mobilna Lindt do zbierania czekoladek — promocja ze zniżkami świątecznymi.
**How to apply:** mobile-only (DeviceOrientation tilt + touch fallback), font Marcellus.

## Stan 2026-06-25 (rebrand czerwony + trudniejsza rozgrywka)
- **Ekran startowy + końcowy:** tło `ELEMENTY/background-red-A.jpg` (czerwone ze śnieżynkami, 1290×2796). CTA (3 buttony) gradient poziomy `97702A→BB9750→97702A`.
- **Karuzela pralinek (coverflow) na ekranie startowym** (zastąpiła `.floating-items`): 6 packshotów w `#carousel`, IIFE `carousel()`. Układ poziomy: front w środku (duży, ostry, blur 0), boki mniejsze, **tył mocno rozmyty (blur ~9px)** i wciśnięty do środka (`xr=Rx*(0.5+0.5*t)`), `-webkit-box-reflect` = odbicia pod spodem. Obrót **skokowy z pauzą**: `base += 2π/6` co **2600 ms**, przejście CSS 0.8s. Param: Rx=108, scale 0.4+t*0.95, blur `(1-t)^1.25*9`, z-index t*1000. Dopasowane 1:1 do referencji Michała.
- **Tło:** `ELEMENTY/background-red.jpg` (1290×2796, czerwone Lindt LINDOR, CZYSTE — bez blika i bez czarnego paska), `center/cover`. (Backup wersji z wtopionym blikiem: `background-red_blik-backup.jpg`, nieużywany.)
- **Blik:** osobna warstwa **rysowana w kodzie na canvasie** (`drawBlik()` — radialny gradient skalowany na elipsę), w pełni przesuwalny góra/dół jedną zmienną `BLIK_FRAC` (ułamek wysokości ekranu, domyślnie 0.80); szerokość/wysokość: `BLIK_WHALF`/`BLIK_HHALF`. Rysowany przed kulkami i koszykiem. Koszyk siada na nim: `basket.y = min(blikY()-basket.h+10, nad paskiem HUD)`. Decyzja: kodowany blik > PNG (ostry na każdym ekranie, przesuwalny, bez zależności od pliku).
- **Koszyk:** nowy `ELEMENTY/koszyk.png` (czekoladowe pudełko z logo Lindt, 1975×1076) zamiast basket.png.
- **HUD:** czarny pasek na dole (`HUD_H=92`), napisy w **złocie Lindt `#BB9750`** (zamiast #f5c842).
- **Font Optima** (patrz [[feedback-lindt-font]] — wyjątek od Marcellus).
- **Bez girlandy** (usunięta zielona z lampkami z góry).
- **Śnieg:** generowany w JS, **90 płatków** (IIFE `makeSnow`), zamiast 28 statycznych divów.
- **Fajerwerki częstsze:** burst 2–5 rakiet, co **2,5–6 s** (było 7–15 s).
- **Renifery białe** (`#f0f0f0`, było #5a2a08), Rudolf nadal czerwony nos.
- **Trudniej:** prędkość `pralineSpeed()` = `1.1 + r³*6.5 + ramp` (mocno zróżnicowana: wolne i bardzo szybkie); rampa `el<12 ? el*0.06 : 0.72+(el-12)*0.24`. Gęstość spawnu time-based: `el<12 ? 1100-el*40 : 620-(el-12)*55` (min 220) — spokojnie do 12 s, potem bardzo gęsto.

## Dodatki 2026-06-25 (countdown / outro / blur tła)
- **Countdown 3-2-1** przed startem (jak ARKANOID3d): `startGame()` robi reset + `drawStaticFrame()` (taca+blik, tło ostre) + `runCountdown(beginPlay)`. Overlay `#countdown` (z-index 30), animacja `@keyframes cdPop`, beep na każdą cyfrę + „GO!". Dopiero `beginPlay()` ustawia gameRunning, timer, muzykę, gameLoop. Spawn dopiero po odliczaniu.
- **Outro — wybuchy pozostałych pralinek:** `endGame()` → `runOutro()`: niewyłapane pralinki znikają po kolei (**co 480 ms — wolno**) `burstAt()` = kolorowe cząstki (reuse `FWParticle` na game-canvas, `outroParticles`), potem `showResult()`. Gdy brak pralinek → wynik od razu.
- **PRÓBA: blur tła w grze** — tło w osobnej warstwie `#game-bg` (z-index 0, `transform:scale(1.06)` overscan; `#game-container` tylko kolor). W `gameLoop`: ostro→narasta do `BG_BLUR_MAX=5px` przez 12 s i **ZOSTAJE** (nie wraca do ostrości; `endGame` już nie zeruje, reset dopiero przy nowej grze). **Flaga `const BG_BLUR_TRIAL = true` — `false` = wersja pierwotna (ostre tło).**

## Mechanika gry
- Telefon przechylasz → koszyk się porusza (gamma z DeviceOrientation API), touch fallback
- Spadające przedmioty: **TYLKO pralinki LINDOR — 12 wariantów** (Lindo-a..g + 5 packshotów: MILK, Extra Dark, MILK HAZELNUT, MILK SEASALT CARAMEL, MILK STRAWBERRY CREAM). Misie/króliki/pudełka USUNIĘTE (2026-06-22). Wszystkie pts:5, równa waga. Packshoty 150×~420 (ratio ~2.8, folia z ogonkami), nazwy ze spacjami działają w `img.src` bez encode.
- Spadające pralinki **kołyszą się lewo-prawo** (±~13°, `Math.sin(item.t*swayFr+swayPh)*0.22`, rysowane przez ctx.save/translate/rotate/restore) — efekt „powietrza".
- 3 życia, missed item = -1 życie
- Czas: **20 sekund** (stała `GAME_TIME`); licznik liczbowy w HUD obok wyniku z etykietą „sec" (pasek postępu USUNIĘTY — był `#timer-bar`)
- Score → zniżka: 0-49=5%, 50-149=10%, 150-299=15%, 300+=20%
- Kody: LINDT5XMAS / LINDT10XMAS / LINDT15XMAS / LINDT20XMAS
- Wake Lock API (ekran nie gaśnie podczas gry)

## Muzyka — TŁO = plik MP3 (aktualne, 2026-06-29 wieczór)
**Tło muzyczne to teraz `ELEMENTY/music.mp3`** (192kbps, ~20s, `<audio id="bg-music" loop preload>` w HTML), NIE synth.
- `startMusic()` = `bgMusic.currentTime=0; bgMusic.play()` — wołane w **`beginPlay()`** = dokładnie start rozgrywki (po countdownie). `stopMusic()` = `pause()` w `endGame()` → **ekran wyniku jest cichy** (stop następuje przed showResult). Głośność `bgMusic.volume=0.4` (przyciszone na życzenie), loop.
- **Mobile unlock:** `unlockBgMusic()` (wyciszone play→pause) wołane w `onPlayPressed()` (gest), żeby późniejszy `play()` w beginPlay zadziałał na iOS/Android.
- **Wszystkie SFX bez zmian** (beep/playCatch/playMiss/playGameOver na `ctx.destination`).
- Plik: oryginał był w `ELEMENTY/ELEMENTY 2/music.mp3` → skopiowany do `ELEMENTY/music.mp3`. Deploy: scp też `ELEMENTY/music.mp3` na VPS (nowy asset, nie tylko index.html!).
- Syntezowana muzyka (Jingle Bells/sleigh/bass — `CAROL`, `playMelodyNote`, `sleighBell`, `bassNote`, `getMusicGain`) ZOSTAŁA w kodzie ale **nieużywana** (startMusic jej nie woła). Można usunąć przy sprzątaniu albo wrócić, gdyby MP3 odpadło.

## (HISTORIA) Synth-muzyka — Jingle Bells, cieplej, ciszej (zastąpiona przez MP3 2026-06-29)
**Melodia = „Jingle Bells" (parafraza)** w `const CAROL` (nazwa stała, treść podmieniona): C-dur, refren ×2 z zakończeniem G G F D C, pętla ~19,6 s, ~185 BPM. Format `[freq,ms]`, **`freq=0` = pauza** (obsłużone w `playMelodyNote`/`bassNote` — return). Web Audio, proceduralnie.
- **Bus muzyczny** (`getMusicGain`): `musicGain(0.42 — ciszej)` → lowpass 6500 → dry(0.9) + **pogłos** (ConvolverNode, `makeImpulse(1.8s,2.6)`, wet 0.22) → destination. Pogłos+lowpass+sinusy = „mniej 8-bit", cieplej.
- **Melodia** = addytywne **sinusy** (celesta/pozytywka): partiale ×1/×2/×3/×4, miękki atak 12ms + sub-oktawa. (Zrezygnowano z triangle/„ding" — było zbyt retro.)
- **Sleigh bells** `sleighBell(accent)` = szum bandpass + **sinusowe** partiale (były square — usunięte); puls `setInterval(188ms)`, akcent co drugi, `sleighInterval`.
- **Bas** `bassNote(freq/2)` na nutach `dur>=280ms`.
- **Muzyka od EKRANU STARTOWEGO:** `kickMusic()` na pierwszym `pointerdown`/`touchstart` (mobile wymaga gestu) → `resume()` + `startMusic()` (flaga `_musicKicked`). `endGame` NIE woła już `stopMusic` → gra dalej przez całą sesję (start→gra→wynik).
SFX (catch/miss/gameover/countdown) zostają na `ctx.destination` (przebijają się). Pokrętła do iteracji: głośność (`musicGain`), gęstość/siła dzwoneczków, ilość pogłosu (wet), tempo (ms w CAROL).

## Tło i dekoracje (stan 2026-06-03)
- `ELEMENTY/background.png` — bitmapa 480×900px (PIL): gradient, drzewa, gwiazdy, bokeh
- CSS: 27 płatków śniegu `.snow` (animacja `snowfall`), 12 mrugających gwiazdek `.star`
- JS: Santa na saniach (SVG, leci co 14–24s), fajerwerki (canvas `#fw-canvas`, co 7–15s)

## Koszyk
- `ELEMENTY/basket.png` — rozmiar: ~1/3 szerokości canvasu, proporcje z naturalnych wymiarów pliku
- Koszyk wypełnia się złapanymi przedmiotami w górnych 42% wysokości (otwarty wierzch pudełka)
- Ruch koszyka: tilt wygładzony przez lerp (`rawTiltX → tiltX`, faktor 0.09), touch lerp 0.10

## Warstwa canvasów (z-index)
- `background.png` — CSS background
- `.snow`, `.star` — z-index 2
- `#santa` SVG — z-index 3
- `#fw-canvas` — z-index 4 (fajerwerki, Canvas 2D)
- `#game-canvas` — z-index 5 (gra)
- HUD, screens — z-index 10–50

## Fajerwerki (`#fw-canvas`)
- Klasy: `FWRocket` (leci w górę ze smugą, eksploduje), `FWParticle` (grawitacja, zanikanie)
- Kolory: złoty, czerwony, niebieski, zielony, pomarańczowy, biały, fioletowy, różowy
- Co 7–15s burst 1–3 rakiet, rakiety lecą do 8–40% wysokości ekranu

## Santa SVG — ważne szczegóły
- `viewBox="0 0 430 90"`, animacja: `translateX(115vw)` → `translateX(-115vw)` (leci w lewo)
- Renifery są po LEWEJ stronie sań (x=34–256), sanie po PRAWEJ (x=300+)
- **Renifery flipnięte**: każda grupa ma `transform="translate(X+26, Y) scale(-1,1)"` — głowy zwrócone w lewo (kierunek lotu)
- Lejce: `M330,50 C260,40 185,36 130,38 C90,40 55,41 22,44`

## Zasoby graficzne (`ELEMENTY/`)
- `basket.png` — koszyk
- `Lindo-a.png` ... `Lindo-g.png` — 7 wariantów LINDOR
- `bear.png`, `bunny.png` — figury czekoladowe (pts=15)
- `box-a.png`, `box-b.png`, `box-c.png` — pudełka (pts=10)
- `background.png` — tło (wygenerowane PIL)

## Pułapki
- `sed -i` na VPS tworzy nowy inode → Docker bind mount widzi stary plik → używać `docker restart`
- Build zawsze natywnie na VPS (Mac=arm64, VPS=amd64)
- Basket.png ma alfa ~255 wszędzie — rysuj koszyk PRZED przedmiotami w środku (nie po)

---

## Stan po sesji 2026-06-16 (tuning rozgrywki + faktyczny deploy + cache)

### Zmiany rozgrywki (wdrożone, commit 09a7e6c)
- **Czas gry 20 s** (`GAME_TIME = 20`). Wcześniej 45 s.
- HUD: zamiast paska czasu — **licznik liczbowy obok wyniku**: `Time` + `#time-val` + `<span class="unit">sec</span>`. Przy ≤5 s `#time-display` dostaje klasę `.low` (czerwony).
- Prędkość spadania: `vy` mnożone **×1.38** (2026-06-22; było ×1.21). Wzór: `(1.3 + rand*1.2 + (GAME_TIME - timeLeft)*0.018) * 1.38`.
- Więcej produktów: `spawnInterval` start **950 ms**, dolny limit **430 ms** (było 1500/700). Im dłużej, tym gęściej (−12 ms/spawn).
- Progi zniżek bez zmian (0-49=5%, 50-149=10%, 150-299=15%, 300+=20%).
- Uwaga: szybciej+gęściej = trudniej przy 3 życiach; ew. do wyważenia.

### FAKTYCZNY mechanizm deployu (WAŻNE — sprostowanie do „bind mount")
- Kontener `lindt-gift-catcher` działa z **OBRAZU** `lindt-gift-catcher:latest` — **BEZ bind-mountu** (`docker inspect ... .Mounts` = []). Restart policy `unless-stopped`, sieć `bridge`, port **8094:80**.
- Domenę obsługuje **systemowy nginx** (NIE dockerowy workload_nginx): `/etc/nginx/sites-available/lindt-gift-catcher`, `proxy_pass http://localhost:8094`, listen 80+443 (SSL).
- Katalog budowania na VPS: **`/tmp/lindt-gift-catcher/`** (UWAGA: /tmp — ulotny po reboocie; wtedy odtworzyć z repo). Zawiera Dockerfile, default.conf, index.html, ELEMENTY/.
- **Procedura deployu (sprawdzona):**
  1. lokalnie: `git add … && commit && git push` (repo `Ikatrefon/lds-lindt-gift-catcher`)
  2. `scp -i ~/.ssh/id_ed25519 <pliki> root@195.35.56.37:/tmp/lindt-gift-catcher/`
  3. na VPS: `cd /tmp/lindt-gift-catcher && docker build -t lindt-gift-catcher:latest .`
  4. `docker stop lindt-gift-catcher && docker rm lindt-gift-catcher && docker run -d --name lindt-gift-catcher --restart unless-stopped -p 8094:80 lindt-gift-catcher:latest`
  5. weryfikacja: `curl -sI https://lds-lindt-gift-catcher.mdmresearch.com/index.html`

### Cache (fix: telefony pokazywały starą wersję)
- Domyślny nginx nie dawał `Cache-Control` → telefony cache'owały stary `index.html`.
- Dodany **`default.conf`** kopiowany w Dockerfile do `/etc/nginx/conf.d/default.conf`: dla `= /index.html` ustawia `Cache-Control: no-cache, no-store, must-revalidate` + `Pragma no-cache` + `expires -1`.
- Dzięki temu każdy kolejny deploy jest od razu widoczny pod tym samym QR — bez zmiany kodu QR.

### QR i zasoby
- **`ELEMENTY/background.png` brakowało w git** — dodane do repo (live i tak je miało z `/tmp`).
- `qr-gift-catcher.png` koduje **`https://lds-lindt-gift-catcher.mdmresearch.com/?v=2`** (`?v=2` = cache-bust dla już-zacache'owanych telefonów). Generacja: `python3` + `qrcode` (ERROR_CORRECT_H, box_size 16). Dekodowanie do weryfikacji: `opencv-python-headless` (`cv2.QRCodeDetector`).
- Test mobilny: ten sam QR zawsze prowadzi do aktualnie wdrożonej wersji.
