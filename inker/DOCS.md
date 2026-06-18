# Home Assistant Add-on: Inker

[Inker](https://github.com/usetrmnl/inker) is a self-hosted e-ink device
management server for the homelab. It works with
[TRMNL](https://usetrmnl.com/) devices and any BYOD e-ink display: design
screens, build widgets with live data from your local network, and manage your
displays from a web interface.

This add-on packages the [`JosemyDuarte/inker`](https://github.com/JosemyDuarte/inker)
fork, published as a single-arch **`aarch64`** image to
`ghcr.io/josemyduarte/inker`. The image bundles PostgreSQL, the NestJS backend,
nginx, and the web UI (the fork drops the Redis dependency that upstream uses).

## Installation

1. Add this add-on repository to your Home Assistant instance (see the
   [repository README](https://github.com/JosemyDuarte/haos-apps)).
2. Find the "Inker" add-on in the store and click **Install**.
3. Open the **Configuration** tab, set your options, and click **Save**.
4. Start the add-on on the **Info** tab.
5. Check the **Log** tab to confirm a clean startup.
6. Click **OPEN WEB UI** to open the dashboard (this opens the direct port — see
   *Accessing Inker* below).

## Configuration

Example configuration:

```yaml
admin_pin: "1111"
timezone: Europe/Madrid
log_level: info
```

### Option: `admin_pin`

The PIN used to log in to the Inker web interface. **Change it from the default**
`1111`.

### Option: `timezone`

Timezone used by clock/date widgets, in TZ database form (for example
`Europe/Madrid`, `America/New_York`). Defaults to `UTC`.

### Option: `log_level`

Controls how verbose the add-on log is. One of `trace`, `debug`, `info`
(default), `warning`, `error`. Raise it to `debug` when troubleshooting.

## Accessing Inker

Both the browser dashboard and your e-ink devices use the **same directly
exposed port**. In the add-on's **Network** tab you'll find the host port mapped
to the container's port `80` (default `8080`). Open the dashboard, and point
your devices, at:

```
http://<your-home-assistant-ip>:8080
```

The **OPEN WEB UI** button on the add-on's Info page opens this URL for you. To
change the port, edit it in the **Network** tab and restart the add-on.

> **Why not Ingress?** Ingress is intentionally **disabled**. Inker's web UI is
> built to be served from the origin root: it uses absolute asset paths
> (`/assets/…`), an absolute API base (`/api`), and client-side routing with no
> base path. Home Assistant Ingress serves add-ons under a subpath
> (`/api/hassio_ingress/<token>/`), so the dashboard's assets would 404 there and
> the page would not render. Until the app supports a configurable base path, the
> direct port is the supported way in — and TRMNL/BYOD devices need it anyway,
> since they cannot authenticate through Ingress.

## Data and backups

All persistent state — the PostgreSQL database and uploaded assets — is stored
in the add-on's `/data` volume (`db/` and `uploads/`) and is captured by Home
Assistant backups. The PostgreSQL data directory (`db/`) is excluded from
backups by default (`backup_exclude`) because it is large and a raw file copy of
a live cluster is not crash-consistent; if you rely on backups for disaster
recovery, consider a database dump instead.

## Known limitations

- **Architecture: `aarch64` only.** The published image (`ghcr.io/josemyduarte/inker`)
  is built for `arm64` only — ideal for a Raspberry Pi 4/5 running Home Assistant
  OS, but it does **not** run on `amd64`/x86 hardware. (A multi-arch image would
  be needed to support both; the fork currently publishes arm64 only.)
- **No Ingress.** The dashboard is reached via the direct port, not the Home
  Assistant sidebar — see *Accessing Inker*.

## Support

- Add-on / packaging issues: <https://github.com/JosemyDuarte/haos-apps/issues>
- Inker fork (the published image): <https://github.com/JosemyDuarte/inker>
- Inker upstream (the application): <https://github.com/usetrmnl/inker>
</content>
