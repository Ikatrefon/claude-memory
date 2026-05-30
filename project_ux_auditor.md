---
name: project-ux-auditor
description: "Projekt UX Auditor — status, repo, lekcje"
metadata: 
  node_type: memory
  type: project
  originSessionId: 64536386-5f9d-4c18-b9fc-52f4927e5a25
---

- Repo: git@github.com:Ikatrefon/ux-auditor.git
- Produkcja: https://auditor.mdmresearch.com
- Lokalna ścieżka: ~/ux-auditor/
- Kontekst sesji: ~/ux-auditor/SETUP-2026-05-24.md
- Admin: admin@assembly.com
- Status: działa produkcyjnie (2026-05-24)

## Co jeszcze nie zrobione
- RAG (reference files)
- Email powiadomienia
- Heurystyki Nielsena jako badge

## Lekcje z tego projektu (stosuj w przyszłości)
- Persistent volume dla plików runtime — screenshoty i uploady muszą mieć persistent volume w Dokploy (General → Volumes)
- Zawsze commituj WSZYSTKIE zmienione pliki — git status przed pushem
- Nowe gałęzie muszą być wypchnięte PRZED pierwszym fetch w Dokploy
- Docker network overlay po restarcie — standalone kontenery wypadają z sieci
- Reference files jako atrapa — można zbudować UI zanim zaimplementuje się logikę
