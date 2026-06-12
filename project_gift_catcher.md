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

## Mechanika gry
- Telefon przechylasz → koszyk się porusza (gamma z DeviceOrientation API), touch fallback
- Spadające przedmioty: 7 wariantów LINDOR, niedźwiedź, królik, 3 pudełka (ważone losowanie)
- 3 życia, missed item = -1 życie
- Czas: 45 sekund, timer jako pasek postępu
- Score → zniżka: 0-49=5%, 50-149=10%, 150-299=15%, 300+=20%
- Kody: LINDT5XMAS / LINDT10XMAS / LINDT15XMAS / LINDT20XMAS
- Wake Lock API (ekran nie gaśnie podczas gry)

## Muzyka
Carol of the Bells — Web Audio API: triangle wave + sub-octave sine + shimmer fifth oscillator.

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
