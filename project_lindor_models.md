---
name: project-lindor-models
description: "Pipeline modeli 3D pralinek LINDOR (kule w folii, game-ready GLB) — Blender headless + PIL + gltf-transform. 4 kolory w /3d/MODELS/LINDORKI"
metadata: 
  node_type: memory
  type: project
  originSessionId: 42ecb71f-295d-47b5-b01f-77be0ea7d00f
---

# Modele 3D pralinek LINDOR

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/3d/MODELS/LINDORKI/`
- **12 GLB (2026-06-21):** czerwony, czarny, brazowy, zolty + jasny_braz, mientowy, niebieskawy, niebieski-ciemny, rozowy-ciemy, rozowy, zielony, zielony_ciemny (game-ready 1024 tex, ~1.0–1.6 MB). Michał dorzucił 8 nowych skanów folii do TEKSTURY/ (wszystkie 1575×950), ja przepuściłem przez pipeline.
- **bbox logo per skan (do make_tex.py, logo=owal, patch=pas wzoru nad logami `[250,20,1150,240]`):** jasny_braz `[515,290,955,625]`, mientowy `[950,260,1460,610]`, niebieskawy `[535,285,960,620]`, niebieski-ciemny `[945,275,1435,575]`, rozowy-ciemy `[955,260,1435,580]`, rozowy `[720,295,1210,620]`, zielony `[920,255,1450,605]`, zielony_ciemny `[720,290,1210,620]`. Oryginały: czerwony `[838,278,1332,616]`/patch`[300,10,1100,250]`, czarny `[890,265,1450,625]`/`[300,20,1100,250]`, brazowy `[555,270,1085,620]`/`[250,15,1050,200]`, zolty `[660,270,1255,620]`/`[250,10,1100,130]`. Rotacja kuli Z=248° dla wszystkich.
- **Skrypty (przetrwały w /tmp):** `/tmp/make_tex.py` (PIL: equirect 1024×512 + normal), `/tmp/build_lindor.py` (Blender headless, args: color_tex normal_tex out_glb render_prefix rot_deg), `/tmp/sweep_lindor.py` (szukanie kąta). Blender: `/Applications/Blender.app/Contents/MacOS/Blender -b -P build_lindor.py -- ...`.
- 4 stare GLB: `lindor-czerwony.glb`, `lindor-czarny.glb`, `lindor-brazowy.glb`, `lindor-zolty.glb`
- `TEKSTURY/` — płaskie skany folii (1575×950 JPG): Lindor-czerwony/czarny/brazowy/zolty
- `TEKSTURY/REFERENCJE/` — zdjęcia produktowe
- `PODGLAD/` — rendery + kopie GLB

## Czym są
Czekoladowe **kule owinięte folią** z owalnym logo Lindt LINDOR z przodu, BEZ przekręconych ogonków celofanu. 4 kolory. Game-ready, najlepsza jakość w tej opcji. Użyte w [[project-pickmix]].

## Pipeline (Blender 5.1.2 headless + PIL + gltf-transform)
1. **Tekstura (PIL), decal:** ze skanu folii buduję equirect **1024×512** = kafelkowane tło wzoru folii (lustrzane) + JEDNO owalne logo wklejone na środek (miękka maska prostokątna). Logo MUSI być wycentrowane w teksturze → trafia na front kuli.
2. **„Richer":** ImageEnhance Color×1.38, Contrast×1.12, blend z multiply 0.40 (gasi wypalone biele, pogłębia kolor), Brightness×1.06.
3. **Normal mapa:** z jasności tekstury (Sobel) → relief pomięcia folii.
4. **Blender (`/tmp/build_lindor.py`):** UV sphere segs 64 / rings 48 / r=0.5, shade smooth, lekki displace (CLOUDS, strength 0.012); materiał Principled: baseColor=tekstura, normal=mapa (strength 0.5), **metalness 0.7, roughness 0.40**; obrót kuli wokół Z o **248°** (bo logo wycentrowane w teksturze trafia na front przy tym kącie — znalezione „sweepem"); świat + 3 area lights miękkie; render EEVEE Next; eksport **GLB** (`export_yup=True`, `export_apply=True`). Wynik: średnica 1, origin na środku, +Y up, ~6k tri, tekstury 1024.
5. **gltf-transform** (`npx @gltf-transform/cli resize --width N --height N`) — do zmniejszania tekstur w GLB (użyte też w arkanoidzie: 4096→512).

## Workflow podmiany tekstury (ustalone z Michałem)
Michał może podkręcić **finalną teksturę** `PODGLAD/<kolor>_tekstura.png` (1024×512, logo na środku) w Photoshopie i oddać — ja podmieniam 1:1, regeneruję normal + GLB w sekundy. Albo podkręca surowy skan, ja przepuszczam przez pipeline.

## Stan / do poprawy później
- Brązowa: logo lekko przesunięte w prawo.
- Czarna/brązowa: ślad bocznego napisu ze skanu („60% CACAO"/„HAZELNUT") obok logo — do docięcia.
- Materiał metaliczny → w grze potrzebny **environment map** (PMREM/RoomEnvironment), inaczej folia wyjdzie matowa.
