---
name: infrastructure
description: Infrastruktura Mac + VPS + schemat deploymentu
metadata: 
  node_type: memory
  type: reference
  originSessionId: 64536386-5f9d-4c18-b9fc-52f4927e5a25
---

## Mac (lokalny)
- Git: user = Michal, email = ika.trefon@gmail.com
- SSH klucz: ~/.ssh/id_ed25519
- Docker Desktop zainstalowany
- Projekty: ~/nazwa-projektu/

## VPS (Hostinger)
- Ubuntu 24.04 LTS, IP: 195.35.56.37
- SSH: `ssh root@195.35.56.37`
- Docker, Docker Swarm, Dokploy
- Panel Dokploy: http://195.35.56.37:3000
- GitHub App: Dokploy-2026-04-18-d9je17

## Schemat deploymentu
Mac → git push → GitHub (Ikatrefon) → Dokploy → Nixpacks → Docker image → Docker Swarm → nginx → domena

## Ważne szczegóły
- nginx (`workload_nginx`) zajmuje porty 80/443
- nginx config: `/opt/workload-estimator/workload-estimator/nginx/nginx.conf`
- Sieć Docker: `dokploy-network` (overlay, attachable)
- Po restarcie VPS: standalone kontenery wypadają z sieci — trzeba ponownie podłączyć (`docker network connect dokploy-network nazwa-kontenera`)
- SSL: `certbot certonly --standalone` (port 80 musi być wolny → najpierw `docker compose stop nginx`, potem certbot, potem `start nginx`)
- Certyfikaty: generowane do `/etc/letsencrypt/live/<domena>/`, kopiowane ręcznie do `nginx/certs/<name>/fullchain.pem` + `privkey.pem`
- Nowe domeny: dodać server block do nginx.conf **wewnątrz** bloku `http {}` (przed ostatnim `}`)
- Postgres jako standalone container z named volume (dane trwałe)
- Domeny: Rejestrator Hostinger (hpanel.hostinger.com), rekordy A, TTL ~300s

## Aktywne domeny
- auditor.mdmresearch.com → UX Auditor
- lds-lindt-webcam.mdmresearch.com → Lindt LiveCam widget (port 8089)

## Publiczny proxy — REALIA (sprostowanie, ustalone 2026-06-18)
- Ruch 80/443 obsługuje **dokeryzowany `workload_nginx`** (kontener), NIE systemowy nginx (systemd nginx jest NIEAKTYWNY — edycja `/etc/nginx/sites-*` nic nie daje).
- Config bind-mount: host `/opt/workload-estimator/workload-estimator/nginx/nginx.conf` → kontener `/etc/nginx/nginx.conf`. Certy: host `.../nginx/certs/` → `/etc/nginx/certs/`.
- Każda gra/domena = blok `server { listen 443 ssl; server_name X; location / { proxy_pass http://195.35.56.37:PORT; } }` (proxy do hosta po IP:port, bo nginx jest w kontenerze — NIE `localhost`).
- Przeładowanie: `docker exec workload_nginx nginx -t && docker exec workload_nginx nginx -s reload`.
- **Nowa domena wymaga:** rekord DNS A (Michał w Hostingerze — **BRAK wildcard**, losowa subdomena się nie rozwiązuje) + cert (certbot) + blok server w nginx.conf + kontener na nowym porcie.
- **Firewall (ufw aktywny):** publicznie tylko 22/80/443. Surowy `IP:port` z zewnątrz NIE przejdzie — wszystko musi iść przez workload_nginx.
- Porty zajęte (przykł.): 8088 pinball, 8089 webcam, 8094 gift-catcher, 8095 arkanoid3d, 8101 pickmix.
- Gry NIE mają restart-policy spójnie (część `unless-stopped`, część `no`) → po reboocie VPS część kontenerów nie wstaje sama; build dir bywa w `/tmp` (ulotny).
