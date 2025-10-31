![Banner: Lightsail Deployer — GitHub Action for AWS Lightsail. rsync/ssh deploys, dry‑run mode, secrets-based.](docs/banner.svg)

# Lightsail Deployer

![CI](https://github.com/jamesbregenzer/aws-lightsail-deployer/actions/workflows/ci.yml/badge.svg?branch=main)
![Release](https://img.shields.io/github/v/release/jamesbregenzer/aws-lightsail-deployer?display_name=tag)
![Last commit](https://img.shields.io/github/last-commit/jamesbregenzer/aws-lightsail-deployer)
![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)
![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
[![Maintained by James Bregenzer](https://img.shields.io/badge/maintained%20by-James%20Bregenzer-F5C518?labelColor=000000)](https://jamesbregenzer.com)

**Composite GitHub Action** for deploying a project to **AWS Lightsail** via **SSH/rsync**.
Optimized for small PHP/WordPress apps, but generic enough for static and Node stacks.

**Author:** James Bregenzer — Full‑Stack Developer · https://jamesbregenzer.com  
**License:** MIT

---

## Quick start

1. **Add repo secrets** (Settings → Secrets and variables → Actions → New repository secret):
   - `AWS_HOST` — Lightsail public IP or hostname
   - `AWS_USER` — SSH user (e.g., `ubuntu`)
   - `AWS_KEY` — private key contents (PEM)
   - `DEPLOY_PATH` — absolute path on server (e.g., `/var/www/html/app`)

2. **Use the example workflow** from `.github/workflows/deploy.yml` (below). Start with `dry-run: true`.

3. **Push to main** or run via **workflow_dispatch** to deploy.

## Example workflow (`.github/workflows/deploy.yml`)

```yaml
name: Deploy to Lightsail

on:
  workflow_dispatch:
    inputs:
      dry-run:
        description: "Simulate without uploading"
        type: boolean
        default: true
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup SSH key
        run: |
          echo "${{ secrets.AWS_KEY }}" > key.pem
          chmod 600 key.pem

      - name: Deploy via rsync
        uses: ./.github/actions/lightsail-deployer
        with:
          host: ${{ secrets.AWS_HOST }}
          user: ${{ secrets.AWS_USER }}
          key: key.pem
          path: ${{ secrets.DEPLOY_PATH }}
          source: ./
          dry-run: ${{ github.event.inputs.dry-run || 'false' }}
          exclude: |
            .git
            .github
            node_modules
            vendor
            *.log
```

## Inputs

| Name      | Required | Default | Description |
|-----------|----------|---------|-------------|
| `host`    | yes      | —       | Server host/IP |
| `user`    | yes      | —       | SSH username |
| `key`     | yes      | —       | Path to private key file on runner |
| `path`    | yes      | —       | Remote destination path |
| `source`  | no       | `./`    | Local source path |
| `dry-run` | no       | `false` | Pass `--dry-run` to rsync |
| `exclude` | no       | —       | Newline-delimited exclude globs |

## Notes & tips
- Uses `rsync -az --delete` with SSH. Adjust flags for your stack.
- For **zero downtime** deploys: sync to a **releases/** dir and atomically switch a `current` symlink.
- Consider `composer install --no-dev --optimize-autoloader` either on CI or as a remote post‑deploy.

## Roadmap
See [ROADMAP](./ROADMAP.md).
