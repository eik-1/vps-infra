# CouchDB and Obsidian Self-hosted LiveSync

This directory runs the CouchDB server used by the **Self-hosted LiveSync**
Obsidian plugin. Starting CouchDB provides storage; each Obsidian device also
needs the plugin's connection and encryption settings.

The primary writing device is **Linux**. **Mac and iPhone** fetch its notes.
LiveSync is normally two-way, even on devices used only for reading: an edit or
deletion there can also reach the other devices.

| Task | Start here |
| --- | --- |
| Connect Mac or iPhone | [Add or reconnect readers](#add-or-reconnect-mac-and-iphone): choose **Join this device**. |
| Recover the ID mismatch we encountered | [Recovery](#recovery-remote-document-id-mismatch): identify the authoritative copy before rebuilding. |
| Inspect a sync failure | [Logs and troubleshooting](#logs-and-other-troubleshooting). |
| Deploy a new server | [First deployment](#first-deployment), then configure the Linux primary. |

## Connection and storage

| Setting | Value or location |
| --- | --- |
| Public CouchDB URL | `https://couch.eik-nano.tech` |
| LiveSync database name | `obsidian_vault` |
| CouchDB username | `COUCHDB_USER` in this directory's private `.env` |
| CouchDB password | `COUCHDB_PASSWORD` in the same `.env` |
| Compose service / container | `couchdb` |
| Persistent database files | `/home/eik/docker/couchdb/data/` |
| Mounted CouchDB configuration | `/home/eik/docker/couchdb/local.ini` |
| HTTPS proxy configuration | `/etc/caddy/Caddyfile` |

Enter the URL and database name into separate plugin fields. The host port is
bound to **`127.0.0.1:5984`**. Caddy proxies HTTPS to it, with Cloudflare in front;
mobile devices should use the public HTTPS URL.

`local.example.ini` documents the single-node settings, request limits, and CORS
origins for Obsidian desktop and mobile. The deployed `local.ini` and `.env`
contain private configuration and stay out of Git.

## Three different passwords/passphrases

| Credential | Purpose | How to retain it |
| --- | --- | --- |
| CouchDB username and password | Authenticate to the VPS database. They do not decrypt notes. | Keep the private `.env` in a secure backup. |
| Vault encryption passphrase | Encrypt/decrypt LiveSync data on the devices before/after transfer. It is not an Obsidian vault login. | Save it in a password manager and use matching encryption settings on all devices. |
| Setup URI passphrase | Protect the exported connection and plugin settings while transferring them to another device. | Save/transfer it separately from the URI. |

The vault encryption passphrase cannot be recovered by resetting the CouchDB
password. LiveSync encryption protects the synchronised data; local Markdown
files and attachments remain readable on the device.

Setup URIs and QR codes carry sensitive settings, including credentials and
encryption configuration. Keep them out of Git, screenshots, and public logs.
Generate them from the working primary device rather than reconstructing the
reader's settings by hand.

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

### Configure the Linux primary in Obsidian

Use first-device setup when the sync database is new, or when Linux is
intentionally replacing the remote with its complete, current vault. Existing
healthy sync does not need this procedure to add another device.

1. Install and enable **Self-hosted LiveSync** in the intended Linux vault.
   Back up that entire vault, including the hidden `.obsidian` folder, and close
   Obsidian on the other devices before intentionally replacing server data.
2. Open **LiveSync settings → Setup → Rerun Wizard**, or the onboarding notice
   on a new installation. Choose **I am setting this up for the first time**.
3. Choose the manual server configuration option when no Setup URI is
   available. Older versions label this **Enter the server information
   manually**; newer versions use **Configure a remote manually**.
4. Configure end-to-end encryption. Use **V2: AES-256-GCM With HKDF** and securely
   save the vault passphrase. The recovery described below used path/property
   obfuscation. Keep encryption, obfuscation, file-name case handling, and ID
   configuration consistent across devices. When preserving existing server
   data, retain its original settings rather than inventing a new passphrase.
5. Select **CouchDB** and enter the connection details above. Leave **Use
   Internal API** off unless a diagnosed connection issue requires it. Run the
   connection test: **Test Settings and Continue**, or **Create or connect to
   database and continue**, depending on the version.
6. For intentional first-device initialisation, select **Restart and Initialise
   Server**. At the final overwrite confirmation, confirm the backups and
   acknowledgements. Under **Advanced**, enable **Use this device's settings**
   when Linux's settings should define the rebuilt remote. Older versions call
   this **Prevent fetching configuration from server**.
7. Select **I Understand, Overwrite Server**. This replaces the selected
   database's sync data with Linux's vault. Keep Linux awake and Obsidian open
   until initialisation/upload completes.
8. Acknowledge **All optional features are disabled**. Leave **Customisation
   Sync** and **Hidden File Sync** off until ordinary notes and attachments
   work; this notice does not disable ordinary file sync. Configure optional
   features separately later.
9. Run **Sync now**, then create and upload a small test note before adding
   readers. Use **Show log** to investigate errors. Menu names and Config
   Doctor recommendations can change between versions; review them rather
   than changing encryption/ID settings just to clear a warning.

## Add or reconnect Mac and iPhone

Start from the Linux primary after its rebuild or normal sync has succeeded.
Use the same current LiveSync version on participating devices. The recovery
on **3 October 2026** used **1.0.34**; this records the working version, rather
than a requirement to remain on it forever.

1. On Linux, run **Self-hosted LiveSync: Copy settings as a new Setup URI** from
   the command palette. The Setup page may call it **Copy the current settings
   to a Setup URI**. Generate it **after** a successful rebuild and protect it
   with a separate Setup URI passphrase. A QR code is also available for mobile.
2. On Mac or iPhone, create a fresh vault and install/enable Self-hosted
   LiveSync. Preserve any unique notes from an old vault before replacing it.
   Avoid another sync service, such as iCloud or Obsidian Sync, writing to the
   same vault alongside LiveSync.
3. Import the primary's URI or scan its QR code. In onboarding, choose
   **I am adding a device to an existing synchronisation setup**.
4. If importing the URI presents these three choices, use this table:

   | Choice | When to use it |
   | --- | --- |
   | **Initialize or overwrite the remote** | Only when the primary deliberately replaces server data with its own vault. |
   | **Join this device** | **Choose this on Mac and iPhone.** Connect the reader to the existing remote and proceed with fetching. |
   | **Apply settings only** | Change settings without running the full join/fetch workflow. This alone does not populate an empty vault. |

5. Select **Restart and Fetch Data** / **Apply and Fetch** when offered. For an
   empty reader vault, choose **Overwrite all with remote files**. This targets
   the reader's local files; it is different from overwriting the server.
6. Let download, decryption, file creation, and any restart finish. Keep
   Obsidian in the foreground and the iPhone awake during initial retrieval.
7. Check a recent note and an attachment. Edit the test note on Linux and
   confirm the new content arrives on the reader, then enable the preferred
   automatic sync mode on each device.

Keep old, unrefreshed vaults closed after a remote rebuild. If reusing one,
preserve its local work, import the current settings, and perform **Reset
Synchronisation on This Device** to fetch the rebuilt remote before normal
sync resumes. In a reader fetch, use the primary's shared configuration;
**Use this device's settings** is not the choice for pushing an old reader's
configuration over the primary's.

## Recovery: remote document ID mismatch

The relevant warning is:

```text
The remote document IDs do not match the configured ID key.
```

Related log entries include:

```text
Metadata identity is unresolved and will be left unchanged ... (document-id-mismatch)
Failed to read file ... Possibly unprocessed or missing
```

In this recovery, Mac had fetched records but its scanner processed **0 files**.
The report showed 208 retained metadata mismatch warnings. Linux initially ran
**0.25.83**; after updating to **1.0.34**, its report also showed **389** metadata
mismatches. Updating alone did not repair the existing sync records.

Both reports used **legacy IDs** (`idDerivationVersion: 0`), E2EE V2, path
obfuscation, and case-insensitive file-name handling. The problem was that the
stored metadata IDs disagreed with IDs derived from the current settings. The
reports did not prove which earlier setting change caused that inconsistency.
The warning's wording did not establish a separately generated random ID key.

The later missing-file messages did not by themselves establish missing server
chunks: Mac had not created the files. Authentication and fetching had worked.
Repeating the Mac fetch brought back the same inconsistent records.

### Full rebuild from the authoritative Linux vault

Use this when the same broad mismatch affects the primary, its ordinary files
are readable and current, and replacing remote sync state is intended. It
replaces remote sync history with Linux's current files. Preserve unique work
on every device first.

1. Close Obsidian on Mac and iPhone. Verify recent Linux notes and attachments,
   and copy the complete Linux vault to a separate backup location. Preserve
   the server database/configuration as well.
2. Align the plugin versions. Retain the intended Linux encryption passphrase,
   obfuscation, case handling, and ID configuration. Do not generate a different
   independent ID key separately on each device.
3. On **Linux**, open **LiveSync settings → Hatch → Overwrite Server Data with
   This Device's Files → Schedule and Restart**.
4. In the final confirmation, expand **Advanced** and enable **Use this device's
   settings** (older label: **Prevent fetching configuration from server**).
   This preserves the intended Linux settings instead of applying configuration
   from the server being replaced. Confirm the backup and overwrite
   acknowledgements, then select **I Understand, Overwrite Server**.
5. Keep Linux awake until **Rebuild everything operation completed** appears.
   The full rebuild resets its local LiveSync database, scans the actual vault
   files under the current settings, and recreates/uploads the remote. A
   remote-only resend would retain the inconsistent local IDs.
6. Run **Sync now** and check that the ID warning is gone. Create a test note
   and confirm upload succeeds before reconnecting readers.
7. Generate a **fresh Setup URI** and follow the reader instructions above,
   choosing **Join this device** and fetching into empty vaults. The user
   confirmed completion after the Linux rebuild and Mac join workflow.

Let the plugin perform the rebuild rather than manually deleting `data/` or
unlocking stale clients. The workflow coordinates local state, remote data,
and the lock used to keep old clients from mixing stale state into the rebuild.

For an isolated mismatch affecting a few records, use the developer's
[metadata ID inspection/repair guide](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/recovery.md#repair-a-metadata-document-id-mismatch)
instead of automatically rebuilding everything. If only one reader fails,
compare its settings/report with the primary before replacing server data.

## Logs and other troubleshooting

Run the server commands from `/home/eik/docker/couchdb`:

```sh
docker compose ps
docker compose logs --tail=200 --timestamps couchdb
```

For device diagnostics, use **LiveSync settings → Show log**, then **Sync now**.
If a notification disappears, use the command palette to run **Self-hosted
LiveSync: Generate full report for opening the issue with debug info**. Compare
the primary and reader reports. Review reports for secrets before sharing.

| Symptom | What to check |
| --- | --- |
| Remote document ID warning / skipped metadata | Compare plugin versions and encryption, ID, path obfuscation, and case settings. Use the recovery above if the authoritative primary also has a broad mismatch. |
| Database locked / device not resolved after rebuild | Refresh the reader's local database from the rebuilt remote. Mark it resolved only after successful retrieval; do not bypass the lock to resume an old local database. |
| HTTP 401 | CouchDB credentials and access to the selected database. Changing the vault passphrase will not fix authentication. |
| HTTP 403 with Cloudflare error 1010 | Cloudflare filtering. During the audit, Python's default user agent was blocked while browser-style requests succeeded with the same credentials. |
| CORS or mobile connection failure | HTTPS URL, proxy, and CORS origins in `local.ini`. Review the server configuration before using Internal API as a workaround. |
| Fetch finishes but files do not appear | Inspect metadata mismatch warnings and Hatch's file-watching/database-reflection suspension controls. A downloaded record count alone does not establish that files were created. |
| New edits stop arriving after initial setup | Check automatic sync settings, run Sync now, and keep the mobile app open while checking. Mobile background execution is limited. |
| Optional features disabled | Ordinary notes/attachments still sync. Hidden files and customisations need separate setup. |

## Backups

Keep both forms of backup:

- **Local vault:** readable notes, attachments, and `.obsidian`, especially on
  Linux before a rebuild. A server backup cannot contain a note that never
  uploaded.
- **VPS:** CouchDB data, the container's complete `/opt/couchdb/etc/` directory,
  private `.env` and `local.ini`, Compose configuration, and the active Caddy
  configuration. Runtime API changes can also be saved in container INI files.

Store backups privately and copy them off the VPS. Keep the vault encryption
passphrase and any independent ID recovery key securely: encrypted database
files plus the CouchDB password are not sufficient to recover readable notes.
Never commit archives, actual credentials, or vault data to this repository.

During the 3 October 2026 recovery, these private archives were created in:

```text
/home/eik/.t3/scratch/2026-10-03-i-have-couchdb-running-in-9bd2c1f7/backups/
```

- `couchdb-20261003T122435Z.tar.gz`: original pre-rebuild server state. Restored
  into an isolated temporary CouchDB container; all 5,103 current documents and
  revisions matched the exports. This tested database restoration, not note
  decryption.
- `couchdb-20261003T135518Z.tar.gz`: intermediate post-rebuild server state,
  before the final successful recovery. Checksum/archive readability verified;
  not restore-tested.

Each has an accompanying `.sha256` file. These are historical snapshots, not a
backup of the final working setup or newer notes. Do not rely on a scratch
directory as permanent backup storage. No scheduled backup was found in the
locations inspected during the audit; provider/external backups were not
checked.

For future file-based backups, follow
[CouchDB's backup guidance](https://docs.couchdb.org/en/stable/maintenance/backups.html),
including copying secondary indexes before main database files. Periodically
test restoration in an isolated instance before depending on an archive.

## Update

Back up first, then update the container from this directory:

```sh
docker compose pull
docker compose up -d
```

The Compose file uses the floating `couchdb:3` image tag. Update the Obsidian
plugin separately on each device and check compatibility before changing shared
encryption/ID settings. A container upgrade does not reset a reader's local
LiveSync database or repair its plugin configuration.

## Upstream guides

- [LiveSync first-device and additional-device setup](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/quick_setup.md)
- [Recovery, full rebuild, and local fetch](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/recovery.md)
- [Legacy and independent ID derivation](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/settings.md#independent-id-derivation)
- [Hidden File Sync](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/tips/hidden-file-sync.md)
