# Linux single-machine install (portable, no Docker)

Runs on **x86_64 / aarch64** Linux. The archive bundles a CPython runtime and every
dependency, so you do **not** need system Python, pip, Node.js or Docker.

- Requirements: glibc ≥ 2.17 (CentOS 7 / Ubuntu 16.04 or newer), 1 GB RAM, 500 MB disk;
- Optional: systemd for auto-start; put nginx/caddy in front for HTTPS;
- Architecture: the published archive is x86_64; on ARM build it on the target machine with
  `scripts/build-single-linux.sh --arch aarch64`;
- Alpine/musl: build with `--libc musl` (experimental, not tested on Alpine).

## 1. Extract

```bash
tar -xzf GimbalX Report-1.0.0-linux-x86_64.tar.gz
cd GimbalX Report-1.0.0-linux-x86_64
```

Or install into `/opt/gimbalx`:

```bash
sudo mkdir -p /opt/gimbalx
sudo tar -xzf GimbalX Report-1.0.0-linux-x86_64.tar.gz -C /opt/gimbalx --strip-components=1
cd /opt/gimbalx
```

Do not place it under `/usr` or `/etc`; the runtime user needs write access to `data/` and `logs/`.

## 2. Run in the foreground

```bash
./start.sh
```

Open <http://127.0.0.1:8090>. On first start you are redirected to `/setup`:

1. create the administrator (username ≥ 3 chars, password ≥ 8);
2. note the machine code (or choose “skip and continue in trial”);
3. finish and sign in.

Press `Ctrl+C` to stop. When it works, register it as a service.

### 2.1 Post-install checklist (recommended)

```bash
# 1) integrity (value on the release page's SHA256SUMS)
sha256sum GimbalX Report-1.0.0-linux-x86_64.tar.gz

# 2) no system Python / Docker needed
./runtime/python/bin/python3 -V          # bundled runtime → Python 3.12.x
env -i PATH=/usr/bin:/bin ./start.sh     # also starts in a minimal environment

# 3) service and listener
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8090/          # 200
curl -s http://127.0.0.1:8090/api/v1/setup/status                        # initialized + machine code
curl -s http://127.0.0.1:8090/api/v1/license/status                      # trial=true → trial edition
ss -ltnp | grep 8090                                                     # 0.0.0.0:8090 by default (LAN reachable)
```

Key points:

- **Browsers**: Firefox, Chrome/Chromium and Edge all work; do not use text browsers (`w3m`, `lynx`).
- **Access**: the package listens on `0.0.0.0:8090`, so other machines can open `http://<server-ip>:8090`
  (open the firewall: `sudo ufw allow 8090/tcp`, plus the cloud security group). Use `HOST=127.0.0.1`
  or `install-service.sh --bind 127.0.0.1` for localhost-only. For untrusted networks put §7's reverse
  proxy in front instead of exposing 8090 directly.
- **First run**: `/setup` has three steps (administrator → license → done). After it finishes click
  “去登录”; if the page does not navigate, open <http://127.0.0.1:8090/login> directly (known issue in
  older builds, fixed in the current package).
- **Concurrent sessions**: the trial edition allows one logged-in session at a time — a new login signs
  the previous one out (“account signed in on another device”). The licensed edition allows `seats`
  concurrent sessions from the `.lic` (default 1); once full, new logins get HTTP 403
  “license seats exhausted (N/N)”. An account always keeps a single session — create one account per person.
- **After systemd install**: `systemctl is-active gimbalx-report` must be `active` and
  `curl -I http://127.0.0.1:8090/` must return 200; `install-service.sh --port/--bind` rewrites
  `PORT`/`HOST` in `.env` so both stay in sync.
- **Trial expectation**: reports longer than **2 pages** carry `[试用版 TRIAL]` on every page; ≤2 pages
  have no watermark. Importing a `.lic` removes it immediately.
- **Data**: `data/qrpt.db` plus `storage/` — copy these two to upgrade or migrate.

## 3. Register as a systemd service (auto-start, recommended)

```bash
sudo ./tools/install-service.sh                 # service gimbalx-report, 0.0.0.0:8090
sudo ./tools/install-service.sh --bind 127.0.0.1  # localhost only
sudo ./tools/install-service.sh --user qrpt     # run as a dedicated user (recommended)
sudo ./tools/install-service.sh --port 8091     # different port
```

It writes `/etc/systemd/system/gimbalx-report.service` and enables/starts it.

```bash
sudo systemctl status gimbalx-report
sudo systemctl restart gimbalx-report
sudo journalctl -u gimbalx-report -f
tail -f logs/service.err.log
```

The service listens on `0.0.0.0:8090` by default so other machines on the same network can use it.
See §7 for a reverse proxy, or set `HOST=127.0.0.1` / `--bind 127.0.0.1` for localhost-only.

