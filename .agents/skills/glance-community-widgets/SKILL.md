---
name: glance-community-widgets
description: >-
  Guide to the community-maintained Glance widgets collection
  (github.com/glanceapp/community-widgets): a catalog of custom-api and
  extension widgets contributed by the community for services like Immich,
  Jellyfin, Sonarr/Radarr, Proxmox, Steam, Tailscale, sports scores, and many
  more. Covers how to find and install a community widget (copy-paste vs
  $include for large widgets), the difference between custom-api and extension
  community widgets, and the maintenance/safety caveats. Load this when the
  user wants a ready-made widget for a specific third-party service rather than
  writing a custom-api template from scratch.
---

# Glance community widgets

The [Community Widgets](https://github.com/glanceapp/community-widgets)
repository is a collection of ready-made widgets for
[Glance](https://github.com/glanceapp/glance), built by the community using the
`custom-api` and `extension` widget types. Use it to find a pre-built widget for
a specific third-party service instead of authoring a `custom-api` template from
scratch.

- To build your own template, use the `glance-custom-api` skill.
- For built-in widget reference, use the `glance-widgets` skill.

## Two kinds of community widget

| | custom-api widgets | extension widgets |
| - | ------------------ | ----------------- |
| Setup | Copy-paste YAML into your config | Run a separate server / Docker container |
| Effort | Low — usually just paste + env vars | Higher — deploy the extension backend |
| Vetting | Vetted by Glance maintainers (safe, but may have bugs / visual/perf quirks) | **Not** monitored by Glance maintainers — use at your own risk |

## Finding a widget
Widgets are listed in the repo's README, grouped into **Custom API Widgets**
(with a "Newly added" and an "All" section) and **Extension Widgets**. Each
entry links to `widgets/<name>/README.md` (custom-api) or an external repo
(extension), and names the author.

A non-exhaustive sense of what's available: media servers (Immich, Jellyfin,
Emby, Plex/Tautulli, Audiobookshelf, Kavita, Komga, Calibre-Web), *arr stack
(Sonarr/Radarr/Lidarr, Prowlarr, Overseerr, SABnzbd, NZBGet, qBittorrent),
infrastructure (Proxmox VE/Backup Server, Portainer, Docker Swarm, TrueNAS,
Synology, Beszel, Scrutiny, Gatus, Uptime Kuma, NetAlertX), networking/VPN
(Tailscale, Netbird, Gluetun, Mullvad, AzireVPN, Cloudflare Tunnels, NPM, Unifi,
NextDNS, Technitium), sports scores (F1, UFC, NFL, NBA, NHL, MLB, football/soccer,
AFL, NCAA CFB), and lots of misc (Steam, Spotify, Last.fm, ListenBrainz, Trakt,
GitHub, LeetCode, weather, Home Assistant, xkcd, chess/Go puzzles, etc.).

A full, mirrored index is in
[references/catalog.md](references/catalog.md). The live README is the source of
truth; fetch <https://github.com/glanceapp/community-widgets> if you need the
latest additions or a widget not in the mirror.

## Installing a widget

### Simple widgets — copy-paste
For short widgets, copy the YAML from the widget's `README.md` directly into a
column's `widgets:` array in `glance.yml`, then replace any URLs / API keys with
environment variables:

```yaml
widgets:
  - type: custom-api
    title: Immich stats
    cache: 1d
    url: https://${IMMICH_URL}/api/server/statistics
    headers:
      x-api-key: ${IMMICH_API_KEY}
    template: |
      ...
```

Prefer `${ENV_VAR}` / `${secret:name}` for anything sensitive so the config can
be committed safely (see `glance-config-basics`).

### Large widgets — use $include
Widgets that span hundreds of lines are hard to indent correctly inline. Put the
widget in its own file (values at the top level, no leading indentation) and pull
it in with `$include`:

```yaml
# glance.yml
widgets:
  - $include: immich-stats.yml
```

```yaml
# immich-stats.yml  (top-level, no extra indentation)
- type: custom-api
  title: Immich stats
  url: https://${IMMICH_URL}/api/server/statistics
  headers:
    x-api-key: ${IMMICH_API_KEY}
  template: |
    ...
```

The correct indentation is applied automatically based on where the file is
included (see the `$include` section in `glance-config-basics`).

### Extension widgets
Follow the linked external repo's instructions to run its server/container, then
add an `extension` widget pointing at it:

```yaml
- type: extension
  url: https://your-extension-host/widget
  # allow-potentially-dangerous-html: true   # only if the extension needs it AND you trust it
  parameters:
    key: value
```

See the `extension` entry in `glance-widgets` for properties. Only enable
`allow-potentially-dangerous-html` for extensions you fully trust.

## Safety & maintenance
- Widget maintenance is the responsibility of **each widget's author**, not the
  Glance maintainers — file issues/PRs on the widget's page or repo.
- `custom-api` community widgets are vetted by Glance maintainers (safe), but may
  still have bugs, visual inconsistencies, or performance issues.
- `extension` widgets are **not** monitored by maintainers — review the code and
  run them at your own risk.
- After adding a widget, set a sensible `cache` to avoid rate-limiting the
  upstream API on config reloads.
