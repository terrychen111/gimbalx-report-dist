# Licensing (trial and licensed editions)

GimbalX Report ships as **one installer and one codebase**. Whether a watermark appears depends only on
whether the local database holds a valid license.

## 1. Two states

| | Trial (default) | Licensed |
|---|---|---|
| Features | Full | Full |
| Time limit | None | Per `.lic` |
| Concurrent sessions | **1** (a new sign-in replaces the old one) | `seats` from the `.lic` (default 1; 3/5/unlimited available) |
| Same account on several devices | Not allowed (only the newest session survives) | Not allowed (only the newest session survives) |
| Rendered report over 2 pages | `[试用版 TRIAL]` watermark on every page | No watermark |
| Rendered report of 2 pages or fewer | No watermark | No watermark |
| CSV / JSON / XML data exports | Unaffected | Unaffected |
| Schedules / API / embed links | Available | Available |

**How seats are counted:** the number of live sign-in sessions (one browser/device = one seat).
When all seats are taken, **the new sign-in is rejected** with “license seats exhausted (N/N)”; people
already signed in are not affected. Someone must sign out (or let the session expire) to free a seat.

**One session per account:** in every edition, signing in with the same account on a second device
immediately signs the first device out (“signed in on another device, this session has ended”). This stops
a team from sharing a single admin account; create one account per person and buy the matching seats.

The decision is made **after rendering, before export**, based on the final page count — not the template
page count and not the number of data rows. If the rendered document exceeds 2 pages, the whole document is
watermarked on every page; 2 pages or fewer stay clean.

## 2. What the machine code is

- Format: `M7S8-6BP1-9ZFF-FSQN` (16 chars in 4 groups), derived from the OS installation identity:
  - Windows: `HKLM\SOFTWARE\Microsoft\Cryptography\MachineGuid`
  - Linux: `/etc/machine-id`
  - Docker: the container's machine-id by default; set `QRPT_MACHINE_ID` to the host value to keep it stable.
- It contains no hardware serial, MAC address or username, and is never transmitted anywhere.
- Reinstalling the OS, replacing the system disk or cloning a VM usually changes it.

## 3. Obtaining a license

1. Sign in, open **Settings → License**, click **Copy** next to the machine code.
2. Contact the author (WeChat QR in the README) with the machine code, organization name and number of machines.
3. The author signs a `.lic` file offline with the Ed25519 private key and sends it back.
4. Click **Import license**, choose the `.lic`; it takes effect immediately (no restart).
5. The page then shows licensee, masked serial, expiry and edition.

The setup wizard's second step accepts a license too, which is handy for pre-provisioned deployments.

## 4. License file format

```json
{
  "v": 1,
  "serial": "6NJKP7BJY8VS9PPFF01S164TSGC21ZQH",
  "licensee": "ACME Ltd.",
  "edition": "standard",
  "features": ["watermark_off", "scheduled_export"],
  "seats": 5,
  "issued_at": "2026-10-30T09:00:00Z",
  "expires_at": "2027-12-31T23:59:59Z",
  "machine_codes": ["M7S8-6BP1-9ZFF-FSQN"],
  "max_pages": null,
  "nonce": "…",
  "sig": "base64(Ed25519 signature)"
}
```

- `machine_codes` may list several machines so one file covers a team.
- `seats` is the **number of concurrent sessions**: `1` means one person online at a time, `5` means up to
  five sessions, `null`/`0` means unlimited. Issued with `--seats N` (default 1).
- An empty `expires_at` means perpetual.
- `sig` covers every other field; any modification invalidates it.
- Only the author's public key is embedded in the application. The private key never ships and stays offline.

## 5. Moving machines / reinstalling / scaling

| Scenario | What to do |
|---|---|
| Reinstalled OS or new machine | Machine code changes — request a new `.lic` |
| Reinstalled the app on the same machine | Machine code usually unchanged — re-import the same `.lic` |
| VM clone | Clones may share the machine-id; issue a separate license per clone |
| Docker container rebuilt | Without `QRPT_MACHINE_ID` the code changes; fix it to keep the license valid |
| More machines | Send the new machine codes; the author adds them or issues separate files |
| More concurrent users | Re-issue with the same machine code and a larger `--seats`, then import it over the old file |

## 6. Data and privacy

- Verification is fully offline: no network calls, no telemetry, no license server.
- The license is stored encrypted in the local `app_setting` table; backing up the database backs it up.
- Removing a license is currently an author-side operation (clear the setting) or a re-issue for a new machine.

## 7. Troubleshooting

- **"Invalid license / bad signature"** — the file was edited or not issued by the author; request the
  original `.lic` and avoid re-saving it in a text editor.
- **"Machine code mismatch"** — the license was issued for another machine; request one for this machine code.
- **"License expired"** — renew; the author issues a new file with a later `expires_at`.
- **A 2-page report still shows a watermark?** — check the rendered page count; footers, blank pages and
  sub-reports count. The rule uses the final rendered page count.
- **Can the watermark be removed?** — It disappears automatically with a valid license. Bypassing it is
  prohibited by the license.
- **Does the trial expire or lock features?** — No time limit and no feature locks; only the >2-page watermark.
- **"License seats exhausted (N/N)" on the licensed edition** — as many sessions are online as the license
  allows. Have someone sign out (or wait for a session to expire); for permanent growth ask the author for a
  larger `seats` value.
- **Can one license/account be shared with the whole company?** — The license is bound to machine codes, so
  using one `.lic` on one server is fine; accounts are one-to-one — signing in with the same account on a
  second device signs the first out. Create one account per person and buy the matching seats.
