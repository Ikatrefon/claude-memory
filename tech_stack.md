---
name: tech-stack
description: "Stack techniczny z pułapkami — Next.js 16, Prisma 7, Tailwind 4, Playwright"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 64536386-5f9d-4c18-b9fc-52f4927e5a25
---

## Next.js 16 (App Router) — BREAKING CHANGES vs wersji z treningu
- `proxy.ts` zamiast `middleware.ts`, funkcja `proxy()` zamiast `middleware()`
- `x-forwarded-proto` w proxy.ts (nginx terminuje SSL, Next.js widzi HTTP)
- `allowedOrigins` w next.config.ts dla Server Actions (problem z trailing dot od nginx)
- Server Actions body limit: `serverActions: { bodySizeLimit: '150mb' }` w next.config.ts
- `export const dynamic = "force-dynamic"` na stronach z DB
- `serverExternalPackages`: Prisma i pg muszą być w tej liście
- Zawsze czytaj `node_modules/next/dist/docs/` przed pisaniem kodu

## Prisma 7 (UWAGA — pułapki)
- Output: `output = "../src/generated/prisma"` w schema
- Import: `import { PrismaClient } from "@/generated/prisma/client"` (nie ma index.ts)
- Wygenerowane pliki commitowane do repo
- **Prisma 7 wymaga adaptera nawet dla SQLite** — jeśli nie potrzebujesz Accelerate/Edge, użyj Prismy 5!

## Prisma 5 (dla prostych projektów z SQLite)
- Import: `import { PrismaClient } from '@prisma/client'` (klasycznie)
- schema.prisma: `url = env("DATABASE_URL")` działa normalnie
- binaryTargets dla Alpine Docker: `["native", "linux-musl-openssl-3.0.x"]`
- `apk add --no-cache openssl` w obu stagach Dockerfile (builder + runner)
- **Kolejność w Dockerfile:** `COPY prisma ./prisma` PRZED `RUN npm ci` (postinstall odpala prisma generate!)
- entrypoint.sh: `node ./node_modules/prisma/build/index.js migrate deploy` zamiast `npx prisma` (npx ściąga Prismę 7!)

## PostgreSQL 16
- Standalone Docker container z named volume
- Driver: `@prisma/adapter-pg` + `pg`
- DATABASE_URL używa nazwy kontenera (nie localhost): `postgresql://postgres:postgres@nazwa-db:5432/nazwabazy`

## Tailwind CSS 4
- Znacząco zmienione API vs Tailwind 3 — sprawdź docs przed pisaniem

## Autentykacja
- JWT w cookie `session` (jose) + bcrypt (bcryptjs)
- Brak NextAuth — własna implementacja przez Server Actions

## Playwright (Chromium)
- `"postinstall": "playwright install chromium --with-deps"` wymagane w package.json
- Screenshoty NIE są serwowane ze statycznego `public/` w Next.js produkcji — wymagają API route

## Anthropic Claude API
- Klucz API w bazie danych (tabela AppSettings), nie w env vars
- Model: claude-opus-4-7

## Nixpacks + Dokploy
- `"engines": { "node": ">=20.19.0" }` w package.json (Prisma 7 nie działa na Node 18)

## AGENTS.md dla nowych projektów Next.js
Utwórz plik z treścią: "This is NOT the Next.js you know — read node_modules/next/dist/docs/ before writing any code."
I CLAUDE.md: `@AGENTS.md`
