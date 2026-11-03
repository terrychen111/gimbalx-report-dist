# macOS single-machine install (portable, no Docker)

For **Apple Silicon (arm64) / Intel (x86_64)** on macOS 12 or newer. The archive bundles a CPython
runtime and all dependencies — no system Python, pip, Node.js or Docker required.

> Build note: macOS packages must be built on macOS (tar must preserve symlinks and executable bits).
> `scripts/build-single-macos.sh` is provided, but this release was **not** built or tested on a Mac.
> If you have no Mac build machine, use Docker Desktop or the source install instead.

## 1. Extract

```bash
tar -xzf GimbalX Report-1.0.0-macos-arm64.tar.gz     # use -macos-x86_64 on Intel
cd GimbalX Report-1.0.0-macos-arm64
```

## 2. First run (Gatekeeper)

```bash
xattr -dr com.apple.quarantine .
./start.sh
```

Open <http://127.0.0.1:8090>. First start redirects to `/setup`: create the administrator, note the
machine code (or skip and continue in trial), then sign in.

> The package listens on `0.0.0.0:8090`, so other machines on the LAN can use
> `http://<mac-ip>:8090` (accept the macOS “allow incoming connections” prompt, or set
> `HOST=127.0.0.1` for localhost-only). The trial edition allows a single active login session; a
> new login signs the previous one out, and importing a license removes that limit.

## 3. Auto-start with launchd

```bash
./tools/install-service.sh               # LaunchAgent (per user, starts at login)
sudo ./tools/install-service.sh --system # LaunchDaemon (starts at boot)
./tools/install-service.sh --port 8091   # different port
```

```bash
launchctl print gui/$(id -u)/com.gimbalx.report | head -20
tail -f logs/service.err.log
```

Uninstall with `./tools/uninstall-service.sh` (`--system` for the daemon).

## 4. Configuration and data

Same `.env` keys as the Linux/Windows packages (`HOST`, `PORT`, `DB_SQLITE_PATH`, `DATA_ROOT`,
`SECRET_KEY`, `CONFIG_ENCRYPTION_KEY`, `CORS_ORIGINS`, `SCHEDULER_ENABLED`). Data lives in `data/`;
back up that directory plus `.env`.

## 5. Upgrade

```bash
launchctl bootout gui/$(id -u)/com.gimbalx.report 2>/dev/null || true
tar -xzf GimbalX Report-<new>-macos-arm64.tar.gz
cp -a old/data new/ && cp -a old/.env new/
cd new && ./tools/install-service.sh
```

`start.sh` runs database migrations automatically on every start.

## 6. Licensing

Trial watermarks every page when a report renders more than 2 pages (preview and exports); two pages
or fewer are clean. “Settings → License”: copy the machine code (macOS `IOPlatformUUID`), send it to
the author, upload the `.lic` — effective immediately. See [licensing.en.md](licensing.en.md).

## 7. Troubleshooting

| Symptom | Fix |
|---|---|
| “cannot be opened because the developer cannot be verified” | `xattr -dr com.apple.quarantine .` or right-click → Open |
| `bad CPU type in executable` | wrong architecture bundle; download the matching one |
| Port in use | `lsof -nP -iTCP:8090 -sTCP:LISTEN`, or change `PORT` |
| Service restarts repeatedly | check `logs/service.err.log`; ensure the directory is writable |

## 8. Verification status

- Build and launchd scripts are included and pass `bash -n` syntax checks plus plist structure validation;
- **Not executed on real macOS hardware**: build, Gatekeeper approval and launchd auto-start are unverified;
- To produce a macOS build, on a Mac run:

```bash
cd frontend && npm install && npm run build
cd .. && bash scripts/build-single-macos.sh --version 1.0.0
```