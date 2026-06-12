---
name: project-pinball
description: "Projekt PIN BALL — gra labirynt 3D z czekoladą Lindt, stan gry i otwarte kwestie"
metadata: 
  node_type: memory
  type: project
  originSessionId: 64536386-5f9d-4c18-b9fc-52f4927e5a25
---

# Projekt PIN BALL — Labirynt 3D Lindt

## Cel
B2B pitch demo — interaktywna gra 3D w labirynt, kulka toczy się po czekoladowym labiryncie (`chocolate2.glb`, 71MB model). Planowane jako widget embeddowalny na stronach klientów.

**Why:** walidacja pomysłu "pinball z layoutu strony" jako produkt B2B.
**How to apply:** decyzje techniczne podporządkowane szybkiemu demo, nie perfekcji.

## Główny plik gry
`/Users/michal/CLAUDE PROJECTS/PIN BALL/maze.html` — all-in-one, Three.js + ES modules, PerspectiveCamera top-down.

## Stack techniczny maze.html
- Three.js r0.158.0 (via importmap CDN)
- GLTFLoader → `chocolate2.glb`
- PerspectiveCamera(FOV=32°) — widok z góry, wysokość Y=14, naturalny foreshortening przy tiltowaniu
- `three-mesh-bvh` v0.6.8 — BVH dla raycastingu (O(log n))
- Pre-baked 2D grid kolizji (GCOLS=64, GROWS=28 = 1792 raycastów przy starcie)
- Sterowanie: strzałki klawiatury (tilt maze); mysz WYŁĄCZONA

## Kluczowe stałe (maze.html)
```
MAZE_SCALE = 12
BR = 0.20          // promień kulki
MX = 6.0           // połowa szerokości w world units
MZ = 2.4           // połowa głębokości
GCOLS = 64, GROWS = 28
GRAVITY = 18
FRICTION = 0.965
RESTITUTION = 0.20
MAX_TILT = PI/7
TILT_STEP = 0.014
TILT_LERP = 0.10
BASE_TILT_X = -0.52360   // -30° — korekcja orientacji eksportu GLB
BASE_ROT_Y  =  3.14159   // 180° — obrót labiryntu do właściwej pozycji startowej
```

## Kamera
```javascript
const camera = new THREE.PerspectiveCamera(32, W / H, 0.1, 60);
camera.position.set(0, 14.0, 0);
camera.rotation.set(-Math.PI / 2, 0, 0);
```

## Hierarchia modelu
```javascript
scene → mazeGroup (rotation.x = tiltX, rotation.z = tiltZ) 
      → glbModel (rotation.y = PI/2 + BASE_ROT_Y, scale = MAZE_SCALE)
```

## System kolizji — siatka 2D (WAŻNE)

### Budowanie siatki
Siatka budowana raz przy starcie, gdy `mazeGroup.rotation=(0,0,0)`:
- 1792 raycasty z góry (GCOLS×GROWS)
- hitY = Y punkt trafienia w local space mazeGroup

### Klasyfikacja komórek — BAND THRESHOLD
```javascript
const range = maxHitY - rawFloor;
const wallEps = range * 0.02;      // top 2% = ściany
const channelBand = range * 0.40;  // 40% poniżej max = kanały
upperBound = maxHitY - wallEps;
lowerBound = maxHitY - channelBand;
// passable = hitY ∈ [lowerBound, upperBound]
```

### Filtr sąsiedztwa — LOKALNE MAKSIMA (poprawiona wersja 2026-05-27)
```javascript
// Usuwa TYLKO komórki będące lokalnym maksimum (wyższe od średniej sąsiadów).
// Szczyty ścian = lokalne maksima → usuwane.
// Narożniki kanałów = tej samej lub niższej wysokości co sąsiedzi → ZOSTAJĄ.
if (nCnt > 0 && hy > nSum / nCnt) {
  gridData[gIdx(xi, zi)] = 0; // lokalne maksimum → szczyt ściany
}
```

**KRYTYCZNE:** Poprzedni filtr (neighborDelta=0.02, "musi być o 0.02 niżej od sąsiadów")
usuwał narożniki korytarzy (L-zakręty) bo były tej samej wysokości co sąsiednie komórki
kanału (delta≈0). Powodowało to, że flood-fill tworzył izolowane proste odcinki bez
możliwości skrętu — kulka odbijała się na końcu odcinka i nie mogła wyjść.

### Flood-fill (po filtrze sąsiedztwa)
```javascript
// Zachowaj tylko największy spójny komponent — eliminuje izolowane kieszenie.
// BFS 4-kierunkowy, kodowanie kolejki: xi * GROWS + zi
gridData.fill(0);
for (const i of bestComp) gridData[i] = 1;
console.log('Flood-fill: largest component =', bestSize, 'cells');
```

### Border exclusion
```javascript
RIM = 3; // Zewnętrzne 3 komórki = ściany (outer rim tabliczki)
```

