---
name: project-job-search
description: "JOB SEARCH — automat dopasowujący CV pod ogłoszenia (UE). Półautomat, człowiek = bramka. Najpierw dla Michała (realne szukanie pracy), potem może komercyjnie."
metadata: 
  node_type: memory
  type: project
  originSessionId: 42ecb71f-295d-47b5-b01f-77be0ea7d00f
---

# Projekt: JOB SEARCH (automat CV)

## Praca nad jakością graficzną CV — strona 1 nagłówek (2026-06-28)
Problem: generowane CV „rozjeżdżało się" (kolizje nazwisko/badge, tag na tekst) — bo nagłówek był układem płynnym (flex) zależnym od treści.
- **Rozwiązanie: nagłówek strony 1 pozycjonowany ABSOLUTNIE w `pt`** = 1:1 z oryginałem (CSS pt == PDF pt na A4). Klasy `.p1-name1/.p1-name2/.p1-photo/.p1-badge/.p1-contact` + `.p1-body{margin-top:178pt}` (treść schodzi pod nagłówek). Współrzędne odczytane z oryginału przez PyMuPDF (`get_text('dict')` bbox, `get_image_info`, `get_drawings`). Nazwisko 75pt, x33, top 29/93pt; zdjęcie left427.7 top35.4 w130.8pt; badge left360 top46; kontakt left33 top182 14pt.
- **Odkrycie:** w oryginalnym PDF imię/nazwisko + badge są ZWEKTORYZOWANE (krzywe, nie tekst) → oryginał sam nie był czytelny dla ATS w polu nazwiska. My robimy je TEKSTEM (ten sam wygląd + ATS czyta).
- **Font:** oryginał = Arial Narrow Bold. msttcorefonts (VPS) NIE zawiera Arial Narrow, Liberation Sans Narrow trudno dostać. **Dołączyłem pliki `template/fonts/ArialNarrow.ttf` + `-Bold.ttf`** (z Maca) przez `@font-face` family „ArialN" → identycznie lokalnie i na VPS. UWAGA LICENCJA: Arial Narrow to font Microsoftu — OK do prywatnego użytku (CV Michała), ale przy komercji/multi-user PODMIENIĆ na otwarty metryczny odpowiednik (Liberation Sans Narrow) albo wykupić licencję.
- Weryfikacja: render lokalny (node Playwright, /tmp/render_cv.mjs) → nakładka/side-by-side na oryginał (PyMuPDF), iteracja rozmiaru/pozycji.
- **DECYZJA (2026-06-28): nagłówek str.1 = gotowy SVG od Michała** (`template/assets/top.svg`, zawiera nazwisko+badge+łuk+zdjęcie+mail+telefon). Moja rekonstrukcja tekstem „to nadal nie było to" — Michał robi nagłówek w Figmie i wgrywa do `ELEMENTY/`; ja kopiuję do `template/assets/top.svg`.
- **KLUCZOWE — SVG osadzany INLINE, nie `<img>`:** `<img src=svg>` rasteryzuje się w PDF → tekst NIEzaznaczalny (ATS nie czyta) i font @font-face nie działa. Dlatego: `pdf.py` wczytuje `top.svg` i wstrzykuje do szablonu jako `{{ top_svg|safe }}` w `<div class="p1-top">` (`left:34pt top:24pt width:527pt`; `.p1-top svg{width:100%;height:auto}`). Lokalny render: helper `/tmp/render_local.py` (mirror pdf.py, też wstrzykuje top_svg) → `node /tmp/render_cv.mjs`. **Weryfikacja ATS:** `fitz ...get_text()` — email/MICHAL/DUCHOWSKI MUSZĄ być w tekście PDF (są).
- **SVG musi mieć TEKST, nie krzywe:** przy eksporcie z Figmy ODKLIKNĄĆ „Outline text" (inaczej `<text>`→`<path>`, ATS nie czyta). Sprawdzenie: `grep -c "<text" top.svg` > 0.
- **Fonty z SVG muszą być dostępne dla Chromium:** SVG używa **Antonio** (nazwisko, Google OFL — dołączony `template/fonts/Antonio.ttf` wariantowy, `@font-face "Antonio" weight 100-700`), **Arial Black** (badge) i **Arial** (kontakt) — te dwa są na VPS (msttcorefonts: ariblk.ttf) i Macu. Embedded Arial Narrow (ArialN) zostaje dla reszty szablonu.
- **Wzorzec na przyszłe sekcje:** Michał dostarcza ozdobne elementy jako SVG (z TEKSTEM, bez outline) do `ELEMENTY/` → kopiuję do `template/assets/`, wstrzykuję INLINE, dbam by użyte fonty były dostępne (Google→osadź @font-face; MS→msttcorefonts).

