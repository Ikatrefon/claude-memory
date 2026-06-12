---
name: project-chocolate-castle
description: "Lindt Chocolate Castle — gra FPP hidden-object desktop, slot machine losuje 3 produkty do znalezienia w zamku, lokalnie port 8097, nie wdrożona"
metadata: 
  node_type: memory
  type: project
  originSessionId: 290770fa-8434-41fe-9aa1-7fd9541223f2
---

# Projekt: Lindt Chocolate Castle

## Cel i pętla gry
Gra desktopowa (NIE mobile) typu hidden-object w 3D FPP: slot machine (jednoręki bandyta) losuje 3 produkty Lindt → gracz wchodzi do zamku i szuka ich w komnatach. Marketingowo: gracz spędza minuty wpatrując się w produkty Lindt.

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/CHOCOLATE CASTLE/` — single-file `index.html` + `ELEMENTS/`
- Serwer lokalny: **port 8097** (`python3 -m http.server`; 8091 był zajęty)
- NIE wdrożona na VPS (stan 2026-06-12), brak repo GitHub

## Przebieg gry (stan po etapie 3, 2026-06-12)
1. Ekran tytułowy "CHOCOLATE CASTLE — Click to enter the castle" (font Marcellus, złoto na czekoladzie — zgodnie z [[feedback-lindt-font]])
2. **Slot machine** — złoto-czekoladowa maszyna, 3 okna, czerwona wajcha po prawej; bębny kręcą się z packshotami i zatrzymują kolejno (lewy→środkowy→prawy), wylosowane okna dostają złotą poświatę; potem nazwy 3 produktów + przycisk ENTER THE CASTLE
3. **Zamek FPP** — HUD: 3 miniaturki szukanych produktów u góry + licznik "0 / 3 FOUND"; ESC = pauza, losowanie się nie powtarza

## Architektura
- Three.js, FPP: WASD + mysz (pointer lock), Shift = bieg, kolizje ze ścianami, head bob, migotanie świateł
- **Data-driven**: tablica `ROOMS` na górze index.html (wymiary, kolory, drzwi, światła) — silnik buduje z tego geometrię
- Katalog `PRODUCTS` — 45 produktów Lindt z packshotami
- 9 GLB pralinek z Arkanoid 3D do ponownego użycia

## Klimat (decyzja Michała — etap 3)
ŻADNYCH "lochów" — przytulny czekoladowy sklep: kremowe ściany, ciemna boazeria ze złotą listwą, podłogi z desek, belki sufitowe, jasne ciepłe światło, mało mgły.
Trzy komnaty: **Grand Hall** (czekoladowa fontanna, kominek + 2 bordowe fotele, 5 regałów, zegar dziadkowy, żyrandole), **Praline Parlour**, trzecia komnata.

## Model TurboSquid "Ancient Alchemy Laboratory" (ID 2558551)
Setup gry: `ELEMENTS/3d/BLENDER ALCHEMY/bl.blend` (532 MB, Blender 5.1.2 w /Applications).

**Inspekcja:** 35 meshy, 1,39 mln quadów (~2,8 mln tri), 129 materiałów, 78 obrazów 4K/6K (~5,4 GB raw GPU), zero świateł/armatur/modyfikatorów, scena 9,4×7,2×5 m. Nazwy obiektów = numery (artefakt importu USD).

## FINALNY PLIK DO GRY: `ELEMENTS/3d/alchemy_game.glb` (33 MB, stan 2026-06-12)
- **2 164 obiekty: 2 129 ruchomych rekwizytów + 35 paczek tła `bg_*`**
- Nazwy: `bottle_001`–499, `book_`405, `scroll_`640, `box_`122, `jug_`54, `jar_`124, `parchment_`285
- **~95 FPS, 4 688 draw calls** (zmierzone Playwright + Chromium z GPU)
- Tekstury 2K JPEG q75, geometria Draco lvl 6; ładowanie 0,4 s
- Użycie w Three.js: `scene.getObjectByName('book_042')`, kategoria po prefiksie nazwy
- Pełne rozbicie (28 344 obiektów) = 5 FPS — NIE robić; hybryda jest konieczna

## Pipeline klasyfikacji rekwizytów (powtarzalny!)
1. `/tmp/cluster_alchemy.py` — split loose parts → klastry po sygnaturze (verts, bbox 2 dec., materiały) → render miniaturek Workbench (1 636 szt., ~20 min)
2. `/tmp/make_sheets.py`, `/tmp/make_cat_sheets.py` — arkusze kontaktowe Pillow (czytelny font Monaco 20px!)
3. Claude OGLĄDA arkusze (Read na PNG) i klasyfikuje → `/tmp/alchemy_classification.json`
4. `/tmp/build_game_glb.py` — split → dopasowanie sygnatur → rename → join tła per paczka → eksport GLB
- Sygnatury są deterministyczne między uruchomieniami — klasyfikacja JSON wystarczy do rebuilda (~5 min)
- Detekcja butelek SZKLANYCH: materiał z Transmission > 0.1; ceramiczne/etykietowane trzeba wyłapać wizualnie!
- Białe walce = ŚWIECE (nie zwoje) — płomienie to osobne billboardy w tle
- W tle celowo: kapelusze czarownic, kule kryształowe, klepsydra, kociołki, miski, kosze, moździerze, szyldy

## Pułapki
- Materiały `KHR_materials_transmission` + `clearcoat` (szkło) — drugi przebieg renderu w Three.js; przy spadku FPS zamienić na `transparent + opacity`
- Klasyfikacja + skrypty zarchiwizowane w `ELEMENTS/3d/classification/` (kopie z /tmp; miniaturki `/tmp/alchemy_thumbs/` można odtworzyć skryptem `cluster_alchemy.py`)

## Inne pliki w ELEMENTS/3d/
- `alchemy_test.glb` (30 MB) — scalona wersja 35 obiektów (porównawcza)
- `gold_bunny.glb` (9,3 MB) — Lindt Gold Bunny ze Sketchfab (CC-BY-4.0, autor dancao — kredyt w `gold_bunny_license.txt` WYMAGANY przy publikacji!). Znormalizowany: 25 cm wysokości, origin na środku podstawy. 154k tri — przy wielu kopiach zrobić decymację. Źródło: `PIN BALL/bunny_model/`
- `alchemy_split.glb` (67 MB) — pełne rozbicie, tylko do inspekcji (5 FPS)

## Tryb czekoladowy — `chocolate-preview.html` (stan 2026-06-12 wieczór)
Cała "czekoladowość" to warstwa w viewerze Three.js — GLB nietknięty, przycisk CHOCOLATE ON/OFF przywraca oryginał w locie (zapisane mapy/kolory materiałów w Map).
- **Tekstury Michała**: `ELEMENTS/TEXTURES/` — wersje 2K zrobione sips-em: `choco_stone.jpg` (karmel→podłoga), `choco_wall.jpg` (mleczna→ściany), `choco_solid.jpg` (gorzka→sufit)
- **Auto-wykrywanie powierzchni**: pole powierzchni + średnia normala per materiał (próbkowanie trójkątów); podłoga ny>0.55 nisko, sufit ny<-0.35 wysoko, ściany |ny|<0.35, tylko materiały area>3 m²; tekstury przez oryginalne UV atlasu (flipY=false!) — działa, bo texel density w atlasie jednolita
- **Światło**: ambient 0.8 + hemi 0.5 (mało wypełnienia = wyraźne cienie), 4 świece-PointLight migoczące sumą 3 sinusów — 2 z cieniami (base 26, mapy 1024) + 2 dopełniające (base 11), fioletowy "magic" point 8±2.5 pod sufitem, FogExp2 0.02, exposure 1.15
- **Płomienie**: detekcja = materiał transparent + dominujący pomarańcz w teksturze (canvas 16×16, avg kolor); emissive 0xffa030 ×2.2 + sprite'y halo (radialny gradient na canvasie, additive, pulsujące) na pozycjach z klastrowania wierzchołków (30 cm)
- **Pył magiczny**: 350 miękkich drobinek (dotTexture gradient) w 3 rozmiarach, additive, dryf + mryganie warstw
- **PUŁAPKA: UnrealBloomPass + KHR_transmission = czarny ekran** — bloom odpada przy tym modelu; glow robić sprite'ami + emissive
- Debug: `window.__sprites`, `__cam`, `__setChoco(bool)`; zrzuty iteracyjne: `/tmp/shot_choco.mjs` (Playwright)
- FPS: ~66 szeroki kadr / ~102 blisko
- Ewentualny tuning: cienie na pozostałych 2 świecach (-10/15 FPS), fioletowe ogniki zamiast pomarańczowych

## Podgląd / narzędzia
- `alchemy-preview.html` — viewer z DRACOLoader: `?glb=plik.glb`, klik = inspekcja obiektu, H/U = ukryj/pokaż, strzałki = chodzenie (Shift = bieg), licznik FPS i draw calls
- Test wydajności: `/tmp/test_split_fps.mjs` (Playwright 1.59 zainstalowany w /tmp, pasuje do chromium-1217 w cache)

## Następny krok (otwarty)
Wpiąć scenę alchemy_game.glb do zamku w index.html, przenosząc tam warstwę czekoladową z chocolate-preview.html (tekstury+światło+płomienie+pył). Michał zaakceptował klimat trybu czekoladowego.
