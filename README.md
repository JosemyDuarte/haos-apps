# Josemy's Home Assistant Apps

A custom [Home Assistant](https://www.home-assistant.io) add-on repository.

Add this repository to your Home Assistant instance to install the add-ons below
directly from the add-on store.

## Add-ons in this repository

| Add-on | Description |
|--------|-------------|
| [**Inker**](./inker) | Self-hosted e-ink device management server for [TRMNL](https://usetrmnl.com/) and BYOD displays. Wraps the [`JosemyDuarte/inker`](https://github.com/JosemyDuarte/inker) fork image (`ghcr.io/josemyduarte/inker`, **aarch64 / Raspberry Pi**). |

---

## Installing this repository

### Option A — one click

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FJosemyDuarte%2Fhaos-apps)

### Option B — manual

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store**.
2. Open the **⋮** menu (top-right) → **Repositories**.
3. Paste the repository URL and click **Add**:

   ```
   https://github.com/JosemyDuarte/haos-apps
   ```
4. Close the dialog. The add-ons from this repository now appear in the store
   (you may need to refresh / **Check for updates**).

> **Requirement:** Home Assistant OS or Supervised (the add-on store is provided
> by the Supervisor) on **`aarch64` hardware** (e.g. a Raspberry Pi 4/5). Add-ons
> are **not** available on Home Assistant Container or Core installs.

---

## Installing the Inker add-on

1. After adding the repository, open the **Inker** add-on from the store and
   click **Install** (the Supervisor builds a thin image on top of the
   `ghcr.io/josemyduarte/inker` fork image — this is quick).
2. Open the **Configuration** tab and set your options (see below), then **Save**.
3. Click **Start**.
4. Open the web UI from the add-on's **OPEN WEB UI** button (it opens the direct
   port — see below).

### Configuration

| Option | Default | Description |
|--------|---------|-------------|
| `admin_pin` | `1111` | PIN to log in to the Inker web UI. **Change it.** |
| `timezone` | `UTC` | Timezone for clock/date widgets (e.g. `Europe/Madrid`). |
| `log_level` | `info` | `trace` / `debug` / `info` / `warning` / `error`. |

### How the app is reached

Both the browser dashboard and e-ink devices use the **same directly-exposed
port** (default host port **8080**, set/observe it in the add-on's **Network**
tab):

```
http://<your-HA-IP>:8080
```

> **Ingress is disabled on purpose.** Inker's web UI is built to be served from
> the origin root (absolute `/assets/…` paths, absolute `/api` base, client-side
> routing with no base path), so it does not render under Home Assistant's
> Ingress subpath (`/api/hassio_ingress/<token>/`). The direct port is the
> supported way in — and TRMNL/BYOD devices need it anyway (they cannot
> authenticate through Ingress).

### Data & backups

All state (PostgreSQL database under `db/`, uploads under `uploads/`) is stored
in the add-on's persistent `/data` volume and is included in Home Assistant
backups. The Redis dependency from upstream is **not** used by this fork. `db/`
is excluded from backups by default — see [`inker/DOCS.md`](./inker/DOCS.md).

### Known limitations

- **`aarch64` only.** The published image (`ghcr.io/josemyduarte/inker`) is built
  for `arm64` only, so this add-on runs on a Raspberry Pi 4/5 but **not** on
  `amd64`/x86 hardware. Supporting both would need a multi-arch image.
- **No Ingress** — the dashboard is reached via the direct port (above).

---

## Updating to a new fork version

This add-on is a thin wrapper around the fork image, so tracking a new release is
a **two-line change**.

When [`JosemyDuarte/inker`](https://github.com/JosemyDuarte/inker) publishes a new
tag (`X.Y.Z`, pushed to [`ghcr.io/josemyduarte/inker`](https://github.com/JosemyDuarte/inker/pkgs/container/inker)):

1. Edit [`inker/build.yaml`](./inker/build.yaml) — bump the image tag:

   ```yaml
   build_from:
     aarch64: ghcr.io/josemyduarte/inker:X.Y.Z
   ```
2. Edit [`inker/config.yaml`](./inker/config.yaml) — match the version:

   ```yaml
   version: "X.Y.Z"
   ```
3. Commit and push. In Home Assistant, **Check for updates** in the add-on store;
   the Inker add-on will show an update. Click **Update**.

> Reconcile the glue in `inker/rootfs/etc/cont-init.d/` **only if** a new fork
> release changes one of the things it depends on:
> data paths (`/var/lib/postgresql/17/main`, `/app/uploads`), internal ports
> (nginx `80`, backend `3002`), env var names (`ADMIN_PIN`, `TZ`, `LOG_LEVEL`),
> the non-root user (`inker`), or the s6 container-environment path
> (`/run/s6/container_environment`). If none moved, the version bump is the whole
> update.

### Automating the bump

This repository ships a [Renovate](https://docs.renovatebot.com/) config at
[`.github/renovate.json`](./.github/renovate.json). It watches the
`ghcr.io/josemyduarte/inker` Docker tag and bumps both `inker/build.yaml` and
`inker/config.yaml` automatically.

To activate it, enable the [Renovate GitHub App](https://github.com/apps/renovate)
on this repository. After that, each fork release becomes an auto-opened pull
request — review, merge, then **Update** in Home Assistant.

---

## How this add-on is built

`inker/` contains:

```
inker/
├── config.yaml        # add-on manifest: ports, options + schema (ingress off)
├── build.yaml         # base image = the pinned ghcr.io/josemyduarte/inker tag
├── Dockerfile         # FROM fork image + COPY rootfs (no apt — see note)
├── DOCS.md            # the in-Home-Assistant Documentation tab
├── icon.png           # store/sidebar icon
├── logo.png           # store logo
├── translations/      # UI labels for the Configuration form
└── rootfs/etc/cont-init.d/
    ├── 00-ha-relocate # relocate Postgres + uploads into the single /data volume
    └── 01-ha-options  # map HA options (/data/options.json) → container env (via node)
```

The fork image is s6-overlay based (`ENTRYPOINT ["/init"]`), so the wrapper sets
`init: false` and adds two `cont-init.d` scripts that run **before** the image's
own `01-postgres-init` / `02-app-init`. The wrapper deliberately avoids
`apt-get` (the image removed its apt lists) and parses options with the `node`
runtime already present in the image.

The boot path, the single-`/data` relocation (Postgres cluster + uploads),
upload-dir writability by the non-root `inker` user, the options→env mapping, a
live backend response, and restart persistence were all verified by running the
wrapper standalone under Docker on native `arm64` before publishing.
</content>
