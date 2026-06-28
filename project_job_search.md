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

## Silnik MVP — DZIAŁA (2026-06-27, etap 1 end-to-end)
Aplikacja w `app/` (FastAPI). Przetestowana w trybie mock end-to-end (paste→ocena→auto-gen≥próg→guardrail→PDF→poczekalnia, akcje, edit, regenerate force, threshold).
- **Uruchomienie:** `cd "JOB SEARCH" && export ANTHROPIC_API_KEY=... && app/.venv/bin/uvicorn app.main:app --reload --port 8200`. Bez klucza = **tryb MOCK** (offline, deterministyczny — `USE_MOCK` w config gdy brak klucza).
- **venv:** `app/.venv` (fastapi, uvicorn, anthropic, jinja2, python-multipart, playwright+chromium). UWAGA: PyMuPDF/fitz jest w SYSTEMOWYM python3 (--user), NIE w venv — do rasteryzacji PDF użyj `python3`, nie venv.
- **Pliki:** `app/config.py` (modele: eval=claude-haiku-4-5-20251001, gen=claude-sonnet-4-6; próg domyślny 60), `db.py` (SQLite: ogloszenia/cv_wygenerowane/konfiguracja), `engine.py` (evaluate+generate+guardrail; Anthropic z cache bazowego CV w bloku system; mock fallback), `pdf.py` (Playwright sync → template/template.html → A4 PDF), `main.py` (routes: / , /paste, /ad/{id}, /pdf/{id}, /ad/{id}/status|regenerate|edit, /config), `templates/` (Tailwind CDN, schludny UI).
- **Guardrail (twardy):** wymusza name/contact/badge/education z bazy; doświadczenie tylko firmy z bazy (firma/rola/daty z bazy, brakujące dołącza), tech_skills ⊆ baza; flaguje bullety z <45% pokrycia słów z bazowymi (możliwe dopiski). Zwraca warnings[] pokazywane w detalu.
- **Generacja** zwraca pełne cv (schemat jak cv.json) → renderowane tym samym szablonem #1. Mock: przestawia `itsme` wg pokrycia z ogłoszeniem + prefiks profilu.
- Render PDF działa, dopasowane CV wygląda jak bazowe (potwierdzone). Podgląd w iframe pusty w chromium-headless-shell (brak wtyczki PDF) — w realnej przeglądarce OK; pobieranie PDF zwraca 200.
- README.md w projekcie z instrukcją.
- **Klucz API:** wczytywany z `JOB SEARCH/.env` (`ANTHROPIC_API_KEY=...`), config czyta `.env` bez zależności; `.env` w `.gitignore`. Michał wkleił klucz w czacie 2026-06-27 → ZALECONA ROTACJA (sprawdzić czy zrobił).
- **LIVE PRZETESTOWANE (2026-06-27):** realne Claude działa end-to-end. Przykład: ad „Head of Design (UX/UI), FinTech Berlin" → ocena Haiku score 78 + trafne summary/gaps; generacja Sonnet przepisała profil + przestawiła itsme pod rolę, guardrail bez ostrzeżeń (zero zmyśleń), uzasadnienie uczciwie wskazało lukę (brak fintechu); PDF wyrenderowany wiernym szablonem. Jakość ocen/generacji bardzo dobra out-of-the-box.
- **NIE zrobione:** Etap 2 (Adzuna+cron+dociąganie opisu), bogatsza edycja w „popraw" (teraz tylko podsumowanie), mocniejszy guardrail (weryfikacja LLM + podświetlanie zmian), ew. WeasyPrint zamiast Chromium na VPS.

