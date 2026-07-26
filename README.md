# homelab

Docker Compose stacks for my home server **odin**, deployed by [Komodo](https://komo.do).

## Layout

```
stacks/<name>/compose.yml    # the stack
stacks/<name>/env.komodo     # what to paste into Komodo's Environment field
komodo/                      # Komodo itself — bootstrap only, NOT managed by Komodo
```

Files are named `compose.yml`, not `compose.yaml`. Komodo defaults its File Paths field to
`compose.yml`, so matching it means the field can be left alone on every stack.

## How deploys work

Komodo clones this repo to `/opt/stacks/<stack-name>/` on odin and runs compose from the
stack's `run_directory` inside that clone. **There is no working copy on the server** —
edit here, push, then deploy.

```
edit → push → Komodo UI → Deploy
```

Webhooks are not wired up: Komodo Core is LAN-only and GitHub cannot reach it. Deploys are
triggered manually from the UI for now.

## Secrets

No secret values live in this repo.

Each stack's `env.komodo` uses `[[NAME]]` references, which Komodo interpolates from its
Variables/Secrets store at deploy time and writes to a `.env` passed via `--env-file`.

- **Canonical copy: 1Password.**
- Working copy: Komodo Variables, backed up nightly to `/opt/komodo/backups` (14 days).

## The Komodo exception

`komodo/` holds the compose file for Komodo Core, Periphery and Mongo. It is version
controlled here but **deployed by hand** on odin from `/opt/docker-compose.yaml` — Komodo
cannot redeploy the project containing its own core without killing the deploying process,
and Mongo's password cannot be a secret stored in Mongo.

## Adding a stack

1. Create `stacks/<name>/compose.yml` and `env.komodo`. Pin the image to a tag that
   actually exists — check with `docker manifest inspect`, as not every project publishes
   rolling major tags.
2. Push.
3. In Komodo: new Stack → this repo → `run_directory: stacks/<name>` → paste the
   environment. Leave File Paths at its `compose.yml` default and Account at `None`
   (the repo is public).
4. Remove the service from the old monolith with
   `docker compose -f /opt/docker-compose.yaml --env-file /opt/docker-compose.env rm -sf <service>`,
   then deploy. Container names collide otherwise.
