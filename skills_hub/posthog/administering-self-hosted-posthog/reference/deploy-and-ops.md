# Deploy & ops (self-host, reference only)

Ops/devops detail kept out of the main skill. A PM rarely runs these; point engineering to the live docs under `/docs/self-host`.

## Table of contents
- [What self-hosting means](#what-self-hosting-means)
- [Requirements](#requirements)
- [Install / upgrade](#install--upgrade)
- [Instance settings & staff users](#instance-settings--staff-users)
- [Securing the instance](#securing-the-instance)
- [Email-enabled instance](#email-enabled-instance)

## What self-hosting means
You run/buy and manage your own infrastructure, choose URLs, and own all scaling and risk. The free Docker Compose ("hobby") deploy is **MIT-licensed, single-machine, unsupported, no SLAs, no recovery from data loss**, and unlikely to scale past a couple hundred thousand events without significant effort. PostHog does not do tagged releases for self-host; commits flow through CI/CD into the same images. New Kubernetes/Helm deployments are no longer supported. For most teams the docs recommend PostHog Cloud (US/EU) instead.

## Requirements
- Linux Ubuntu VM, roughly 4 vCPU / 16GB RAM / >30GB storage (e.g. a Hetzner box).
- An `A` DNS record pointing a domain at the instance; PostHog auto-issues a LetsEncrypt SSL cert.

## Install / upgrade
- Install: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/posthog/posthog/HEAD/bin/deploy-hobby)"` (prompts for DockerHub release tag + domain; waits ~5–10 min for migrations + TLS).
- Beta TUI: download `hobby-installer` from GitHub releases (`--ci --domain=...` for programmatic).
- Upgrade: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/posthog/posthog/HEAD/bin/upgrade-hobby)"` — **back up data first.** Keep up to date.
- Customize via `docker-compose.yml` env vars, then restart with docker-compose.
- Troubleshoot: `docker ps` (all containers up), `docker logs <container>` (start with `web` — runs migrations).
- Struggling server: increase instance size or migrate to PostHog Cloud.

## Instance settings & staff users
- Instance settings at `/instance/settings` (PostHog 1.33.0+); applied immediately across all pods. Runtime-behavior settings must use environment variables.
- **Staff users** = instance-level permission (independent of org/project roles) to manage instance-wide settings. First user of an instance (1.32.0+) is staff; manage others at `/instance/status/` > Staff users tab (1.34.0+) or via Django shell (`user.is_staff = True`).

## Securing the instance
You own all hardening. Key controls:
- `SECRET_KEY` — required (`openssl rand -hex 32`); PostHog won't work without your own.
- **Always run HTTPS/TLS.** `SECURE_COOKIES` defaults True in production — keep on for live data.
- Restrict admin access by IP: `ALLOWED_IP_BLOCKS` (doesn't apply to capture endpoints). Behind a proxy: `TRUSTED_PROXIES`, `TRUST_ALL_PROXIES`, `IS_BEHIND_PROXY`.
- **SSRF/egress**: Cloud routes outbound (webhooks/integrations) through Stripe's Smokescreen — **not part of self-host**. You must run your own egress proxy or firewall/VPC rules so private ranges and the cloud metadata service (`169.254.169.254`) are unreachable.

## Email-enabled instance
Configure SMTP (Settings/env per `/docs/self-host/configure/email`). Without it: invite emails, join notifications, password-reset emails, etc. won't send — share invite links manually. Many governance/notification features assume an email-enabled instance.
