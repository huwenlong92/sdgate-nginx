# SDKit Gate Release And Deploy

SDKit Gate is an Nginx-only visual configuration tool for single-machine reverse proxy and site entry management.

This document is the public-facing release, install, upgrade, and packaging guide. The main project README is written in Chinese for local development and product context.

---

## Install

Install the latest release:

```bash
curl -fsSL https://raw.githubusercontent.com/huwenlong92/sdgate-nginx/main/scripts/install.sh | sh
```

Install a specific version:

```bash
VERSION=0.1.0 curl -fsSL https://raw.githubusercontent.com/huwenlong92/sdgate-nginx/main/scripts/install.sh | sh
```

By default, the installer downloads the release asset matching the current OS and CPU architecture:

```text
sdgate-darwin-arm64.tar.gz
sdgate-darwin-amd64.tar.gz
sdgate-linux-arm64.tar.gz
sdgate-linux-amd64.tar.gz
```

The binary is installed to:

```text
/usr/local/bin/sdgate
```

Override the install prefix:

```bash
PREFIX=/opt/sdgate curl -fsSL https://raw.githubusercontent.com/huwenlong92/sdgate-nginx/main/scripts/install.sh | sh
```

---

## Quick Start

Run SDKit Gate with the default local paths:

```bash
sdgate
```

Open the Web UI:

```text
http://127.0.0.1:9100
```

Default login:

```text
admin / admin
```

Change the password after the first login.

Default local runtime paths:

```text
~/.sdgate/
├── config.toml
├── sdgate.db
├── data/
└── logs/
```

---

## Install As systemd Service

Install and start SDKit Gate as a systemd service:

```bash
curl -fsSL https://raw.githubusercontent.com/huwenlong92/sdgate-nginx/main/scripts/install.sh | \
  sudo env INSTALL_SYSTEMD=1 sh
```

With `INSTALL_SYSTEMD=1`, the installer:

- installs `/usr/local/bin/sdgate`;
- creates `/etc/sdgate/config.toml` if it does not exist;
- creates `/var/lib/sdgate`, `/var/lib/sdgate/data`, and `/var/log/sdgate`;
- renders `/etc/systemd/system/sdgate.service`;
- runs `systemctl daemon-reload`;
- starts the service, or restarts it if it is already active.

Existing files are preserved:

```text
/etc/sdgate/config.toml
/var/lib/sdgate/sdgate.db
```

Recommended systemd paths:

```text
/etc/sdgate/config.toml
/var/lib/sdgate/sdgate.db
/var/lib/sdgate/data/
/var/log/sdgate
```

Check service status:

```bash
systemctl status sdgate
```

View logs:

```bash
journalctl -u sdgate -f
```

---

## Configuration

Example production config:

```toml
port = 9100
logs_dir = "/var/log/sdgate"
data_dir = "/var/lib/sdgate/data"

[database]
driver = "sqlite"
path = "/var/lib/sdgate/sdgate.db"
```

Override config at startup:

```bash
sdgate --config /etc/sdgate/config.toml --port 9100
```

For reverse proxy deployment, use Nginx or another edge proxy in front of SDKit Gate and proxy traffic to `127.0.0.1:9100`.

---

## Installer Environment Variables

Common variables:

| Variable | Default | Description |
| --- | --- | --- |
| `VERSION` | `latest` | Release version. Accepts `0.1.0` or `v0.1.0`. |
| `PREFIX` | `/usr/local` | Binary install prefix. |
| `GITHUB_REPO` | `huwenlong92/sdgate-nginx` | GitHub release repository. |
| `INSTALL_SYSTEMD` | `0` | Set to `1` to install the systemd service. |
| `CONFIG` | `/etc/sdgate/config.toml` | Config path for systemd install. |
| `DATA_DIR` | `/var/lib/sdgate` | Runtime data root for systemd install. |
| `LOGS_DIR` | `/var/log/sdgate` | Log directory for systemd install. |
| `NO_START` | `0` | Set to `1` to install systemd files without starting the service. |
| `GITHUB_PROXY` | empty | Optional GitHub download proxy. Supports `{url}` placeholder. |
| `RELEASE_BASE_URL` | empty | Optional custom release asset base URL. |
| `SDGATE_BIN_PATH` | empty | Install a local binary instead of downloading a release. |
| `DEPLOY_DIR` | empty | Local deploy template directory when using `SDGATE_BIN_PATH`. |

Install from a local binary:

```bash
sudo env \
  SDGATE_BIN_PATH=/path/to/sdgate \
  DEPLOY_DIR=/path/to/deploy \
  INSTALL_SYSTEMD=1 \
  scripts/install.sh
```

---

## Upgrade

For systemd installs, rerun the installer:

```bash
curl -fsSL https://raw.githubusercontent.com/huwenlong92/sdgate-nginx/main/scripts/install.sh | \
  sudo env INSTALL_SYSTEMD=1 VERSION=0.1.0 sh
```

For binary-only installs:

```bash
VERSION=0.1.0 curl -fsSL https://raw.githubusercontent.com/huwenlong92/sdgate-nginx/main/scripts/install.sh | sh
```

Existing config and SQLite data are kept.

---

## Build From Source

Install Web dependencies:

```bash
make install-web
```

Build the Web UI and Rust binary:

```bash
make build
```

Run the release binary with the checked-in local config:

```bash
make run
```

Build output:

```text
target/release/sdgate
static-dist/
```

The Vue application is built into `static-dist/` and embedded into the Rust binary.

---

## Packaging

Build the current platform release tarball:

```bash
make release-current-dry-run
```

Build Linux amd64 with Docker:

```bash
make release-linux-dry-run
```

Publish current platform:

```bash
make release-current
```

Publish Linux package:

```bash
make release-linux
```

Build both current platform and Linux packages without uploading:

```bash
make release-dry-run
```

Full release:

```bash
make release
```

The Linux release uses Docker because macOS cannot produce the Linux binary directly.

If the local `sdkit-rs` path dependency is not at the default relative location, set:

```bash
SDKIT_RS_DIR=/path/to/sdkit-rs make release-linux
```

Release tarballs contain:

```text
sdgate
README.md
```

Deployment templates live in:

```text
deploy/
├── Dockerfile.release
├── nginx/sdgate.conf
├── sdgate/config.toml
├── systemd/sdgate.service.tpl
└── release/
```

---

## Public README Sync

Sync public GitHub README/install/deploy files:

```bash
make sync-github-public
```

Alias:

```bash
make publish-readme
```

---

## Notes

- SDKit Gate manages its own Nginx configuration root and does not require modifying the user's existing system `nginx.conf`.
- Existing external Nginx files are treated as read-only import sources.
- Change the default admin password after first login.
- Keep `/etc/sdgate/config.toml` and `/var/lib/sdgate/sdgate.db` during upgrades.