### Wynik dla obecnego modelu
- band: [0.48, 1.47]
- cells: ~183 passable (przed flood-fill)
- Kanały labiryntu są TYLKO w obszarze z<0 (górna połowa local space)
- Obszar z>0 to gładka spodnia czekolady — wygląda jak rowki z przechylonej kamery ale nie jest nawigowalne

### canMoveTo — tylko środek (poprawiona wersja 2026-05-27)
```javascript
function canMoveTo(x, z) {
  return gridPassable(x, z);
}
```

**KRYTYCZNE:** Poprzedni kod sprawdzał 5 punktów (środek + 4 przesunięcia r=0.06).
Na zakrętach korytarza przesunięcia prostopadle do kierunku ruchu trafiały w ściany
sąsiednich cel, blokując skręt. Przy cell width≈0.190 i maks. przesunięciu/substep≈0.065
tunelowanie niemożliwe, więc sprawdzanie tylko środka jest bezpieczne.

### Per-cell floor Y dla kulki
```javascript
function cellFloorY(x, z) { ... return hitYsGrid[cell] lub floorY jako fallback }
// Ball Y = cellFloorY(ball.x, ball.z) + BR
```

### Substepy fizyki (3 kroki)
```javascript
const SUBSTEPS = 3; // zapobiega tunelowaniu przez cienkie ścianki
```

## Grawitacja/fizyka
```javascript
const gX = GRAVITY * (-Math.sin(tiltZ));
const gZ = GRAVITY * (Math.sin(tiltX - BASE_TILT_X));
// BASE_TILT_X jako offset — przy pozycji startowej gZ=0 (kulka nieruchoma)
```

## Pozycja startowa i meta
- Start: `findPassablePos(RIM, GCOLS-1-RIM, RIM, GROWS-1-RIM)` — pierwsza passable komórka w interior
- Meta: komórka passable najdalej (Manhattan) od startu

## Stan po sesji 2026-05-29 (piąta sesja)

### game.html — stan końcowy
- Screeny win/over przetłumaczone na angielski: "Well done!", "Better luck next time!", "Play again", "Close", "Score: X pts"
- Screeny win/over: tło pełne `rgb(38,20,8)` (usunięto `rgba` + `backdrop-filter: blur`) — gra nie przebija przez overlay
- Ekrany: scr-ready, scr-win, scr-over — wszystkie opaque, bez bleed-through

### Otwarte zadania game.html
- Deploy game.html na VPS (aktualizacja plików)

---

## Stan po sesji 2026-05-28 (wieczór — czwarta sesja)

### Co działa ✓
- Kulka leży na powierzchni rowka (per-cell floor Y)
- Kulka nieruchoma na starcie gdy tabliczka płasko
- Kulka rusza się przy przechylaniu
- Kulka blokowana przez ścianki labiryntu
- Perspektywa poprawna
- Ekrany start/win/restart działają
- Start: lewy górny korytarz (auto, potwierdzone przez Michała jako OK)
- **Meta: auto-detekcja dziury w prawej ściance** (geometryczna przerwa w wall mesh)
- Debug overlay: zielone=passable, pomarańczowy kwadrat=wykryta dziura

### Mechanika dziury (nowa, 2026-05-28)
- RIM exclusion nie obejmuje prawej strony — passable cells przy xi >= GCOLS-RIM to dziura
- Win: `ball.x > holeX - 0.35 && |ball.z - holeZ| < 0.9`
- `SHOW_DEBUG_GRID = true` — do wyłączenia przed deployem
- `MANUAL_START` / `MANUAL_GOAL` w constants = overrides ręczne (null = auto)

### Otwarte zadania
1. Weryfikacja: czy dziura auto-detekuje się w miejscu żółtego kółka (prawa ścianka, dolny prawy obszar)?
2. Test: czy kulka może dotoczyć się do dziury i triggeruje win?
3. Tune fizyki (GRAVITY / FRICTION / MAX_TILT)
4. Mysz (delta-based) — nadal wyłączona
5. Wyłączyć `SHOW_DEBUG_GRID` przed deployem
6. Deploy maze.html na VPS

## Deploy
- Subdomena maze3d: `https://lds-lindt-maze3d.mdmresearch.com` ✅ (SSL, wdrożone 2026-06-01)
- VPS: kontener `pinball-demo` na porcie 8088, pliki w `/opt/pinball-demo/`
- Index: `index.html` = kopia `maze.html`
- GitHub: `github.com/Ikatrefon/pin_ball` (razem z game.html w jednym repo) ✅

## Historia debugowania kolizji (skrót)
1. Ball sinks → per-cell floorY
2. Ball doesn't move → r=0.06 buffer w canMoveTo blokowało zakręty → usunięto, tylko środek
3. Ball at (0,0) wall → findPassablePos szuka po całym interior
4. Wrong passable area (outer rim) → band threshold zamiast min/max threshold
5. Ball on wall tops → neighbor filter (pierwotnie delta=0.02) + flood-fill
6. **Neighbor filter usuwał narożniki** (delta≈0 bo kanał tej samej wysokości co sąsiedni kanał)
   → zmieniono na filtr lokalnych maksimów (hy > avg_neighbor)
