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
Kupiony/pobrany jako potencjalny setup: `ELEMENTS/3d/BLENDER ALCHEMY/bl.blend` (532 MB, Blender 5.1.2 zainstalowany w /Applications).

**Inspekcja (2026-06-12):** 35 meshy, 1,39 mln quadów (~2,8 mln tri), 129 materiałów, 78 obrazów WSZYSTKIE 4K/6K (~5,4 GB raw GPU — za dużo!), zero świateł/armatur/modyfikatorów, scena 9,4×7,2×5 m.

**Eksport testowy — SUKCES:** `ELEMENTS/3d/alchemy_test.glb` — **30 MB** (tekstury zmniejszone do 2K JPEG q75 + Draco level 6). Skrypt: `/tmp/export_alchemy_glb.py` (Blender headless `--background --python`). W GLB: 58 obrazów (deduplikacja), ~1,3 GB raw GPU — OK na desktop; w razie potrzeby KTX2 przez gltf-transform zbije do ~200–300 MB.

**Podgląd:** `alchemy-preview.html` (http://localhost:8097/alchemy-preview.html) — wymaga DRACOLoader (decoder z CDN three@0.160.0).

**Pułapka:** materiały używają `KHR_materials_transmission` + `clearcoat` (szklane butelki) — transmission w Three.js robi dodatkowy przebieg renderu; jeśli FPS spadnie, zamienić na `transparent + opacity`.

## Następny krok (otwarty)
Michał ocenia jakość 2K w podglądzie; jeśli OK — wpiąć scenę alchemy do zamku w index.html.
