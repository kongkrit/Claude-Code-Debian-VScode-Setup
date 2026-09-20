# Debian and VScode setup for Claude Code

[VScode Setup](#vscode-setup)

## Debian Setup — Required Packages

Build/run dependencies for the projects in this workspace.

- **OS:** Debian 13 (trixie)
- **Already present:** `docker`
- **Installed by the command below:** `git`, `gh`, Python toolchain, `curl`

### Install command

```bash
sudo apt install -y git gh python3 python3-venv python3-pip python3-dev build-essential curl
```

### What each package is for

| Package | Purpose |
|---|---|
| `git` | Version control. |
| `gh` | GitHub CLI — clone, push, PRs, releases. |
| `python3` | Interpreter. Most projects are Python (data analysis, backtesting, a CLI tool). |
| `python3-venv` | Virtual environments — **required** on Debian 13 (see caveat below). |
| `python3-pip` | Installs Python dependencies (pandas, numpy, matplotlib, openpyxl, requests, yfinance, python-dateutil, pytest). These ship as prebuilt wheels. |
| `python3-dev` | C headers for building any dependency that lacks a prebuilt wheel. |
| `build-essential` | C/C++ compiler + make, for the same source-build cases. Precautionary. |
| `curl` | Used by a data-fetch script to download source CSVs. |

### Docker multi-arch builds (one-time setup)

Docker is already installed, but building multi-arch images
(`linux/amd64` + `linux/arm64`) needs QEMU emulation plus a `docker-container`
buildx builder. Run once:

```bash
docker run --privileged --rm tonistiigi/binfmt --install all
docker buildx create --name multiarch --driver docker-container --bootstrap --use
```

- First line registers QEMU so the host can emulate the **non-native**
  architecture — `arm64` on an x86_64 host, `amd64` on an Apple Silicon /
  arm64 host. `--install all` covers either, so the command is host-agnostic.
- Second line creates and activates a buildx builder for multi-platform
  output. Reused by every later `docker buildx build`.

> **Apple Silicon + OrbStack:** cross-arch emulation is built in (amd64 runs
> automatically via Rosetta), so the first line is usually unnecessary — the
> `buildx create` line is all you need.

### Not required

- **node / npm** — the one web project is a static PWA with a vendored `.wasm` binary; there is no build step. Serve it locally with `python3 -m http.server`.
- **.NET SDK** — no .NET projects are present.
- **docker** — already installed; it covers any project shipping a `Dockerfile`.
- Document-only folders (PDF / DOCX / HTML) need nothing to build.

### Important: Debian 13 is externally managed (PEP 668)

A system-wide `pip install` is blocked. Use a virtual environment per project:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt      # or: pip install -e ".[dev]"
```

Optional: `sudo apt install -y pipx` to install a packaged CLI tool cleanly
(`pipx install .`) without managing a venv by hand.

## VScode setup

- Create a new profile if needed.
- (Locally) install Microsoft's `Remote - SSH` and `Remote Explorer` extensions.
- Connect to remote host:
- Install Anthropic's `Claude Code for VS Code` extension.
- Check that Anthropic's extension is installed remotely.
