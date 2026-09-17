# Simhive in Docker

[Simhive](https://simhive.app) is a free simming network for World of
Warcraft: everyone's sims run on everyone's machines. Most people use the
desktop app; this is for a server, a NAS or a spare Linux box — one container,
one switch:

| `SIMHIVE_INTERFACE` | you get |
|---|---|
| `true` | your own Localbots page on port 4747 (sims go to the pool) **and** this machine's cores lent to the pool |
| `false` | cores only — nothing served, the machine just works for the pool |

Nothing to compile: SimulationCraft is fetched from the release channel at
the exact commit the pool runs (35 MB at first start) and follows the pool
from then on.

## Install

You need Docker with the Compose plugin (`curl -fsSL https://get.docker.com | sh` on a fresh Debian/Ubuntu) on x86-64 Linux, and a member token — one per person, the same on every machine you own: https://simhive.app/join (or in the app: Settings → Pool → Get a token).

```bash
git clone https://github.com/rbardtke/simhive-docker.git && cd simhive-docker
cp .env.example .env        # put your token in it; set SIMHIVE_INTERFACE=false for cores only
docker compose up -d
```

With the interface on, open `http://<this machine>:4747`. The first start
also fetches the game tables (60 MB); the page says so until they are there.

Without git: download [compose.yml](compose.yml) and [.env.example](.env.example)
into a folder, same steps.

## Managing it

- **Threads, schedule, pause** — Settings → Pool, on this page or in the app
  on any machine of yours: every machine with your token is listed and editable
  there. A headless machine is managed the same way.
- **Update** — `docker compose pull && docker compose up -d`. The image is
  rebuilt on every engine change; the pool's version column says "behind"
  when a machine should do this.
- **simc** needs nothing from you: when the pool pins a new build, the running
  container fetches it and reconnects.
- **Stop / start** — `docker compose down` / `docker compose up -d`. The
  volumes keep simc, the game tables, your saved sims and this machine's
  settings across that.
- **Keep cores for yourself** — the thread slider in Settings → Pool, or
  `cpuset:` in `compose.yml` for a hard limit.

## On a NAS (Unraid, Synology, Portainer, …)

Paste `compose.yml` into the stack editor and set `SIMHIVE_TOKEN` (and
`SIMHIVE_INTERFACE`) as environment variables instead of the `.env` file.

## Safety

Other people's characters get simmed on your machine. The engine never lets a
sim ask SimulationCraft to touch files or the network; `compose.yml` also
keeps it from being able to — read-only filesystem apart from the volumes,
`/tmp` and the job directory; no capabilities; no privilege escalation; a
process and memory cap. The interface has no login: keep the port on your
LAN — never expose it to the internet as is.

## What's what

- `ghcr.io/rbardtke/simhive` — the image: the Simhive engine (a fork of
  [Localbots](https://github.com/balovich-matje/localbots), MIT) on Node,
  built from the engine repository on every change
- [rbardtke/simc-builds](https://github.com/rbardtke/simc-builds) — the
  SimulationCraft builds it fetches (unmodified simc, GPLv3, one release per
  commit, checksums in every release)
- [rbardtke/simhive-releases](https://github.com/rbardtke/simhive-releases) —
  the desktop app
