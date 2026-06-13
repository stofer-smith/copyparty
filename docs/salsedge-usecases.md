# copyparty — SALS Edge Use Cases (draft)

Status: **draft for review**. Captured for our next session to expound on.
Project: copyparty v1.20.16 — single-file Python file server, multi-protocol, dependency-light.

## Why copyparty fits us

- One process speaks HTTP(S), WebDAV, SFTP, FTP(S), TFTP, SMB/CIFS.
- Per-folder, per-user accounts and volumes with granular permission flags (read / write / move / delete / admin).
- Resumable, unlimited-size uploads (chunked up2k protocol) with symlink dedup.
- Event hooks (`bin/hooks/`) and handlers (`bin/handlers/`) — arbitrary Python on upload/rename/delete. This is the real integration surface.
- SQLite search index over path / name / size / date / audio tags.
- Layered Docker images (`min` -> `im` -> `ac` -> `iv` -> `dj`); pick only the feature weight you need.
- fail2ban handler ships in-box (`bin/handlers/404-to-fail2ban.py`).

## Use cases

### 1. Client / vendor file exchange (Bibbeo, Loodon)
WebDAV plus per-volume accounts gives a partner a folder they map as a network drive. No SaaS, no per-seat cost. Resumable uploads handle large or flaky transfers that email and Workato choke on. Self-hosted Dropbox replacement on DigitalOcean, behind Nginx, fitting our non-default-port / UFW-first posture.

- **Effort:** low. **Risk:** low.
- **Open question:** account provisioning — manual conf edits vs. an IdP (Authelia/Authentik examples ship in `docs/examples/docker/`).

### 2. Automation ingestion endpoint (Workato / Make.com)
Point an upload volume at it; a hook in `bin/hooks/` fires on every new file — POST a webhook to Workato, push to Trackvia, drop a queue message. Turns "partner uploaded a file" into an automation trigger with no polling. Starter templates: `reloc-by-ext.py`, `notify.py`.

- **Effort:** medium (write/test the hook). **Risk:** low.
- **Open question:** delivery guarantees — does the hook retry on webhook failure, or do we need a queue in front?

### 3. Incident / log artifact drop
Realtime growing-file streaming + textfile viewer means logs are tailable in a browser. Natural landing zone for evidence files referenced by `/sals:incident-log`. `xiu-sha.py` checksums on upload for chain-of-custody.

- **Effort:** low. **Risk:** low.
- **Open question:** retention / auto-expiry policy for sensitive artifacts.

### 4. Internal media / asset library
Audio-tag indexing, thumbnail and spectrogram generation, cbz/manga and m3u8 playlist viewers make a workable internal asset browser. Lower-effort than Nextcloud.

- **Effort:** medium (needs the heavier `ac`/`iv` image + FFmpeg). **Risk:** low.

### 5. Throwaway / time-boxed shares
Self-destruct uploads (TTL) and temporary share links cover "send me that 4GB file" with no permanent account, scoped to one volume.

- **Effort:** low. **Risk:** low.

## Security notes (our non-negotiables)

- **Run non-root.** Set `user: "1000:1000"` explicitly. The base images do not force a non-root UID by default.
- **Avoid SMB over WAN.** README labels the SMB server "unsafe, slow, not recommended for wan." Use WebDAV or SFTP off-LAN.
- **TLS at the edge.** Terminate TLS at Nginx; keep copyparty on localhost / Docker network.
- **Restrict source IPs** where it's a LAN-only or partner-only service (`ipa:` volflag).
- **Version warnings.** Enable a `vc-url` so the control panel warns on known-vulnerable versions; `vc-exit` to shut down rather than run vulnerable.
- **fail2ban** via the in-box handler to throttle scanners.

## Next session

- Pick the first one or two to actually deploy (leaning #1 + #2).
- Decide accounts vs. IdP.
- Expand the chosen hook design (retry, payload shape, target system).