## Wersja mobilna + deploy prep (2026-06-28)
- **Decyzja Michała:** to ma być aplikacja webowa → potrzebna **domena** + **wersja mobilna**. Testy jakości później.
- **Mobile:** UI już responsywny (Tailwind mobile-first — sekcje/przyciski stackują się w pionie). Drobne poprawki: pola formularza `grid-cols-1 sm:grid-cols-2`; podgląd PDF w iframe ukryty na mobile (`hidden sm:block`) + przycisk „Otwórz podgląd CV (PDF)" na małych ekranach. Potwierdzone zrzutami 390px (index + detail OK).
- **Deploy artefakty (gotowe, w repo projektu):** `Dockerfile` (python:3.12-slim + **msttcorefonts=prawdziwy Arial/Arial Narrow** dla wierności CV + `playwright install --with-deps chromium`), `requirements.txt`, `.dockerignore` (UWAGA: NIE wykluczać `template/assets/*.png` — logotypy muszą wejść do obrazu; wyklucza tylko cv.pdf/oryginał/zrzuty/.env/.venv/data).
- Runtime: `ANTHROPIC_API_KEY` przez `-e` (nie w obrazie), dane trwałe przez wolumen na `/srv/app/data`, port 8000.
## WDROŻONE NA VPS (2026-06-28) — https://job.mdmresearch.com
Działa publicznie (200, http→https, cert Let's Encrypt do 26.09.2026). Mobile OK.
- **DNS:** `job.mdmresearch.com` A → 195.35.56.37 (Michał dodał, TTL 300).
- **Kontener:** `jobsearch` (obraz `jobsearch:latest` ~1.95GB), `-p 8110:8000`, `--env-file /opt/jobsearch/.env` (klucz, root-only, NIE w obrazie), `-v /opt/jobsearch/data:/srv/app/data` (sqlite+pdfy trwałe), `--restart unless-stopped`. Kod na VPS w `/opt/jobsearch/`.
- **Proxy:** publiczny = **dockerowy `workload_nginx`** (system nginx nieaktywny). Dodany blok w `/opt/workload-estimator/workload-estimator/nginx/nginx.conf` (80 redirect + 443 ssl → `http://195.35.56.37:8110`, `proxy_read/send_timeout 180s` bo Claude+PDF wolne). Backup: `nginx.conf.bak.job`. Reload: `docker exec workload_nginx nginx -t && nginx -s reload`.
- **Cert:** certbot standalone (krótki `docker stop workload_nginx` → certbot → start), certy skopiowane do `nginx/certs/job/{fullchain,privkey}.pem`. Live: `/etc/letsencrypt/live/job.mdmresearch.com/` (auto-renew ustawiony przez certbot).
- **Port 8110** dopisany do zajętych (patrz [[infrastructure]]).
- **Procedura aktualizacji:** `rsync app template → /opt/jobsearch/` → `cd /opt/jobsearch && docker build -t jobsearch:latest .` (warstwy fonty/chromium/pip cache'owane → szybko) → `docker rm -f jobsearch && docker run -d ...` (jak wyżej).
- **Fix wdrożeniowy:** starlette w obrazie wymaga `templates.TemplateResponse(request, name, context)` (stara sygnatura `(name,{...})` → „unhashable type: dict"). Naprawione.
- **Klucz API zrotowany 2026-06-28** (Michał wymienił w `/opt/jobsearch/.env` + `docker restart jobsearch`), zweryfikowany na żywo (ocena działa). Stary (z czatu) skasowany.
- **Rozdzielenie ocena/generacja (2026-06-28):** `/paste` robi TYLKO ocenę (`_evaluate_ad`, ~4 s) → redirect do detalu (od razu score/summary/gaps). Generacja CV (`_generate_cv`, ~30-50 s) na żądanie: przycisk „Generuj dopasowane CV" → `POST /ad/{id}/generate`. Overlay ze spinnerem na obu (index: „Oceniam…", detail: „Generuję…"). Powód: paste „wisiał" ~52 s bez feedbacku → user myślał że nie działa. Endpoint `/regenerate` zmieniony na `/generate`.
- **GOTCHA docker+env:** `docker restart` NIE czyta na nowo `--env-file`! Po zmianie `/opt/jobsearch/.env` trzeba `docker rm -f jobsearch && docker run ... --env-file ...` (recreate), nie restart.
- **Naprawa .env (2026-06-28):** edycja w nano przez Michała uszkodziła plik — nowy klucz trafił PRZED nazwę zmiennej (`<NOWY>ANTHROPIC_API_KEY=<STARY>`) → brak poprawnej zmiennej → MOCK. Naprawione serwerowo (wyciągnięcie nowego klucza sprzed „ANTHROPIC_API_KEY=", zapis `ANTHROPIC_API_KEY=<nowy>`), recreate kontenera. Zweryfikowane: real Claude działa (ocena 4 s, score 78). Backup uszkodzonego: `.env.broken.bak` (zawiera oba klucze — stary już dead, ew. usunąć).
- **Czytelna ocena (2026-06-28):** `evaluate()` zwraca strukturę: `score`, `wants[]` (priorytety pracodawcy), `gaps={missing[],tune[]}` — po POLSKU (dla Michała). W db jako JSON w polach summary/gaps (wstecznie zgodne: stare wpisy tekstowe renderują się jako tekst). Detal: osobne karty „Czego chce ogłoszenie" (bullety) i „Luki vs bazowe CV" rozbite na „Brakuje (nie ma w CV)" + „Warto podkręcić". Zamiast 3 zlewających się bloków.
- **Basic Auth (2026-06-28):** strona za hasłem (login `michal`). Konfiguracja w bloku `job` nginx: `auth_basic` + `auth_basic_user_file /etc/nginx/certs/job/.htpasswd` (plik na host: `.../nginx/certs/job/.htpasswd`, format `user:apr1-hash`). Zmiana hasła: `printf 'michal:%s\n' "$(openssl passwd -apr1 'NOWE')" > .../nginx/certs/job/.htpasswd` → `docker exec workload_nginx nginx -s reload`.
