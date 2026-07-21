# CouchDB

This directory runs a single-node CouchDB instance for the Obsidian vault.

## First deployment

1. Create the local configuration and credentials. Neither file is committed:

   ```sh
   cp local.example.ini local.ini
   cp .env.example .env
   chmod 600 .env local.ini
   ```

2. Set a unique admin username and a long random password in `.env`.
3. Start CouchDB:

   ```sh
   docker compose up -d
   ```

The first start initializes the admin account from `COUCHDB_USER` and
`COUCHDB_PASSWORD`. The database is stored in `data/`, which is deliberately
ignored by Git. Back it up separately before rebuilding the server.

## Update

```sh
docker compose pull
docker compose up -d
```

The service is bound to `127.0.0.1:5984`; Caddy proxies the public HTTPS
endpoint.
