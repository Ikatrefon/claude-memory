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

---

## Stan po sesji 2026-06-16 (FPP + kolizje w chocolate-preview.html)

### Plik roboczy
**Pracujemy teraz w `chocolate-preview.html`**, NIE w index.html (index.html = stara wersja, odłożona, ewentualne elementy do wyciągnięcia później). Decyzja Michała.

### Geometria modelu alchemy_game.glb — POPRAWKI do wcześniejszych zapisów
- To **jeden budynek (gabinet alchemika)**, NIE „trzy komnaty Grand Hall/Parlour" — tamto było planem `ROOMS` w index.html, nie tym GLB.
- **DWA poziomy** połączone drewnianymi **schodami**. Parter (podłoga Y≈0) + antresola/piętro (podłoga ~2,3–2,4 m). Schody i wszystkie ściany są wtopione w jeden mesh-skorupę.
- Schody (three.js): podstawa x≈−1,1, z≈0,3 (pas z three ≈ −0,5…+1,0), wznoszą się wzdłuż −X do podestu x≈−3,4 na wysokości ~2,4 m. Stopień ~0,18 m.
- Wymiary sceny: X[−5,2; 4,2] · Y(wys.)[0; 4,9] · Z[−3,3; 3,9]. Fasada z drzwiami/oknami na z≈+3,9. Antresola przy fasadzie ma podłogę ~2,3 m i wolne ~2,3 m nad głową.
- **Mapowanie Blender→three.js**: three.x = Blender.x · three.y = Blender.z (wysokość) · three.z = −Blender.y. (GLTFLoader trzyma glTF Y-up; Blender konwertuje na Z-up.)

### KRYTYCZNE: jak budować collider z paczek bg_
Paczki `bg_<numer>` w GLB to **GRUPY** (puste węzły). Geometria siedzi w meshach-DZIECIACH o nazwach BEZ prefiksu, rozbitych per materiał (`<numer>`, `<numer>_1`, `_2`…). Filtr `mesh.name.startsWith('bg_')` łapie tylko ~5 meshy = 51 tys. tri → ściany i schody NIE kolidują (gracz przez nie przechodzi).
**Poprawnie:** mesh jest architekturą, gdy on LUB którykolwiek **przodek** ma nazwę `bg_*`. Wtedy collider = 105+ meshy ≈ **1,92 mln tri** (pełne ściany, schody, meble). Rekwizyty (bottle_/book_/scroll_/jar_/box_/jug_/parchment_) pomijamy, żeby gracz nie zacinał się na drobiazgach.

### System gracza/kolizji (three-mesh-bvh 0.7.6)
- Kapsuła vs BVH (shapecast + closestPointToSegment), 5 podkroków/klatkę, grawitacja −28.
- `PLAYER`: radius **0,22** (mały — wchodzi w zakamarki), eye stojąc 1,50 m (segLen 1,28), kucając 0,84 m (segLen 0,62). pos.y = czubek kapsuły = oczy.
- Kucanie: przy zmianie stanu przesuwamy pos.y o ±(STAND_SEG−CROUCH_SEG), żeby STOPY zostały na podłodze a oczy się obniżyły (inaczej kucanie podnosi stopy, oczy bez zmian — błąd, który naprawiliśmy).
- Skok JUMP=7 (edge-trigger, tylko onGround). WALK 2,6 / RUN 5,2 (Shift), kucając ×0,5.
- Sterowanie: **PointerLockControls** (klik = wejście). WASD + strzałki ruch · mysz · Shift bieg · **LewyCtrl kucanie** · **Spacja skok** · **C** podgląd colliderów · **R** respawn.
- Start: na antresoli przy fasadzie/drzwiach, tyłem do nich, twarzą w głąb. `SPAWN=(−3.5, 4.0, 3.3)`, `SPAWN_YAW=0` (yaw 0 = patrzy w −Z). Stałe `SPAWN`/`SPAWN_YAW` na górze skryptu = łatwa zmiana.

### Debug/narzędzia (z tej sesji)
- `window.__player`, `window.__collider`, `window.__cam`, `window.__setChoco(bool)`, `window.__fps`.
- Podgląd colliderów: zielona x-ray powłoka (MeshBasicMaterial, depthTest:false, opacity 0,16), toggle **C**.
- Inspekcja modelu w Blenderze headless: `/tmp/inspect_castle.py`, `/tmp/find_stairs.py`, rendery przekrojów (top-cut przez clip kamery ORTHO). Skorupa = mesh zawierający `222227378305`.
- Testy Playwright (chromium w /tmp): chodzenie, schody, kolizje, mechanika (kucanie/skok/strzałki), mapy podłóg przez raycast na `window.__collider`.

### Otwarte / do potwierdzenia przez Michała
- Czy `SPAWN` jest dokładnie przy właściwych drzwiach (pozycja wywnioskowana z renderów).
- Strojenie: prędkość kucania/skoku, ew. blokada „wspinania się" kapsuły na niskie meble (teraz wchodzi na nie jak na schody).
- FPS z podglądem colliderów ON ~48–51 (collider 1,9 mln tri) — do normalnej gry C wyłącza podgląd.
