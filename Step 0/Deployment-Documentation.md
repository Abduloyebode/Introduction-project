# Deployment Documentation

This document describes how the Coolify + Next.js deployment for this task was set up, so another developer could replicate it end-to-end.

## Overview

- **VM**: Ubuntu 26.04 server hosted on Hetzner (bare metal/VPS, accessed via SSH with username/password).
- **Deployment platform**: [Coolify](https://coolify.io/) v4.1.2, self-hosted on the VM.
- **App**: `demo-nextjs-app`, a Next.js demonstration application.
- **Live URL**: `https://uw752r62ktlsxq8p4y72rq97.89.167.4.27.sslip.io` (HTTPS via Coolify's built-in Let's Encrypt integration, using a free `sslip.io` wildcard DNS domain that resolves to the VM's IP — no custom domain was assigned for this task).

## 1. VM Access

Connected via SSH:
```
ssh <username>@<vm-ip>
```
Credentials were provided via a 1Password shared vault link (not stored in this repo — see the security note at the end).

## 2. Installing Coolify

Ran the official Coolify install script as root:
```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

**Issue hit: install hung indefinitely on Step 1/9 (package installation).**

- Root cause: the underlying `apt-get install` process got stuck holding the `dpkg` frontend lock — most likely due to an interactive `needrestart` prompt (asking which services to restart) that had nothing to answer it, since the script runs non-interactively via `curl | sudo bash`.
- Symptom: `apt-get`/`dpkg` process shown in `ps aux` with almost 0 CPU time used over an hour — i.e., blocked, not actually working.
- **Fix**:
  1. Identify the stuck process: `ps aux | grep apt-get`
  2. Kill it: `sudo kill -9 <pid>` (had to kill both the parent and child `apt-get` processes)
  3. Confirm the dpkg lock was released: `sudo lsof /var/lib/dpkg/lock-frontend` (should return nothing)
  4. Repair any partial package state: `sudo dpkg --configure -a`
  5. Set `needrestart` to non-interactive mode to prevent recurrence:
     ```bash
     echo '$nrconf{restart} = "a";' | sudo tee -a /etc/needrestart/needrestart.conf
     ```
  6. Re-ran the install script — it completed successfully in a few minutes, skipping already-installed packages.

Once complete, Coolify was accessible at:
```
http://<vm-ip>:8000
```
Created the admin account through the web UI on first visit.

**Note**: Coolify warns to back up `/data/coolify/source/.env` (contains its secrets/keys) to a password manager — do this outside of any git repo.

## 3. Connecting the Repository

The task's demo app repo (`CrazedBySerenity/demo-nextjs-app`) was only granted **read access**, so branches couldn't be created directly on it.

**Fix**: forked the repo to a personal GitHub account (`Abduloyebode/demo-nextjs-app`), and created the feature branch there instead:
```
feature/update-content
```

In Coolify:
- **New Resource → Applications → Public Repository**
- Repository URL: `https://github.com/Abduloyebode/demo-nextjs-app`
- Branch: `feature/update-content`
- Build Pack: **Nixpacks** (auto-detected Node/Next.js app)
- Base Directory: `/`
- Port: `3000`
- Static site: No (Next.js app runs as a Node server)

**Gotcha**: the branch dropdown on the initial "Create Application" screen didn't reliably pick up the newly created branch (likely a stale/rate-limited fetch from GitHub's public API). Workaround: create the application anyway, then go to **Configuration → Git Source** after creation — the branch field there is directly editable and reliably takes the correct branch name.

## 4. First Deployment & the "Bad Gateway" Issue

First deploy succeeded (build completed, container started), but visiting the app's URL returned **502 Bad Gateway**.

- **Root cause**: Next.js's "standalone" server build binds to whatever `process.env.HOSTNAME` is set to. Docker automatically sets `HOSTNAME` to the container's ID (visible in the logs: `Local: http://<container-id>:3000`), so the app was listening on that specific hostname instead of all interfaces — meaning Coolify's reverse proxy (Traefik) couldn't reach it.
- **Fix**: added an environment variable in Coolify (**Environment Variables** tab):
  ```
  HOSTNAME=0.0.0.0
  ```
  Redeployed — app logs then showed it listening correctly, and the Bad Gateway was resolved.

## 5. Enabling HTTPS

In the application's **General** settings, changed the Domain field from `http://...sslip.io` to `https://...sslip.io` and saved, then redeployed. `sslip.io` domains resolve directly to the VM's public IP, so Let's Encrypt's HTTP-01 challenge works without any custom DNS setup.

**Issues hit along the way:**

1. **"No available server"** (Traefik's own error page) right after switching to HTTPS — the router config only regenerates on a fresh deploy, so a plain Save wasn't enough. Fix: trigger a new **Deploy**, not just Save.
2. **Browser showed "Not secure" / a self-signed `TRAEFIK DEFAULT CERT`** — meaning the real Let's Encrypt certificate was never issued. Diagnosed by:
   - Confirming `ufw status` was `inactive` (not a local firewall issue).
   - Checking the proxy container's logs: `sudo docker logs coolify-proxy --tail 100`, which showed a `Router defined multiple times with different configurations` error — a stale routing entry left over from an earlier deployment/container.
   - Restarting the proxy to clear stale state: `sudo docker restart coolify-proxy`.
   - Confirmed the container's Traefik labels were correct (`certresolver: letsencrypt`) via `sudo docker inspect <container> | grep -i traefik`.
   - Confirmed the resolver's static config was correct via `sudo docker inspect coolify-proxy --format '{{range .Args}}{{println .}}{{end}}'` (HTTP-01 challenge on the `http` entrypoint, matching the open port 80).
   - Checked the actual certificate store: `sudo docker exec coolify-proxy cat /traefik/acme.json` — this confirmed a real, valid Let's Encrypt certificate had in fact been issued for the domain.
3. The earlier "Not secure" warning turned out to be a **stale browser cache** of the old self-signed cert from before the fix. Reloading in an incognito window showed the real certificate and a secure padlock.

## 6. Automatic Deployment on Push (Webhook)

Coolify's **Webhooks** tab exposes two different things — important not to confuse them:
- **Deploy Webhook** (`/api/v1/deploy?uuid=...`) — a manual trigger endpoint requiring a Bearer token in an `Authorization` header. GitHub's native webhook feature can't send custom auth headers, so this one isn't directly usable as a GitHub webhook.
- **Manual Git Webhooks → GitHub** (`/webhooks/source/github/events/manual`) — the correct one. Uses a shared secret for HMAC signature verification, which is exactly what GitHub's webhook UI supports natively.

**Setup**: copied the GitHub webhook URL and secret from Coolify, then in the forked repo (`Abduloyebode/demo-nextjs-app` → Settings → Webhooks → Add webhook): set the Payload URL to the Coolify URL, Content type to `application/json`, pasted in the secret, and left it set to trigger on push events.

**Verified working**: a direct commit to `feature/update-content` on GitHub triggered an automatic deployment in Coolify within seconds, with no manual "Deploy" click.

## 7. Local Development & Code Change

- Cloned the fork locally: `git clone https://github.com/Abduloyebode/demo-nextjs-app.git`, checked out `feature/update-content`.
- Installed dependencies with `npm install` and confirmed the app ran locally with `npm run dev` (`http://localhost:3000`).
- Opened the project in **Cursor** as the editor.
- Made a small, meaningful content change: updated the hero subheading copy on the homepage from "...moving work forward, and learning every week." to "...moving work forward, and celebrating progress every week."
- Committed and pushed only the intentional change — noticed `npm run dev` had also regenerated `next-env.d.ts` as a side effect; reverted that file before committing, since it was unrelated to the actual change.

## 8. Pull Request

Opened from the fork back to the upstream repo, per the standard GitHub contribution model for a repo where only read access was granted:

**`Abduloyebode:feature/update-content` → `CrazedBySerenity:main`**

Included: a summary of the change, testing notes (local verification + confirmation on the live deployed URL), and before/after screenshots of the homepage.

## Security Notes

- No passwords, SSH keys, tokens, webhook secrets, or the Coolify `.env` contents are included in this document or committed anywhere in this repo.
- VM credentials were shared via 1Password and used directly; not stored in git.
