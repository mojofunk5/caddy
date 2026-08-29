# caddy

Shared Caddy reverse proxy / TLS termination for every frontend hosted on the VPS at
`213.171.213.90`. Not app-specific - each frontend repo (`things-ui`, `hobbs-ui`, ...) only ships its
own static build; this repo owns the one process on the box allowed to bind ports 80/443 and routes
each domain to the right place.

## Why this exists as its own repo

Originally each frontend (`things-ui`) ran its own Caddy container. That worked until a second
frontend (`hobbs-ui`) needed the same host ports 80/443 - only one process can bind them, so two
independent Caddy containers can't coexist. Consolidated into one shared instance instead of
picking non-standard ports for one of them.

## Adding a new site

1. Add a new site block to `Caddyfile` (domain, `/api/*` reverse proxy to that app's own container
   by name - e.g. `things-app-1:8080` - and a `handle` block serving its static build)
2. Add that app's external Docker network (created by its own `docker-compose.yml`, e.g.
   `things-net`) to this repo's `docker-compose.yml`, and add a read-only volume mount for its
   `/opt/<repo>` directory
3. Push to `master` - `.github/workflows/deploy.yml` syncs both files to `/opt/caddy` on the VPS and
   reloads Caddy. No downtime - `caddy reload` swaps config without dropping connections.

## Deployment

Needs a `VPS_SSH_KEY` repository secret - see `~/Documents/ClaudeContext/ci-deploy-keys.md` for the
generic setup process (a dedicated keypair for this repo, not shared with any app repo's own key).
