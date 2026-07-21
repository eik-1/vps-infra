# VPS infrastructure

Configuration for the Docker services running on this VPS. Runtime state,
database contents, certificates, and credentials are intentionally excluded
from this repository.

The CouchDB service supports a self-hosted Obsidian Sync setup. For the setup
guide, see [the original walkthrough](https://x.com/0xeik/status/2079472262315680067).

## Rebuild from zero

1. Install Docker Engine and the Docker Compose plugin.
2. Clone this repository on the server.
3. Configure and start CouchDB:

   ```sh
   cd couchdb
   cp local.example.ini local.ini
   cp .env.example .env
   chmod 600 .env local.ini
   # Edit .env and set a unique username and strong random password.
   docker compose up -d
   ```

4. Install Caddy and place `caddy/Caddyfile` at `/etc/caddy/Caddyfile`.
5. Ensure the DNS record for the CouchDB hostname points to this VPS, then
   validate and reload Caddy:

   ```sh
   caddy validate --config /etc/caddy/Caddyfile
   systemctl reload caddy
   ```

6. Verify the service through its HTTPS hostname. Caddy obtains and renews
   TLS certificates automatically when ports 80 and 443 are reachable.

## Operational notes

- Back up `couchdb/data/` independently; it contains all CouchDB data and is
  excluded from Git.
- Keep `couchdb/.env` and `couchdb/local.ini` only on the server or in a
  dedicated secrets manager.
- Do not commit Caddy's certificate or runtime-state directories if they are
  later stored below this repository.
