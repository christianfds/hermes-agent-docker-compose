# hermes-agent-docker-compose

Deployment for the [Hermes Agent](https://hermes-agent.nousresearch.com/) on my
VPS, managed by **Portainer** (Git repository stack) and reverse-proxied by the
existing **Traefik** to:

> **https://hermes.christianfds.dev**

The agent is powered by **OpenCode Go** models (`https://opencode.ai/zen/go/v1`).

## What this stack does

- Runs the official image `nousresearch/hermes-agent:latest` as a persistent
  gateway with the built-in **web dashboard** enabled (`HERMES_DASHBOARD=1`).
- Stores all state in `/data/container/hermes` on the host (mounted at
  `/opt/data` in the container): config, sessions, memory, skills, logs.
- Joins the existing external `traefik` Docker network and publishes no host
  ports — Traefik reaches the dashboard on container port `9119`.

## Layout

| File | Purpose |
|------|---------|
| `docker-compose.yml` | The stack (no secrets). |
| `config.example.yaml` | Reference for the seeded `/data/container/hermes/config.yaml`. |
| `.env.example` | Reference for the host `/data/container/hermes/.env` (secrets). |

## Host prerequisites (created once, not in git)

```sh
sudo mkdir -p /data/container/hermes
sudo chown -R 1000:1000 /data/container/hermes

# Model + reverse-proxy settings (no secrets)
cp config.example.yaml /data/container/hermes/config.yaml
```

## Deploy (Portainer)

1. **Stacks → Add stack → Repository**.
2. Repository URL: `https://github.com/christianfds/hermes-agent-docker-compose`
3. Reference: `refs/heads/main`, Compose path: `docker-compose.yml`
4. **Environment variables** — add the values from `.env.example`:
   - `OPENCODE_GO_API_KEY`
   - `HERMES_DASHBOARD_BASIC_AUTH_USERNAME`
   - `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD`
   - `HERMES_DASHBOARD_BASIC_AUTH_SECRET` (`openssl rand -base64 32`)
5. **Deploy the stack.**

## DNS / TLS

Cloudflare `A` record `hermes` → VPS IP, **proxied**. Traefik issues the
certificate through the existing `letsencryptresolver` (Cloudflare DNS-01),
so no extra TLS configuration is needed.

## Updating

Portainer → Stacks → the stack → **Pull and redeploy** (re-pulls the image on
recreate), or `docker compose pull && docker compose up -d` on the host. The
`/data/container/hermes` volume is preserved across updates.
