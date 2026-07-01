---
name: glance-widgets
description: >-
  Reference for every built-in Glance widget used inside page columns of
  glance.yml. Covers content widgets (rss, videos, hacker-news, lobsters,
  reddit, releases, repository, change-detection, markets, twitch-channels,
  twitch-top-games), utility widgets (search, weather, clock, calendar,
  calendar-legacy, to-do, bookmarks, iframe, html), monitoring widgets
  (monitor, dns-stats, server-stats, docker-containers), and layout widgets
  (group, split-column) including their properties, styles, and YAML anchor
  sharing. Load this when adding, configuring, or troubleshooting any widget
  except custom-api (use glance-custom-api for that).
---

# Glance widgets

Widgets are listed under a column's `widgets:` array. Every widget needs a
`type` and shares the common properties (`title`, `title-url`, `hide-header`,
`cache`, `css-class`) documented in `glance-config-basics`.

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets:
          - type: weather
            location: London, United Kingdom
```

Not every widget fits every column size; many offer a `style` property to adapt.

## Widget index
- **Content**: `rss`, `videos`, `hacker-news`, `lobsters`, `reddit`,
  `releases`, `repository`, `change-detection`, `markets`, `twitch-channels`,
  `twitch-top-games`
- **Utility**: `search`, `weather`, `clock`, `calendar`, `calendar-legacy`,
  `to-do`, `bookmarks`, `iframe`, `html`, `extension`
- **Monitoring**: `monitor`, `dns-stats`, `server-stats`, `docker-containers`
- **Layout**: `group`, `split-column`
- **Custom**: `custom-api` — see the `glance-custom-api` skill

## Full property reference
Detailed properties, styles, and examples for every widget are in
[references/widgets.md](references/widgets.md). Consult it before configuring a
widget so options and defaults are correct. Key cross-cutting notes below.

## Layout widgets (quick reference)

### group
Combine multiple widgets into one tabbed widget. Cannot nest a `group` or
`split-column` inside a `group`.

```yaml
- type: group
  widgets:
    - type: reddit
      subreddit: gamingnews
    - type: reddit
      subreddit: games
```

Share repeated properties with YAML anchors:

```yaml
- type: group
  define: &shared
      type: reddit
      show-thumbnails: true
      collapse-after: 6
  widgets:
    - { subreddit: gamingnews, <<: *shared }
    - { subreddit: games,      <<: *shared }
```

### split-column
Split a `full` column into side-by-side sub-columns (collapses to one column on
mobile). Use `max-columns` for 3/4/5-column and masonry layouts. You can nest a
`group` inside a `split-column`, but not a `split-column` inside a `group`.

```yaml
- size: full
  widgets:
    - type: split-column
      max-columns: 3
      widgets:
        - { type: reddit, subreddit: selfhosted, collapse-after: 15 }
        - { type: reddit, subreddit: homelab,    collapse-after: 15 }
        - { type: reddit, subreddit: sysadmin,   collapse-after: 15 }
```

## Common conventions
- **`collapse-after`** — number of items before a "SHOW MORE" button; `-1`
  never collapses (widget-dependent).
- **`limit`** — maximum number of items to fetch/show.
- **`style`** — switches layout; valid values vary per widget (see reference).
- **`icon`** — see Icons in `glance-config-basics` (`si:`, `sh:`, `di:`,
  `mdi:`, URL, or `auto-invert ...`).
- **Tokens/secrets** — prefer `${ENV_VAR}` / `${secret:name}` over hardcoding
  (GitHub/GitLab tokens, Reddit app auth, DNS passwords, etc.).
- **VPS + Reddit** — Reddit blocks VPS IPs (403); use `app-auth`, `proxy`, or
  `request-url-template`.
- **docker-containers** — requires mounting `/var/run/docker.sock`; configure
  via container labels (`glance.name`, `glance.icon`, `glance.url`, etc.) or the
  `containers` map.
- **server-stats remote** — requires the Glance Agent running on the target
  server.
