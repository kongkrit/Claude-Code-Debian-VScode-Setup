# Debian and VScode setup for Claude Code

[VScode Setup](#vscode-setup)

## Debian Setup — Required Packages

Build/run dependencies for the projects in this workspace.

- **OS:** Debian 13 (trixie)
- **Already present:** `docker`
- **Installed by the command below:** `git`, `gh`, `python3` toolchain, `curl`, `jq`, `wget`
- **Installed separately:** `uv` (not packaged for Debian; see [Install uv](#install-uv))

### Install command

```bash
sudo apt install -y git gh python3 python3-dev build-essential curl jq wget
```

### What each package is for

| Package | Purpose |
|---|---|
| `git` | Version control. |
| `gh` | GitHub CLI — clone, push, PRs, releases. |
| `python3` | Interpreter. Most projects are Python (data analysis, backtesting, a CLI tool). Dependencies (pandas, numpy, matplotlib, openpyxl, requests, yfinance, python-dateutil, pytest) are installed per project with `uv` (see below) and ship as prebuilt wheels. |
| `python3-dev` | C headers for building any dependency that lacks a prebuilt wheel. |
| `build-essential` | C/C++ compiler + make, for the same source-build cases. Precautionary. |
| `curl` | Used by a data-fetch script to download source CSVs, and by the `uv` installer below. |
| `jq` | Command-line JSON processor: parses, filters, and reshapes JSON (e.g., API responses or `gh ... --json` output) in shell scripts. |
| `wget` | Non-interactive file downloader: handles large or interrupted downloads (`-c` resumes) and recursive fetches. Overlaps with `curl`. Some install scripts call it specifically. |

### Install uv

`uv` is not packaged for Debian; install it with the standalone installer (needs `curl` from above). It puts `uv` and `uvx` in `~/.local/bin`, which Debian's default `~/.profile` adds to `PATH` on next login:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Verify: `git --version && gh --version && curl --version && uv --version`.

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

### Python: use uv only

Use `uv` for every venv and package install, not `pip`, `python3 -m venv`, or `pipx` (Debian 13 also blocks system-wide `pip install`, PEP 668). One venv per project:

```bash
uv sync                                          # project with a pyproject.toml; add packages with: uv add <pkg>
uv venv && uv pip install -r requirements.txt    # project with only a requirements.txt
uv run <cmd>                                     # run a command in the project venv
```

For a packaged CLI, `uv tool install .` avoids managing a venv by hand.

## VScode setup

- Create a new profile if needed.
- (Locally) install Microsoft's `Remote - SSH` and `Remote Explorer` extensions.
- Connect to remote host:
- Install Anthropic's `Claude Code for VS Code` extension.
- Check that Anthropic's extension is installed remotely.
