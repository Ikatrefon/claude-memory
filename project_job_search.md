---
name: project-job-search
description: "JOB SEARCH — automat dopasowujący CV pod ogłoszenia (UE). Półautomat, człowiek = bramka. Najpierw dla Michała (realne szukanie pracy), potem może komercyjnie."
metadata: 
  node_type: memory
  type: project
  originSessionId: 42ecb71f-295d-47b5-b01f-77be0ea7d00f
---

# Projekt: JOB SEARCH (automat CV)

## Lokalizacja
`/Users/michal/CLAUDE PROJECTS/JOB SEARCH/`
- `specyfikacja_cv_automat.md` — zamknięta specyfikacja v1.0 (przeczytać na start).
- `Michal_Duchowski_CV.pdf` — bazowe CV (3 strony A4).
- `template/` — szablon #1 (wierne odwzorowanie bazowego CV).

## Cel i zasady (z rozmowy 2026-06-27)
- Z bazowego CV + ogłoszenia → **dopasowane CV w PDF** do akceptacji. System **nie wysyła** aplikacji (człowiek = bramka, założenie projektowe).
- **Obszar: cała UE** (pozwolenie na pracę). **CV po angielsku** (jeden język — bez tłumaczeń). **Branże wynikają z treści CV** (nie z konfiguracji).
- **Zasada nadrzędna:** warstwa źródeł odcięta od silnika (ogłoszenie → wspólny format JSON). MVP = kanał ręczny (wklejanie); Adzuna i scrapery dokładane później bez ruszania silnika.
- **Guardrail:** generacja przestawia/akcentuje PRAWDZIWĄ treść, NIE dopisuje kompetencji. Plan: bazowe CV jako dane strukturalne + przebieg weryfikujący + podświetlanie zmyśleń w poczekalni.
- **Cel biznesowy:** najpierw działa dla Michała (dogfooding na realnym szukaniu pracy), potem ew. komercja. Teraz BEZ multi-user/logowania (YAGNI) — architektura i tak pozwoli na przeskok później.

## Stos (rekomendacja)
Python + FastAPI(+Jinja2+HTMX) + SQLite + API Anthropic (ocena tani model/Haiku, generacja Sonnet/Opus) + prompt caching bazowego CV. PDF: HTML/CSS→PDF.
**Renderer PDF — decyzja do rewizji:** na razie **Chromium (Playwright)** dla pełnej wierności CSS bogatego szablonu (już dostępny). WeasyPrint było pierwotną rekomendacją (lżejszy na VPS) — wrócić, jeśli szablony zostaną proste.

## Szablon #1 — stan (2026-06-27, DONE wstępnie)
Wierne odwzorowanie bazowego CV w `template/`:
- `cv.json` — strukturalne CV (źródło prawdy): name, badge, contact, profile, itsme[], additional[], experience[8], education, about_me/interests/tech_skills/quotes, signature.
- `template.html` — Jinja2 + wbudowany CSS (3 strony A4, monochrom czerń/szarość + logotypy w kolorze, zdjęcie grayscale).
- `assets/` — logotypy wyciągnięte z PDF (PyMuPDF): assembly, publicis, capgemini, accenture, ikea, grey, dwell, pjatk(pieczątka), swps(statek), photo. **GEMACO bez logo** → renderowany jako tekst (do ew. dorobienia).
- `render` pipeline: `python3` (jinja2: cv.json+template.html→cv.html) → node Playwright (`/tmp/render_cv.mjs`, file:// → cv.pdf, A4, printBackground, preferCSSPageSize).
- **Fonty oryginału:** Arial / Arial Narrow / Verdana (standard, bez licencji). Nagłówki/nazwisko = Arial Narrow Bold; na VPS doinstalować odpowiednik (Liberation/Arimo Narrow).
- Wierność ~90%, potwierdzona porównaniem obok oryginału (3 strony). Drobne do dociągnięcia: logo GEMACO, ikonki przy kafelkach edukacji, dokładny krój podpisu (teraz cursive systemowy).

## Narzędzia (Mac, do renderu/ekstrakcji)
- PyMuPDF (`pip install pymupdf`) — render PDF→PNG, ekstrakcja obrazów/tekstu/coords (bez poppler).
- Jinja2, Playwright (node w /tmp — bywa czyszczony, reinstall `npm i playwright`; chromium z cache executablePath chromium_headless_shell-1217).

## Stan budowy
Specyfikacja: etap 1 (MVP) = kanał ręczny→ocena→generacja(guardrail)→PDF z szablonu→poczekalnia. NIE zaczęty (poza szablonem #1). Następny krok: rozpisać/zbudować MVP silnika.
