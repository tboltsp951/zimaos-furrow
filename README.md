# Furrow for ZimaOS

Turns the single-file `furrow-garden-planner.html` into a self-hosted ZimaOS
app. The page keeps all of its behavior — bed grid planning, plant library,
frost-date calendar, tasks, harvest log — and gains one new thing:
**data is saved to your NAS disk** via a tiny storage bridge, instead of
living only in the browser's localStorage. That means every browser on your
network sees the same garden, and it survives container rebuilds.

## How it works

- `app/server.js` is a dependency-free Node HTTP server.
  - Serves `app/furrow-garden-planner.html` with a small `window.storage`
    bridge injected. The page already checks for `window.storage` first and
    falls back to localStorage, so the bridge just makes the on-NAS store win.
  - Exposes `GET/PUT /__storage__/<key>`, persisted to `/data/furrow.json`.
- `Dockerfile` builds a `node:20-alpine` image.
- `Apps/Furrow/docker-compose.yml` is the ZimaOS/CasaOS app manifest
  (`x-casaos` metadata) that mounts `/DATA/AppData/furrow` into the
  container. The `Apps/` folder layout is what ZimaOS's third-party store
  loader scans for apps.

## Files

```
zimaos-furrow/
  app/
    furrow-garden-planner.html   the app itself (unchanged)
    server.js                    web server + storage bridge
  Apps/
    Furrow/
      docker-compose.yml         ZimaOS app manifest (store entry)
      icon.svg                   app icon
      thumbnail.svg              store card image
      screenshot.svg             store screenshot
  Dockerfile
```

## Install on ZimaOS

Two paths — pick one.

### A. Build & run it yourself (no store needed)

Copy this folder to the ZimaOS device (or clone the repo), then over
SSH/terminal:

```sh
docker build -t furrow:latest .
docker run -d --name furrow \
  -p 8191:8080 \
  -v /DATA/AppData/furrow:/data \
  --restart unless-stopped \
  furrow:latest
```

Open `http://<zimaos-ip>:8191`.

### B. Add it as an app in the ZimaOS app store

1. Push this repo to GitHub (as `tboltsp951/zimaos-furrow`).
2. The `publish.yml` workflow builds and pushes
   `ghcr.io/tboltsp951/zimaos-furrow:latest` on every push to `main`.
3. In ZimaOS: **Settings → App Store → Add third-party store**, then paste
   the archive URL of your repo:
   `https://github.com/tboltsp951/zimaos-furrow/archive/refs/heads/main.zip`
4. Install **Furrow**. The default host port is **8191**; change it in the
   UI if you prefer.

The icon/thumbnail URLs in the manifest point at raw GitHub files, so they
only resolve once the repo is public.

## Testing locally (no Docker needed)

```sh
node app/server.js          # DATA_DIR defaults to app/data, PORT defaults to 8080
```

Then open `http://localhost:8080`, plant a couple squares, and confirm
`app/data/furrow.json` appears on disk.

## Notes

- The container runs as root so the bind mount at `/DATA/AppData/furrow`
  is always writable. Fine for a single-user home tool.
- React, Babel, and Google Fonts are loaded from the web (cdnjs / fonts
 .googleapis.com); the app still runs if they're unavailable, it just falls
  back to system fonts.
- ZIP frost-date estimation uses the public zippopotam.us and open-meteo
  APIs, so it needs outbound internet from the browser.
- Multi-browser sharing works because storage is server-side now.
