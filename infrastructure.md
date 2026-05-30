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
