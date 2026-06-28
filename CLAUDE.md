# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **reprepro-managed Debian APT package repository** served via GitHub Pages at `jasonking3.github.io/macipgw-deb`. It hosts a custom Linux kernel package (`linux-image-4.15.18-macipgw`) for the macipgw IP-over-AppleTalk gateway. There is no source code here — only repository metadata, package files, and GPG signing infrastructure.

## Repository Structure

- `conf/distributions` — reprepro configuration: codename (`buster`), architectures (`amd64 armhf`), component (`contrib`), and signing key fingerprint
- `pool/` — actual `.deb` files organized by component/source package
- `dists/` — generated index files (`Packages`, `Release`, `InRelease`, `Release.gpg`) — **do not edit manually**
- `db/` — reprepro internal database — **do not edit manually**
- `PUBLIC.KEY` — the GPG public key consumers use to verify packages (key ID `7531F0FCF5D3360718F086F6BEEA91EFF8C7639A`)

## Common reprepro Commands

All commands must be run from the repository root.

```bash
# Add a new .deb package
reprepro includedeb buster /path/to/package.deb

# List packages in the repo
reprepro list buster

# Remove a package
reprepro remove buster linux-image-4.15.18-macipgw

# Re-export/rebuild all index files (e.g. after manual changes)
reprepro export buster

# Check repository integrity
reprepro check buster
```

## GPG Signing

All releases are signed with key `7531F0FCF5D3360718F086F6BEEA91EFF8C7639A`. The private key must be present in the local GPG keyring for reprepro to sign on `includedeb` or `export`. If the key is on a separate machine or smartcard, import it before running reprepro.

Consumers add this repo with:
```bash
curl -fsSL https://jasonking3.github.io/macipgw-deb/PUBLIC.KEY | sudo apt-key add -
echo "deb https://jasonking3.github.io/macipgw-deb buster contrib" | sudo tee /etc/apt/sources.list.d/macipgw.list
sudo apt update
```

## Building Packages

Packages are built in the companion repo [macipgw-kernel](https://github.com/jasonking3/macipgw-kernel), which contains the kernel config, patches, and GitHub Actions workflow. The workflow pushes finished `.deb` files into this repo via a deploy key.

## Adding a New Package Version (manual)

1. Build the `.deb` externally (kernel build environment required)
2. `reprepro includedeb buster /path/to/new.deb` — this updates `pool/`, regenerates `dists/`, and signs the release
3. Commit and push — GitHub Pages serves the result automatically

## Architecture Notes

- Only `amd64` currently has a package; `armhf` entries exist in the index but the Packages file is empty
- The `contrib` component is used (not `main`) because the kernel is built from patched upstream sources
- `conf/distributions` controls what reprepro accepts; update it before adding new architectures or codenames