## Strona 2 — głowy prac jako SVG (2026-06-28)
- Usunięty `aside` PJATK/„Lecturer faculty of UX Design" z ASSEMBLY w cv.json (info jest na str.1). Zostało 5 prac na str.2: ASSEMBLY, Publicis Le Pont, Capgemini, accenture, IKEA (`experience[:5]`); str.3 = `experience[5:]` (GEMACO, d-well, GREY — stary układ logo+rola+daty).
- **Każda praca str.2: „głowa" = inline SVG od Michała** (logo lewo + stanowisko + daty prawo) + pod spodem `<ul class="bul">` punkty **12pt Arial Narrow** (to jest część dopasowywana pod ofertę; stanowisko/daty/logo stałe siedzą w SVG).
- Pliki: `ELEMENTY/{Assembly,Publicis production,CapGemini,Accenture,IKEA}.svg` → kopiowane do `template/assets/job_{assembly,publicis,capgemini,accenture,ikea}.svg`. `pdf.py` ma `HEAD_SVG = {firma: plik}` i `_inject_head_svgs()` → wstrzykuje treść jako `e["head_svg"]`; szablon: `{% if j.head_svg %}<div class="job2-head">{{ j.head_svg|safe }}</div>`. (Mapowanie po nazwie firmy, bo guardrail nie zachowuje dodatkowych pól.)
- SVG-i używają font-family **„Arial Narrow"** → dodany alias `@font-face "Arial Narrow"` (400/700) na pliki ArialNarrow.ttf (na VPS nie ma Arial Narrow).
- `.job2-head { width:527pt; overflow:hidden }` + `svg{width:100%}` → skaluje każdy SVG do kolumny → daty wyrównane do prawej, loga do lewej. **Szerokość bloku/kolumny = 527pt = 186mm ≈ 703px** (A4 210mm − 2×12mm marginesu). Michał eksportuje SVG-y na stałej ramce **527** (1 jednostka=1pt → brak przeskalowania, fonty 14pt spójne). Wielkość logo wewnątrz SVG = jego decyzja (wymienia plik gdy chce inną).
- Odstępy str.2: `.h-big{margin-bottom:16pt}` (pod EMPLOYMENT HISTORY), `.job2{margin-bottom:18pt}` (między blokami). Mieści się 5 prac (y≈805, dolny margines ~814) — przy długich tekstach pod ofertę wróci temat objętości.
- **GOTCHA „duch" logo z Figmy → ROZWIĄZANE AUTOMATEM (2026-06-29):** Figma eksportuje logo jako `<rect fill="url(#pattern)">`; Chromium źle renderuje `pattern` w PDF (powiela kafelek = „duch"). **`pdf.py` ma teraz `depattern(svg)` wołane na KAŻDYM wstrzykiwanym SVG** (top/heads/edu) — zamienia każdy taki rect na zagnieżdżony `<svg viewport>` + `<image>` (viewport KADRUJE jak pattern; pozycja/skala z transformu use). Obsługuje `transform` w stylu `matrix()` ORAZ `scale()`/`translate()` (Figma używa obu! — pierwsza wersja łapała tylko matrix → gigantyczny obraz/puste głowy). Pomija rotację/skos (b/c≠0). Dzięki temu re-eksporty z Figmy są odporne — pliki na dysku zostają oryginalne (accenture przywrócony do oryginału). Ta sama logika zduplikowana w `/tmp/render_local.py`.

## Strona 3 — GOTOWE (2026-06-29)
Zbudowana, wdrożona (live 200). Struktura: **3 prace** (GEMACO, d-well, GREY = `experience[5:]`) jak str.2 — SVG głowa (527) + punkty pod spodem. **Edukacja** = inline `edu.svg` (statyczny, wstrzykiwany jako `edu_svg` w pdf.py/render_local). **4 kolumny** about me / my interests / tech skills / quotes (`.about` grid 4-kol, Arial Narrow 10pt, col-h 11pt). Ozdobniki „…earlier"/„…and this is not over" zostają. Podpis usunięty.
- Pliki: `ELEMENTY/{Gemaco,d-well,GREY,EDUCATION}.svg` → `template/assets/{job_gemaco,job_dwell,job_grey,edu}.svg`. HEAD_SVG rozszerzone o GEMACO/d-well/GREY.
- **Redukcja punktów starych prac BEZ kasowania z cv.json:** szablon pokazuje `j.bullets[:3]` (pełne dane zostają w cv.json = lepszy materiał do tailoringu; po generacji [:3] = 3 najtrafniejsze, bo generator przestawia). Klasy `.job3` (margin 9pt, bullety 11pt — ciaśniej niż str.2). Mieści się (y≈767, margines ~814).
- **Dwa tory ( wciąż DO ZROBIENIA):** wersja GRAFICZNA (ten szablon) vs ATS-friendly (jedna kolumna, czysty tekst) z tego samego cv.json.
- **OTWARTE/do ew. dopieszczenia:** „…and this is not over" (`.notover` absolutnie pozycjonowany bottom:92mm) — sprawdzić czy widoczny/nie nachodzi; logo edukacji „Kozminski University" pojawia się 2× (tak w SVG Michała). Cały szablon 3 stron = DONE wizualnie.

## Pipeline renderu/podglądu (stan 2026-06-29)
- Lokalny render: `/tmp/render_local.py` (mirror `pdf.py`: wczytuje cv.json + top.svg + wstrzykuje head_svg per firma wg mapy HEAD) → pisze `template/cv.html` → `node /tmp/render_cv.mjs` (chromium headless, executablePath z `~/Library/Caches/ms-playwright/chromium_headless_shell-1217/...`) → `template/cv.pdf`. Podglądy PNG (200dpi) do `JOB SEARCH/podglad/CV_strona_{1,2,3}.png` (Michał ogląda dwuklikiem). Weryfikacja dopasowania: rasteryzacja fitz + side-by-side/overlay na oryginał; weryfikacja ATS: `n[i].get_text()`.
- **Deploy każdej zmiany:** rsync zmienionych plików do `/opt/jobsearch/{template,app}/...` (z poprawnym /app/!) → `docker build` + `docker rm -f` + `docker run` (NIE restart — env-file czyta się tylko przy run) → curl localhost:8110 → commit+push (`git`, klucz `~/.ssh/id_ed25519`). Live: https://job.mdmresearch.com (port 8110, Basic Auth michal / T3YKB95nMdoH).

## Menedżer dokumentów kandydata (2026-06-28)
W Ustawieniach sekcja „Dokumenty kandydata": bazowe CV (`cv.json`) = rdzeń szablonu (stałe), + dodatkowe dokumenty (PDF/TXT/MD) dodaj/usuń. Tekst dokumentów (`engine.extra_context()` z sidecarów `data/docs/<id>.txt`) dołączany do system-promptu OCENY i GENERACJI (więcej prawdziwego materiału, bez zmyślania) ORAZ do „zbioru prawdy" guardraila (tokeny dokumentów dodane do base_tok → uzasadnione dopiski nie flagowane). Tabela `dokumenty(id,created_at,orig,fname)`; pliki w `app/data/docs/` (wolumen). Routes: `POST /docs/upload` (UploadFile), `POST /docs/{id}/delete`. Ekstrakcja PDF przez **PyMuPDF** (dodane do requirements). Pełna podmiana bazowego CV (nowy layout) = osobny, większy temat (PDF→struktura→szablon) — NIE zrobione.
- **GOTCHA rsync:** pliki app/ deployować do `/opt/jobsearch/app/` (z `/app/`!), nie do `/opt/jobsearch/` — inaczej Docker COPY bierze starą wersję. requirements.txt → do `/opt/jobsearch/`.

## Repo / git (2026-06-28)
- **GitHub:** `git@github.com:Ikatrefon/job-search.git` (PRYWATNE). Lokalne repo w `JOB SEARCH/`, branch `main`. Push przez SSH: `GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519" git push`. (Brak `gh`/tokenu na Macu — repo zakładał Michał ręcznie.)
- `.gitignore` wyklucza: `.env` (klucz!), `app/.venv`, `app/data` (sqlite+pdf), `__pycache__`, `template/cv.html`/`cv.pdf`/`_render.html`, `Zrzut*.png`. `template/assets/*.png` (logotypy) SĄ w repo.
- **Deploy na VPS nadal przez rsync** (`/opt/jobsearch/`), nie git pull — można później przełączyć na deploy z gita.

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
- **Głębsza ocena + eskalacja Opus na żądanie (2026-06-28):** Haiku dawał płytkie oceny. Rozwiązanie (uzgodnione): (1) bogatszy prompt `EVAL_SYS` — rekruter, ocenia seniority/transfer branżowy/red flagi/ton + zwraca `verdict` (2-4 zdania szczerej oceny); (2) **domyślna ocena = Sonnet 4.6** (`config.EVAL_MODEL`), bo wyjście oceny małe → tanio; (3) przycisk **„🔬 Pogłęb analizę (Opus 4.8)"** w detalu = ręczna eskalacja `POST /ad/{id}/deepen` → `_evaluate_ad(model=OPUS_MODEL)`. Filozofia: człowiek decyduje, kiedy płacić za Opus (zamiast kruchej auto-bramki). Auto-kaskada Haiku→Opus zostaje na ETAP 2 (Adzuna bulk).
  - `engine.evaluate(ad, base, model=None)` zwraca `(score, verdict, wants, gaps)`. summary w db = JSON `{verdict, wants}`; kolumna `ogloszenia.eval_model` (migracja ALTER). UI: box „Werdykt" + etykieta „oceniono: <model>". Ocena Sonnet ~24 s, Opus deepen ~17 s (overlay „Pracuję…").
  - Przykład sensu: oferta „Senior Product Designer (IC)" — Opus trafnie obniżył score do 58 i wskazał, że CV to profil MENEDŻERSKI (Head of UX), nie IC. Haiku tego nie wyłapywał.
- **Strona Ustawienia + przełącznik modeli (2026-06-28):** osobna podstrona `/settings` (link w nagłówku base.html: Poczekalnia/Ustawienia). Dropdowny wyboru modelu dla OCENY i GENERACJI z `config.MODELS` (Haiku 4.5 / Sonnet 4.6 / Opus 4.8) + próg. Wybór zapisywany w db `konfiguracja` (klucze `eval_model`, `gen_model`); silnik czyta `engine._eval_model()/_gen_model()` = `db.get_config(...) or config.EVAL_MODEL/GEN_MODEL`. Walidacja przez `config.VALID_MODEL_IDS`. Domyślnie nadal eval=Haiku, gen=Sonnet. Stary `/config` zastąpiony przez `/settings` GET+POST. Engine importuje teraz `db` (bez cyklu: config→db→engine).
- **Detal v2 (2026-06-28):** firma/kraj/link/„Zapisz dane" w `<details>` (zwijane); widoczny tylko tytuł (Enter submituje form) + link „otwórz ogłoszenie ↗". Karty analizy PEŁNEJ szerokości (stacked, jak werdykt), bez piktogramów: „Wymagania z ogłoszenia" (było „Czego chce ogłoszenie") i „Pola do poprawy" (było „Luki vs bazowe CV") — to drugie zachowuje 2 wewn. kolumny: „Brakuje (nie ma w CV)" | „Warto podkręcić".
- **Przearanżowany detal + edycja danych (2026-06-28):** Zamiast „długa lewa kolumna + krótka prawa" — jeden pionowy przepływ (max-w-4xl): nagłówek (score + EDYTOWALNE tytuł/firma/kraj/link — `POST /ad/{id}/meta`, `db.update_ad_meta`, działa po audycie) + werdykt + eval/deepen → analiza w 2 kolumnach (Czego chce | Luki) → treść ogłoszenia → **CV/generacja NA DOLE** (gen button na dole; przy istniejącym CV: Pobierz PDF + podgląd + „Popraw" + „Generuj ponownie").
- **Uproszczenie + drag&drop kolejności (2026-06-28):** Usunięty panel akcji z detalu (Akceptuj/Odrzuć/Eksportuj/Usuń/status — był zbędny; „Eksportuj" tylko zmieniał etykietę). Zostaje: generacja + edycja + Pobierz PDF wewnątrz detalu; kasowanie ✕ na liście. Statusy wycofane (kolumna `status` zostaje, nieużywana; route `/status` usunięty; znacznik statusu zdjęty z listy). **Ręczna kolejność poczekalni:** kolumna `ogloszenia.ord` (migracja ALTER; nowe ad: `ord=-id` → na górze), `db.reorder(ids)`, `list_ads ORDER BY ord ASC, id DESC`. UI: SortableJS (CDN) na `#ad-list`, uchwyt `.drag-handle` (⠿), `onEnd` → `fetch POST /reorder {ids:[...]}` (cała lista w nowej kolejności). Filtrowanie = później.
- **Kasowanie ogłoszeń (2026-06-28):** `db.delete_ad()` + `POST /ad/{id}/delete` (usuwa wpis + cv_wygenerowane + plik PDF) → redirect na listę. UI: ✕ przy wierszu w poczekalni (wiersz przebudowany: `<a>` link + osobny form, bez zagnieżdżania) oraz „Usuń" w akcjach detalu; oba z `confirm()`.
- **Basic Auth (2026-06-28):** strona za hasłem (login `michal`). Konfiguracja w bloku `job` nginx: `auth_basic` + `auth_basic_user_file /etc/nginx/certs/job/.htpasswd` (plik na host: `.../nginx/certs/job/.htpasswd`, format `user:apr1-hash`). Zmiana hasła: `printf 'michal:%s\n' "$(openssl passwd -apr1 'NOWE')" > .../nginx/certs/job/.htpasswd` → `docker exec workload_nginx nginx -s reload`.
- **Fix (2026-06-28):** ocena Opus wywalała się `Extra data` — Opus dodawał komentarz PO obiekcie JSON. `_extract_json` używa teraz `json.JSONDecoder().raw_decode` od pierwszego `{` (bierze 1 kompletny obiekt, ignoruje resztę).
