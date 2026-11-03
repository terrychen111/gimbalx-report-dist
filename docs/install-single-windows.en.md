# Windows single-machine installation and operations

Applies to `GimbalX Report-<version>-win64.zip`. The target machine needs **no Python, Node or MySQL**.

## 1. Requirements

| Item | Requirement |
|---|---|
| OS | Windows 10 / 11 / Server 2016+, x64 |
| RAM | 4 GB minimum, 8 GB recommended for large exports |
| Disk | ~150–250 MB for the package, plus your data |
| Port | 8090 by default (loopback only by default) |
| Privileges | Normal user for `start.bat`; administrator to install the service |

Do not install under `C:\Program Files` (service accounts cannot write logs/data there). `D:\GimbalX Report` is recommended.

## 2. Extract and first start

1. Extract the zip to `D:\GimbalX Report`. The folder should contain `app\`, `runtime\`, `frontend\dist\`,
   `alembic\`, `data\`, `tools\`, `.env`, `start.bat`, `install-service.bat`, `README.txt`.
2. Double-click `start.bat` (runs database migrations, then starts the server).
3. Open <http://127.0.0.1:8090>. The `/setup` wizard appears:
   - Step 1: create the administrator (password ≥ 8 characters).
   - Step 2: shows the machine code; import a `.lic` if you have one, otherwise **Skip and use the trial**.
   - Step 3: done — log in with the new account.
4. Keep the console window open, or install it as a service (next section).

## 3. Install as a Windows service

1. Download [NSSM](https://nssm.cc/download) and extract `nssm.exe` (e.g. `C:\tools\nssm.exe`).
2. Run `install-service.bat` as administrator, or:
   ```powershell
   powershell -ExecutionPolicy Bypass -File tools\install-service.ps1 -NssmExe C:\tools\nssm.exe
   ```
3. Options: `-Port`, `-ServiceName`, `-OpenFirewall`, `-BindHost` (default `0.0.0.0`; use `127.0.0.1` for localhost-only).
4. Logs: `logs\service.out.log`, `logs\service.err.log` (10 MB rotation).
5. Stop / remove:
   ```powershell
   & C:\tools\nssm.exe stop GimbalXReport
   & C:\tools\nssm.exe remove GimbalXReport confirm
   ```

> LAN access: the portable package already listens on `0.0.0.0:8090`, so other machines on the same
> network can open `http://<server-ip>:8090` directly. Allow TCP 8090 in Windows Firewall
> (`install-service.ps1 -OpenFirewall` does it automatically). To restrict to this machine only,
> set `HOST=127.0.0.1` in `.env` and restart, or pass `-BindHost 127.0.0.1` to the service script.
>
> Security: change the default administrator password before exposing the port and restrict the
> allowed network range. The trial edition allows only one active login session at a time (a new
> login signs the previous session out). The licensed edition allows `seats` concurrent sessions from
> the `.lic` (default 1) and rejects new logins once full. An account always keeps a single session.

## 4. Licensing (trial → licensed)

- Without a license the product runs in trial mode: any rendered report longer than 2 pages carries a
  `[试用版 TRIAL]` watermark on every page. All features remain available.
- Upgrade steps:
  1. **Settings → License**, copy the machine code (`XXXX-XXXX-XXXX-XXXX`).
  2. Send it to the author (WeChat QR in the README).
  3. Import the `.lic` file on the same page. Effective immediately, no restart.
- The machine code changes if you move to another machine or reinstall the OS; request a new `.lic`.
- Verification is fully offline; the application never contacts an activation server.

See [licensing.en.md](licensing.en.md).

## 5. Data, backup, upgrade

| Content | Location |
|---|---|
| SQLite database | `data\qrpt.db` |
| Reports and images | `data\reports\` |
| Datasource files | `data\datasource_files\` |
| Export temp | `data\temp\` (safe to clear) |
| Logs | `logs\` |

Backup: stop the service and copy the whole `data\` folder (daily recommended).
Upgrade: extract the new version into a new folder, stop the old service, copy `data\` and `.env` across,
verify with `start.bat`, then point the service at the new folder
(`nssm set GimbalXReport AppDirectory <new folder>`). Schema upgrades run automatically on startup.

## 6. Optional MySQL 8

```sql
CREATE DATABASE qrpt DEFAULT CHARACTER SET utf8mb4;
CREATE USER 'qrpt'@'127.0.0.1' IDENTIFIED BY 'strong-password';
GRANT ALL PRIVILEGES ON qrpt.* TO 'qrpt'@'127.0.0.1';
```

Then edit `.env`: `DB_TYPE=mysql`, `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`,
`DB_AUTO_MIGRATE=true`, restart. For migrating existing SQLite data, use the scripts in `deploy/migrate/`.

## 7. Offline installation

The package already bundles the runtime and all dependencies — extract and run, no internet needed.
Only optional drivers (Oracle `oracledb`, SQL Server `pyodbc` + ODBC Driver 18) must be added manually
by unzipping their wheels into `runtime\Lib\site-packages`. PDF export uses ReportLab and does not need a browser.

## 8. Troubleshooting

- **start.bat closes immediately** — run it from `cmd` to see the error; usually a used port or empty `SECRET_KEY`.
- **Blank page in browser** — check `frontend\dist\index.html` exists and `SERVE_FRONTEND=true`.
- **Chinese characters are boxes in PDF** — install Microsoft YaHei on the machine, then restart.
- **Forgot the admin password** — `runtime\python.exe -m app.cli create-admin --username admin --password NewPass --force`.
- **"Machine code mismatch" when importing** — the `.lic` was issued for another machine; request a new one.
- **Copying the folder to another machine** — data moves with it, but request a new `.lic` for the new machine code.