# Plane Local Docker Setup

This folder runs Plane Community using Docker Compose.

## Start

```powershell
docker compose up -d
```

Open the app:

```text
http://127.0.0.1/
```

Open the admin panel:

```text
http://127.0.0.1/god-mode/
```

## Admin Login

```text
Email: admin@plane.local
Password: PlaneAdmin123!
```

Do not register again after setup. This local instance already has its first
admin account, so use sign in.

## Stop

```powershell
docker compose down
```

## Restart Only Plane

```powershell
docker compose restart plane
```

If you changed `docker-compose.yml` or `proxy/Caddyfile`, recreate only the app
container:

```powershell
docker compose up -d --force-recreate --renew-anon-volumes --no-deps plane
```

The `--renew-anon-volumes` flag is needed because the current
`makeplane/plane-aio-community:stable` image has malformed anonymous volume
metadata.

## Check Status

```powershell
docker compose ps
```

```powershell
docker logs --tail 100 plane
```

## Useful URLs

```text
Main app:    http://127.0.0.1/
Admin panel: http://127.0.0.1/god-mode/
MinIO UI:    http://127.0.0.1:9002/
```

`/god-mode/admin/` is not a valid Plane admin route in this setup. It redirects
back to `/god-mode/`.

## Troubleshooting

If the admin page loops or still shows old registration state, clear browser
site data for `127.0.0.1` or open an incognito window.

If the app shows a loading screen immediately after restart, wait until the API
is ready:

```powershell
Invoke-WebRequest -UseBasicParsing -Uri http://127.0.0.1/api/instances/
```

Expected result is HTTP `200`.
