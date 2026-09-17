# Quartz Publish (NAS)

Runtime config for self-hosted single-note publishing. Images are built by
the separate [`quartz-publish`](https://github.com/jt196/quartz-publish)
repo and pulled from GHCR — this repo is just `docker-compose.yaml` + `.env`.

Unlike `docker-npm`/`docker-syncthing`, `.env` here is **not** committed —
it holds a real filesystem path and domain specific to this NAS, and this
repo is public. Copy `.env.example` to `.env` and fill in real values
before running anything.

Two containers, split so the internet-facing one never touches the vault:

- `stager` — reads the vault (an Obsidian vault folder synced onto this
  NAS via the existing `syncthing` container) read-only, mirrors only
  `publish: true` notes into an internal Docker volume (`content`). No
  exposed ports.
- `web` — Quartz + Caddy (static serving only), reads `content` read-only,
  listens on `8098` (host) → `8080` (container).

## Before first run

```bash
cp .env.example .env
$EDITOR .env   # set VAULT_DIR (the vault folder itself, not a parent
               # folder it happens to live in — see .env.example) and
               # PUBLIC_DOMAIN
```

## Bring up / update

```bash
docker compose up -d              # first run
docker compose pull && docker compose up -d   # update to latest image
```

## One manual step: Nginx Proxy Manager

Not part of compose — add a proxy host in the NPM UI:

- Domain: whatever you set `PUBLIC_DOMAIN` to
- Forward to: this NAS's IP, port `8098`
- Request a Let's Encrypt cert the same way as the other NPM-fronted
  services on this box.

## Using it

Install the [Private Quartz Publish](https://community.obsidian.md/plugins/private-quartz-publish)
Obsidian plugin, set its Base URL to `https://<PUBLIC_DOMAIN>`, then
right-click any note → **Publish to web**. Unpublish / Rotate URL work
the same way to revoke a link.