## 4. Configuration (.env)

| Variable | Default | Meaning |
|---|---|---|
| `HOST` | `0.0.0.0` | Bind address; `0.0.0.0` exposes it to the LAN, `127.0.0.1` for localhost only |
| `PORT` | `8090` | Port (re-run `install-service.sh --port` after changing) |
| `DB_SQLITE_PATH` | `./data/qrpt.db` | SQLite database file |
| `DATA_ROOT` | `./data` | Data root (reports, uploads) |
| `SECRET_KEY` | random | JWT signing key — keep private; changing it logs everyone out |
| `CONFIG_ENCRYPTION_KEY` | random | Encrypts stored datasource passwords — changing it makes them unreadable |
| `CORS_ORIGINS` | localhost | Keep default when front and back end are on the same host |
| `SCHEDULER_ENABLED` | `false` | Scheduled reports |
| `TZ` | `Asia/Shanghai` | Time zone |

## 5. Data and backup

```
data/qrpt.db               SQLite database (users, reports, license)
data/reports/              report files and export cache
data/datasource_files/     uploaded CSV/Excel data sources
.env                       configuration and keys
```

```bash
sqlite3 data/qrpt.db ".backup '/backup/qrpt-$(date +%F).db'" 2>/dev/null \
  || cp data/qrpt.db "/backup/qrpt-$(date +%F).db"
tar -czf "/backup/gimbalx-data-$(date +%F).tar.gz" data .env
```

Restore: stop the service, copy `data/` and `.env` back, start the service.

## 6. Upgrade (keep data)

```bash
sudo systemctl stop gimbalx-report
sudo tar -xzf GimbalX Report-<new>-linux-x86_64.tar.gz -C /opt
sudo cp -a /opt/gimbalx/data /opt/GimbalX Report-<new>-linux-x86_64/
sudo cp -a /opt/gimbalx/.env /opt/GimbalX Report-<new>-linux-x86_64/
sudo /opt/GimbalX Report-<new>-linux-x86_64/tools/install-service.sh --dir /opt/GimbalX Report-<new>-linux-x86_64
```

`start.sh` runs `app.cli db-upgrade` automatically on every start, so no manual Alembic step is needed.

## 7. Reverse proxy / HTTPS (optional)

```nginx
server {
    listen 80;
    server_name report.example.com;
    client_max_body_size 200m;

    location / {
        proxy_pass http://127.0.0.1:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 300s;
    }
}
```

Add TLS with `certbot --nginx` (or your corporate certificate), set `CORS_ORIGINS` in `.env` to the
public URL, then restart. Only expose 80/443 — never publish 8090 directly.

## 8. Licensing (trial → licensed)

- Trial: fully functional; when a report renders **more than 2 pages**, every page of the document
  carries a `[TRIAL]` watermark (preview and exports). Two pages or fewer have no watermark.
- Licensed: “Settings → License”, copy the machine code, send it to the author, upload the `.lic`
  you receive — effective immediately, no reinstall or restart.
- Machine code comes from `/etc/machine-id`; for containers/clones pin it with `QRPT_MACHINE_ID`.
- Details: [licensing.en.md](licensing.en.md).

## 9. Troubleshooting

| Symptom | Fix |
|---|---|
| `Permission denied` on `start.sh` | `chmod +x start.sh`; make sure extraction kept permissions |
| `Address already in use` | `ss -lntp \| grep 8090`, stop the other process or change `PORT` |
| `GLIBC_2.xx not found` | OS too old (< CentOS 7); use the Docker image or upgrade |
| Login 401 / blank page | check `logs/service.err.log`, `journalctl -u gimbalx-report`, and `CORS_ORIGINS` |
| Upload returns 413 | raise `client_max_body_size` in nginx |
| Service won't start | `systemctl status gimbalx-report -l`; ensure `WorkingDirectory` matches `--dir` |

Logs: foreground output, or `logs/service.out.log` / `logs/service.err.log` under systemd.

## 10. Uninstall

```bash
sudo ./tools/uninstall-service.sh      # stop and remove the unit; keeps data/ and .env
rm -rf /opt/gimbalx                        # remove program and data when no longer needed
```

## 11. Build the Linux archive yourself

```bash
cd frontend && npm ci && npm run build && cd ..
bash scripts/build-single-linux.sh --version 1.0.0          # output: dist-single-linux/
bash scripts/build-single-linux.sh --arch aarch64           # ARM servers
bash scripts/build-single-linux.sh --libc musl              # Alpine (experimental)
bash scripts/build-single-linux.sh --reuse-runtime          # repack without re-downloading
```

The build machine needs `curl`, `tar` and network access; the target machine needs neither.