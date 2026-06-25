---
name: feedback-lindt-font
description: Projekty Lindt używają fontu Marcellus (Google Fonts) zamiast Georgia
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

We wszystkich projektach dla Lindta używaj fontu **Marcellus** z Google Fonts zamiast Georgia.

```html
<link href="https://fonts.googleapis.com/css2?family=Marcellus&display=swap" rel="stylesheet">
```

```css
font-family: 'Marcellus', serif;
```

**Why:** preferencja Michała — Marcellus lepiej pasuje do estetyki marki Lindt.
**How to apply:** przy każdym nowym elemencie tekstowym w projektach Lindt (game.html, maze.html, webcam/index.html i kolejne).

## WYJĄTEK: Gift Catcher → Optima (2026-06-25)
W [[project-gift-catcher]] Michał poprosił o **Optimę** zamiast Marcellus. Optima NIE jest w Google Fonts (komercyjny Linotype) — Michał dał plik `Optima (1).ttc` (kolekcja macOS). Self-hostowane: wyodrębniłem Regular+Bold przez `fontTools` (`TTCollection`, `flavor='woff2'`, `.save()`) → `ELEMENTY/optima-{regular,bold}.woff2` (~20 KB każdy), `@font-face` 400/700. Nie zmienia globalnej reguły Marcellus dla innych projektów — chyba że Michał powie inaczej.
