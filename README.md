# Josemy's Home Assistant Apps

A custom [Home Assistant](https://www.home-assistant.io) add-on repository.

Add this repository to your Home Assistant instance to install the add-ons below
directly from the add-on store.

## Add-ons in this repository

| Add-on | Description |
|--------|-------------|
| [**Inker**](./inker) | Self-hosted e-ink device management server for [TRMNL](https://usetrmnl.com/) and BYOD displays. Wraps the upstream [`usetrmnl/inker`](https://github.com/usetrmnl/inker) image. |

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
> by the Supervisor). Add-ons are **not** available on Home Assistant Container
> or Core installs.

---

## Installing the Inker add-on

1. After adding the repository, open the **Inker** add-on from the store and
   click **Install** (the Supervisor builds a thin image on top of the upstream
   `wojooo/inker` image — this is quick).
2. Open the **Configuration** tab and set your options (see below), then **Save**.
3. Click **Start**.
4. Open the web UI from the sidebar (**Open Web UI** / the Inker panel).

### Configuration

| Option | Default | Description |
|--------|---------|-------------|
| `admin_pin` | `1111` | PIN to log in to the Inker web UI. **Change it.** |
| `timezone` | `UTC` | Timezone for clock/date widgets (e.g. `Europe/Madrid`). |
| `log_level` | `info` | `trace` / `debug` / `info` / `warning` / `error`. |

### Two ways the app is reached

- **Browser → Ingress.** The dashboard is served through the Home Assistant
  sidebar, protected by your Home Assistant login. No extra port needed.
- **E-ink devices → direct port.** TRMNL / BYOD devices cannot use Ingress (they
  have no Home Assistant session), so they connect to a directly-exposed port.
  Set/observe it in the add-on's **Network** tab (default host port **8080**);
  devices then talk to `http://<your-HA-IP>:8080`.

### Data & backups

All state (PostgreSQL database, Redis, uploads) is stored under the add-on's
persistent `/data` volume and is included in Home Assistant backups.

### Known limitations

- **`amd64` only.** The upstream image bundles x86-only components (headless
  Chrome, an x86 s6-overlay build), so this add-on does **not** run on
  `aarch64` / Raspberry Pi.

---

## Updating to a new upstream version

This add-on is a thin wrapper around the upstream image, so tracking a new
upstream release is a **two-line change**.

When [`usetrmnl/inker`](https://github.com/usetrmnl/inker) publishes a new tag
(`X.Y.Z`, mirrored to Docker Hub as [`wojooo/inker`](https://hub.docker.com/r/wojooo/inker)):

1. Edit [`inker/build.yaml`](./inker/build.yaml) — bump the image tag:

   ```yaml
   build_from:
     amd64: docker.io/wojooo/inker:X.Y.Z
   ```
2. Edit [`inker/config.yaml`](./inker/config.yaml) — match the version:

   ```yaml
   version: "X.Y.Z"
   ```
3. Commit and push. In Home Assistant, **Check for updates** in the add-on store;
   the Inker add-on will show an update. Click **Update**.

> Reconcile the glue in `inker/rootfs/etc/cont-init.d/` **only if** an upstream
> release changes one of the things it depends on:
> data paths (`/var/lib/postgresql/17/main`, `/app/uploads`, redis `--dir /data`),
> internal ports (nginx `80`, backend `3002`), env var names
> (`ADMIN_PIN`, `TZ`, `LOG_LEVEL`), or the s6 container-environment path
> (`/run/s6/container_environment`). If none moved, the version bump is the whole
> update.

### Automating the bump

This repository already ships a [Renovate](https://docs.renovatebot.com/) config
at [`.github/renovate.json`](./.github/renovate.json). It watches the
[`wojooo/inker`](https://hub.docker.com/r/wojooo/inker) Docker tag and bumps both
`inker/build.yaml` and `inker/config.yaml` automatically.

To activate it, enable the [Renovate GitHub App](https://github.com/apps/renovate)
on this repository. After that, each upstream release becomes an auto-opened pull
request — review, merge, then **Update** in Home Assistant.

---

## How this add-on is built

`inker/` contains:

```
inker/
├── config.yaml        # add-on manifest: ingress, ports, options + schema
├── build.yaml         # base image = the pinned upstream wojooo/inker tag
├── Dockerfile         # FROM upstream image + COPY rootfs (no apt — see note)
├── DOCS.md            # the in-Home-Assistant Documentation tab
├── icon.png           # store/sidebar icon
├── logo.png           # store logo
├── translations/      # UI labels for the Configuration form
└── rootfs/etc/cont-init.d/
    ├── 00-ha-relocate # relocate Postgres + uploads into the single /data volume
    └── 01-ha-options  # map HA options (/data/options.json) → container env (via node)
```

The wrapper deliberately avoids `apt-get` (the upstream image's package
dependency graph is pruned and cannot resolve installs) and parses options with
the `node` runtime already present in the image.

The boot path, the single-`/data` relocation, the options→env mapping, and the
healthy startup were verified by running the wrapper standalone under Docker
before publishing.
