# Home Assistant Add-on: Inker

[Inker](https://github.com/usetrmnl/inker) is a self-hosted e-ink device
management server for the homelab. It works with
[TRMNL](https://usetrmnl.com/) devices and any BYOD e-ink display: design
screens, build widgets with live data from your local network, and manage your
displays from a web interface.

This add-on packages the upstream `usetrmnl/inker` container (PostgreSQL, Redis,
the NestJS backend, and the web UI) for Home Assistant.

## Installation

1. Add this add-on repository to your Home Assistant instance (see the
   [repository README](https://github.com/JosemyDuarte/haos-apps)).
2. Find the "Inker" add-on in the store and click **Install**.
3. Open the **Configuration** tab, set your options, and click **Save**.
4. Start the add-on on the **Info** tab.
5. Check the **Log** tab to confirm a clean startup.
6. Click **OPEN WEB UI** to open the dashboard.

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

There are two ways to reach the app, by design:

### From a browser — Ingress

The dashboard is available through the Home Assistant sidebar (the Inker panel /
**OPEN WEB UI**). This route is protected by your Home Assistant login — no extra
port is exposed to your network.

### From e-ink devices — direct port

TRMNL and BYOD devices cannot authenticate through Home Assistant Ingress, so
they connect to a **directly exposed port** instead. In the add-on's **Network**
tab you'll find the host port mapped to the container's port `80` (default
`8080`). Point your devices at:

```
http://<your-home-assistant-ip>:8080
```

To change the port, edit it in the **Network** tab and restart the add-on.

## Data and backups

All persistent state — the PostgreSQL database, Redis data, and uploaded
assets — is stored in the add-on's `/data` volume and is captured by Home
Assistant backups. The PostgreSQL data directory (`db/`) is excluded from
backups by default (`backup_exclude`) because it is large; if you rely on
backups for disaster recovery, consider a database dump instead.

## Known limitations

- **Architecture: `amd64` only.** The upstream image bundles x86-only components
  (a headless Chrome build and an x86 s6-overlay), so this add-on does not run on
  `aarch64` hardware such as a Raspberry Pi.

## Support

- Add-on / packaging issues: <https://github.com/JosemyDuarte/haos-apps/issues>
- Inker itself (the application): <https://github.com/usetrmnl/inker>
