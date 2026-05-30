---
name: project-webcam
description: Projekt Lindt LiveCam — widget YouTube z przełączaniem lokalizacji Kilchberg, status i szczegóły techniczne
metadata:
  node_type: memory
  type: project
  originSessionId: b7193715-5ac0-4b41-bd65-f205ce3954e2
---

# Projekt Lindt LiveCam (WEBCAM)

## Cel
Widget embeddowalny na stronie Lindt — wyświetla YouTube livestream z fabryki/muzeum Kilchberg, z możliwością przełączania między lokalizacjami przyciskami prev/next.

**Why:** materiał prezentacyjny dla klienta Lindt.
**How to apply:** projekt statyczny (HTML/CSS/JS), brak backendu, deploy jako nginx static serve.

## Lokalizacja pliku
`/Users/michal/CLAUDE PROJECTS/WEBCAM/index.html` — all-in-one, pełny widget z edytorem layoutu.

## GitHub
Repo: `Ikatrefon/lindt-webcam` (GitHub)

## Deploy
- VPS: `http://195.35.56.37:8089`
- Domena: **https://lds-lindt-webcam.mdmresearch.com** (aktywna od 2026-05-30)
- SSL: certbot, cert expires 2026-08-28 (auto-renewal skonfigurowany)
- Serwer na VPS: dedykowany kontener nginx (nie workload_nginx)
- nginx proxy: `workload_nginx` → cert w `/opt/workload-estimator/workload-estimator/nginx/certs/webcam/`

## Lokalizacje (YouTube)
```javascript
const locations = [
  { locationSrc: 'ELEMENTS/Kilchberg - factory.svg', ytId: 'hZ5b-65Srhg' },
  { locationSrc: 'ELEMENTS/Kilchberg - museum.svg',  ytId: 'agoOHWNKA1w' }
];
```

## Ważne: YouTube embed wymaga HTTP/HTTPS
YouTube blokuje embedy z `file://` protokołu (Error 153). Lokalnie: `python3 -m http.server 8090`.

## Pliki ELEMENTS/
- `Live_cam_tlo.svg` — tło z subtelnymi liniami (rama) — plik w katalogu głównym WEBCAM (nie ELEMENTS!)
- `livecam_dot.json` — animacja Lottie (pulsujący czerwony dot)
- `Kilchberg - factory.svg` — napis lokalizacji
- `Kilchberg - museum.svg` — napis lokalizacji
- `description.svg` — opis (długa nazwa, skopiowana jako description.svg)
- Pozostałe SVG: strzałki nawigacyjne, tytuł, etc.

## Edytor layoutu
Klawisz **L** — toggle panelu edytora z suwakami X/Y/Skala dla każdego elementu.
Przycisk "Kopiuj CSS" → wypisuje finalne wartości do konsoli (`console.log`).

## Stack
- Marcellus (Google Fonts) — font [[feedback-lindt-font]]
- Lottie-web CDN — animacja dot
- YouTube IFrame API — embed video
- CSS `position: absolute` dla wszystkich child elementów (umożliwia edytor)
