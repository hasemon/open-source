# Medusa Local Development

This repository contains a local Medusa development setup with a Docker-backed Medusa backend and a separate Next.js storefront.

Project layout:

```text
medusa/
├── docker-compose.yml
├── medusa-backend/
└── storefront/
```

## Project URLs

| Service | URL |
| --- | --- |
| Medusa Backend API | `http://localhost:9000` |
| Medusa Admin | `http://localhost:9000/app` |
| Next.js Storefront | `http://localhost:8000` |
| Admin HMR WebSocket | `ws://localhost:42021/app` |
| PostgreSQL | `postgresql://medusa:medusa@localhost:55432/medusa` |
| Redis | `redis://localhost:6379` |

Inside Docker, use these service URLs instead:

```text
postgresql://medusa:medusa@postgres:5432/medusa
redis://redis:6379
```

## Requirements

- Docker Desktop
- Git
- Node.js 20 or 22 recommended when running the backend or storefront directly on Windows

## Start Services

```powershell
docker compose up -d postgres redis
```

Check that the services are running:

```powershell
docker compose ps
```

## Create the Medusa Project

If `medusa-backend` already exists and is complete, skip this section.

If a previous setup was interrupted and `medusa-backend` only contains partial files, remove it first:

```powershell
Remove-Item -Recurse -Force .\medusa-backend
```

Then scaffold the backend through Docker:

```powershell
docker compose run --rm medusa-cli
```

This creates `medusa-backend` and connects it to the Compose PostgreSQL service.

On Windows, dependency installation can be slow when it writes `node_modules` through a Docker bind mount. If that happens, keep PostgreSQL and Redis running with Docker, then scaffold from PowerShell instead:

```powershell
npx create-medusa-app@latest medusa-backend --db-url "postgresql://medusa:medusa@127.0.0.1:55432/medusa" --no-browser --use-npm
```

Choose `No` when asked about the Next.js Starter Storefront unless you want the storefront included.

## Run Database Migrations

After `medusa-backend` exists, run migrations before opening the admin:

```powershell
docker compose run --rm --no-deps --entrypoint sh medusa-backend -c "cd apps/backend && npx medusa db:migrate --execute-all-links --verbose"
```

The Docker setup uses `postgres` as the database hostname from inside containers. The backend config disables PostgreSQL SSL for this local Docker connection:

```ts
databaseDriverOptions: {
  connection: {
    ssl: false,
  },
}
```

## Run the Medusa Backend

```powershell
docker compose up -d medusa-backend
docker compose logs -f medusa-backend
```

When the logs show `Server is ready on port: 9000`, open:

```text
http://localhost:9000/app
```

Local admin login:

```text
Email: admin@medusa.local
Password: medusa123
```

The API base URL is:

```text
http://localhost:9000
```

## Run the Storefront

The storefront is installed separately in:

```text
storefront/
```

Install dependencies once:

```powershell
cd .\storefront
npm install
```

Run the storefront:

```powershell
npm run dev
```

Open:

```text
http://localhost:8000
```

The local storefront environment file is `storefront/.env.local`:

```env
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_DEFAULT_REGION=dk
NEXT_PUBLIC_BASE_URL=http://localhost:8000
```

It also contains the local publishable API key created by the backend seed:

```env
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_a0933d71b53f077a7176e3d125be2bf7aaee60fcc04921d523af86a3da1e4b36
```

If you create a new publishable key in the admin, replace this value in `storefront/.env.local`.

## Environment Notes

If the project was created from inside Docker, the generated backend environment may point to `postgres` and `redis`.

For running the backend inside Docker, use service names in `medusa-backend/apps/backend/.env`:

```env
DATABASE_URL=postgresql://medusa:medusa@postgres:5432/medusa
REDIS_URL=redis://redis:6379
```

For running the backend directly from Windows, use localhost values instead:

```env
DATABASE_URL=postgresql://medusa:medusa@127.0.0.1:55432/medusa
REDIS_URL=redis://127.0.0.1:6379
```

## Stop Services

Stop containers but keep the database volume:

```powershell
docker compose down
```

Stop containers and delete the database volume:

```powershell
docker compose down -v
```
