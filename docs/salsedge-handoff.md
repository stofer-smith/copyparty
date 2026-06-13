# copyparty — SALS Edge Session Handoff

Last session: 2026-06-13. Status: **drafts complete, nothing deployed.**

## What got done

1. **Use-case ideas doc** — `docs/salsedge-usecases.md`
   5 use cases (client file exchange, automation ingestion, incident artifact drop,
   media library, throwaway shares) with effort/risk, open questions, and our
   security non-negotiables.

2. **Hardened Docker deployment** — `docs/examples/docker/salsedge-hardened/`
   - `docker-compose.yml` — non-root (1000:1000), loopback-only bind (TLS at Nginx
     upstream), read-only rootfs, `cap_drop: ALL`, `no-new-privileges`, tmpfs /tmp.
   - `copyparty.conf` — placeholder accounts (NOT real secrets), a partner drop
     volume, IP allowlist hooks, version-vuln warnings (`vc-url` + `vc-exit`).

3. **Validated** — `docker compose config` passed clean (exit 0). All hardening
   directives resolved correctly; port bound to 127.0.0.1 as designed.

## State

- Branch: `develop`. **Nothing committed** — all three files are untracked working changes.
- No container ever run. No data path created. Accounts are placeholders.

## Pick up here (next session)

Decisions still open, roughly in order:

1. **Which use case(s) to deploy first.** Recommended: #1 (client file exchange)
   + #2 (Workato/Make ingestion hook). Both low-effort.
2. **Accounts vs. IdP.** Placeholder accounts in the conf today. Authelia/Authentik
   examples ship in `docs/examples/docker/` if we want SSO.
3. **TLS topology.** Compose assumes Nginx terminates TLS upstream and proxies to
   127.0.0.1:3923. Confirm that vs. copyparty handling TLS directly.

## Before any real deploy (gotchas the validator surfaced)

- The `./:/cfg` mount expands to wherever compose runs. Copy the `salsedge-hardened`
  folder to the deploy host, or set an absolute `/cfg` path.
- `xff-src: 127.0.0.1` in the conf trusts a *local* proxy. If Nginx is a separate
  container, change this to the Docker network IP/subnet or every client logs as the proxy.
- Create `/srv/copyparty/data` (or chosen path) owned by UID 1000 before first run —
  compose does not create or check it.
- Replace `CHANGE_ME_*` passwords. Scope `ipa:` to the actual audience.

## Reference

- README TOC covers everything; key sections: accounts-and-volumes, event-hooks,
  webdav-server, file-indexing.
- Hooks live in `bin/hooks/` (start from `notify.py`, `reloc-by-ext.py`, `xiu-sha.py`).
- fail2ban handler: `bin/handlers/404-to-fail2ban.py`.
