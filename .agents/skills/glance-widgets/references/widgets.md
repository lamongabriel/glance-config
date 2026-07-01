# Glance widgets — full reference

Detailed properties, styles, and examples for every built-in widget. All
widgets also support the shared properties: `type`, `title`, `title-url`,
`hide-header`, `cache`, `css-class`.

---

## RSS
Display articles from multiple RSS/Atom feeds.

```yaml
- type: rss
  title: News
  style: horizontal-cards
  feeds:
    - url: https://feeds.bloomberg.com/markets/news.rss
      title: Bloomberg
    - url: https://moxie.foxbusiness.com/google-publisher/technology.xml
      title: Fox Business
```

| Name | Type | Default |
| ---- | ---- | ------- |
| style | string | vertical-list |
| feeds | array (required) | |
| thumbnail-height | float | 10 |
| card-height | float | 27 |
| limit | integer | 25 |
| preserve-order | bool | false |
| single-line-titles | boolean | false |
| collapse-after | integer | 5 |

- `style`: `vertical-list` (full/small), `detailed-list` (full),
  `horizontal-cards` (full), `horizontal-cards-2` (full).
- `collapse-after`: `-1` never collapses.
- `preserve-order`: keep feed order; set per-feed `limit` if using many feeds.
- `single-line-titles`: truncate long titles (only `vertical-list`).
- `thumbnail-height`: rem, only for `horizontal-cards` (default 10).
- `card-height`: rem, only for `horizontal-cards-2` (default 27).

Per-feed properties:

| Name | Type | Default | Notes |
| ---- | ---- | ------- | ----- |
| url | string (required) | | |
| title | string | feed's title | |
| hide-categories | boolean | false | `detailed-list` only |
| hide-description | boolean | false | `detailed-list` only |
| limit | integer | | per-feed cap |
| item-link-prefix | string | | prefix each item link |
| headers | key/value | | request headers |

```yaml
- type: rss
  feeds:
    - url: https://domain.com/rss
      headers:
        User-Agent: Custom User Agent
```

---

## Videos
Latest videos from YouTube channels/playlists.

```yaml
- type: videos
  channels:
    - UCXuqSBlHAE6Xw-yeJA0Tunw
    - UCBJycsmduvYEL83R_U4JriQ
```

| Name | Type | Default |
| ---- | ---- | ------- |
| channels | array (required unless playlists) | |
| playlists | array | |
| limit | integer | 25 |
| style | string | horizontal-cards |
| collapse-after | integer | 7 |
| collapse-after-rows | integer | 4 |
| include-shorts | boolean | false |
| video-url-template | string | https://www.youtube.com/watch?v={VIDEO-ID} |

- `channels`: channel IDs (Channel page → description → Share channel → Copy
  channel ID).
- `playlists`: playlist IDs (from `...&list={ID}&...`).
- `style`: `horizontal-cards`, `vertical-list`, `grid-cards`.
- `collapse-after`: for `vertical-list`. `collapse-after-rows`: for `grid-cards`.
- `video-url-template`: `{VIDEO-ID}` placeholder (e.g. Invidious front-end).

---

## Hacker News
```yaml
- type: hacker-news
  limit: 15
  collapse-after: 5
```

| Name | Type | Default |
| ---- | ---- | ------- |
| limit | integer | 15 |
| collapse-after | integer | 5 |
| comments-url-template | string | https://news.ycombinator.com/item?id={POST-ID} |
| sort-by | string | top |
| extra-sort-by | string | |

- `sort-by`: `top`, `new`, `best`.
- `extra-sort-by`: only `engagement` (points+comments, favors recent).
- `comments-url-template`: `{POST-ID}` placeholder.

---

## Lobsters
```yaml
- type: lobsters
  sort-by: hot
  tags: [go, security, linux]
  limit: 15
  collapse-after: 5
```

| Name | Type | Default |
| ---- | ---- | ------- |
| instance-url | string | https://lobste.rs/ |
| custom-url | string | |
| limit | integer | 15 |
| collapse-after | integer | 5 |
| sort-by | string | hot |
| tags | array | |

- `custom-url`: if set, ignores `instance-url`, `sort-by`, `tags`.
- `sort-by`: `hot` or `new`. Filtering by `tags` forces `hot`.
- `collapse-after`: `-1` never collapses.

---

## Reddit
> Reddit blocks VPS IPs (403). Use `app-auth`, `proxy`, `request-url-template`,
> or a VPN.

```yaml
- type: reddit
  subreddit: technology
```

| Name | Type | Default |
| ---- | ---- | ------- |
| subreddit | string (required) | |
| style | string | vertical-list |
| show-thumbnails | boolean | false |
| show-flairs | boolean | false |
| limit | integer | 15 |
| collapse-after | integer | 5 |
| comments-url-template | string | https://www.reddit.com/{POST-PATH} |
| request-url-template | string | |
| proxy | string or object | |
| sort-by | string | hot |
| top-period | string | day |
| search | string | |
| extra-sort-by | string | |
| app-auth | object | |

- `style`: `vertical-list`, `horizontal-cards` (both full), `vertical-cards`
  (small).
- `show-thumbnails`: only `vertical-list`; some subreddits lack thumbnails.
- `collapse-after`: not available for `vertical-cards`/`horizontal-cards`.
- `comments-url-template` placeholders: `{POST-PATH}`, `{POST-ID}`, `{SUBREDDIT}`.
- `request-url-template`: `{REQUEST-URL}` placeholder for a proxy passthrough.
- `sort-by`: `hot`, `new`, `top`, `rising`. `top-period` (when `top`): `hour`,
  `day`, `week`, `month`, `year`, `all`.
- `extra-sort-by`: only `engagement`.

`proxy` object form:

```yaml
proxy:
  url: http://proxy.com:8080
  allow-insecure: true
  timeout: 10s
```

`app-auth` (register at https://ssl.reddit.com/prefs/apps/):

```yaml
app-auth:
  name: ${REDDIT_APP_NAME}
  id: ${REDDIT_APP_CLIENT_ID}
  secret: ${REDDIT_APP_SECRET}
```

---

## Search
```yaml
- type: search
  search-engine: duckduckgo
  bangs:
    - title: YouTube
      shortcut: "!yt"
      url: https://www.youtube.com/results?search_query={QUERY}
```

| Name | Type | Default |
| ---- | ---- | ------- |
| search-engine | string | duckduckgo |
| new-tab | boolean | false |
| autofocus | boolean | false |
| target | string | _blank |
| placeholder | string | Type here to search… |
| bangs | array | |

- `search-engine`: a name (`duckduckgo`, `google`, `bing`, `perplexity`,
  `kagi`, `startpage`) or a custom URL with `{QUERY}`.
- `target`: `_blank`, `_self`, `_parent`, `_top`.
- Bang properties: `title` (optional), `shortcut` (required, quote values
  starting with `!`), `url` (required, `{QUERY}` placeholder).

Keyboard: `S` focus, `Enter` search same tab, `Ctrl+Enter` new tab, `Escape`
blur, `Up` insert last query. `new-tab: true` swaps Enter/Ctrl+Enter behavior.

---

## Group
See SKILL.md. Tabbed container; no nested group/split-column.

## Split Column
See SKILL.md. Splits a full column; supports `max-columns`; can nest a group.

---

## Extension
Display a 3rd-party widget.

```yaml
- type: extension
  url: https://domain.com/widget/display-a-message
  allow-potentially-dangerous-html: true
  parameters:
    message: Hello, world!
```

| Name | Type | Default |
| ---- | ---- | ------- |
| url | string (required) | |
| fallback-content-type | string | |
| allow-potentially-dangerous-html | boolean | false |
| headers | key/value | |
| parameters | key/value | |

- Query on `url` is stripped; use `parameters` instead.
- `fallback-content-type`: only `html` supported.
- `allow-potentially-dangerous-html`: only enable for extensions you trust.

---

## Weather
Data from open-meteo.com.

```yaml
- type: weather
  units: metric
  hour-format: 12h
  location: London, United Kingdom
```

| Name | Type | Default |
| ---- | ---- | ------- |
| location | string (required) | |
| units | string | metric |
| hour-format | string | 12h |
| hide-location | boolean | false |
| show-area-name | boolean | false |

- `location`: uses the first geocoding match. For ambiguous US cities add the
  state (e.g. `Greenville, North Carolina, United States`).
- `units`: `metric` or `imperial`. `hour-format`: `12h` or `24h`.
- `show-area-name`: include state/administrative area in display.
- Cache updates on the hour (not configurable).

---

## Todo
Browser-local to-do list.

```yaml
- type: to-do
  id: my-list
```

| Name | Type | Default |
| ---- | ---- | ------- |
| id | string | |

- `id`: separate lists need distinct IDs (stored in browser local storage);
  same ID shares tasks.

---

## Monitor
Reachability of sites via GET (200 = OK), with response time.

```yaml
- type: monitor
  cache: 1m
  title: Services
  sites:
    - title: Jellyfin
      url: https://jellyfin.yourdomain.com
      icon: /assets/jellyfin-logo.png
```

| Name | Type | Default |
| ---- | ---- | ------- |
| sites | array (required) | |
| style | string | |
| show-failing-only | boolean | false |

- `style`: `compact`.

Per-site properties:

| Name | Type | Default |
| ---- | ---- | ------- |
| title | string (required) | |
| url | string (required) | |
| check-url | string | (uses url) |
| error-url | string | (uses url) |
| icon | string | |
| timeout | string | 3s |
| allow-insecure | boolean | false |
| same-tab | boolean | false |
| alt-status-codes | array | |
| basic-auth | object | |

```yaml
alt-status-codes: [403]
basic-auth:
  username: your-username
  password: your-password
```

---

## Releases
Latest releases from GitHub, GitLab, Codeberg, Docker Hub.

```yaml
- type: releases
  show-source-icon: true
  repositories:
    - go-gitea/gitea
    - codeberg:redict/redict
    - gitlab:fdroid/fdroidclient
    - dockerhub:gotify/server
```

| Name | Type | Default |
| ---- | ---- | ------- |
| repositories | array (required) | |
| show-source-icon | boolean | false |
| token | string | |
| gitlab-token | string | |
| limit | integer | 10 |
| collapse-after | integer | 5 |

- Prefixes: `gitlab:`, `codeberg:`, `dockerhub:`. Docker Hub official images
  omit owner (`dockerhub:nginx`); tags allowed (`dockerhub:nginx:stable-alpine`).
- Prereleases (GitHub only):

```yaml
repositories:
  - repository: glanceapp/glance
    include-prereleases: true
```

- `token` / `gitlab-token`: use `${GITHUB_TOKEN}` etc. GitHub is 60 req/hr
  unauthenticated.
- `collapse-after`: `-1` never collapses.

---

## Docker Containers
> Requires mounting `/var/run/docker.sock`.

```yaml
- type: docker-containers
  hide-by-default: false
```

| Name | Type | Default |
| ---- | ---- | ------- |
| hide-by-default | boolean | false |
| format-container-names | boolean | false |
| sock-path | string | /var/run/docker.sock |
| category | string | |
| running-only | boolean | false |

Configure via container labels:

```yaml
labels:
  glance.name: Jellyfin
  glance.icon: si:jellyfin
  glance.url: https://jellyfin.domain.com
  glance.description: Movies & shows
  glance.category: media
  glance.id: immich       # mark a parent
  glance.parent: immich   # attach child to parent
  glance.hide: false
  glance.same-tab: false
```

Or inline via `containers` (key = container name, no `glance.` prefix):

```yaml
- type: docker-containers
  containers:
    container_name_1:
      name: Container Name
      description: Description
      url: https://container.domain.com
      icon: si:container-icon
      hide: false
```

- `hide-by-default: true` requires `glance.hide: false` per shown container.
- Parent/child: child status propagates to parent.
- `category`: filter by `glance.category` (multiple widgets per category).

---

## DNS Stats
AdGuard Home, Pi-hole, or Technitium.

```yaml
- type: dns-stats
  service: adguard
  url: https://adguard.domain.com/
  username: admin
  password: ${ADGUARD_PASSWORD}
```

| Name | Type | Required |
| ---- | ---- | -------- |
| service | string | no (default pihole) |
| allow-insecure | bool | no |
| url | string | yes |
| username | string | adguard |
| password | string | adguard or pihole-v6 |
| token | string | pihole (v5) / technitium |
| hide-graph | bool | no |
| hide-top-domains | bool | no |
| hour-format | string | no (12h) |

- `service`: `adguard`, `technitium`, `pihole` (v5 and below), `pihole-v6`.
- Pi-hole v6 password = admin/app password (Settings → Web Interface/API).
- Pi-hole v5 token = Settings → API → Show API token. Technitium token via
  Administration → Sessions → Create Token.

---

## Server Stats
CPU/memory/disk of local or remote servers. Remote requires the Glance Agent.

```yaml
- type: server-stats
  servers:
    - type: local
      name: Services
```

- `servers` optional; omit to show the local server. CPU >80°C shows a flame
  icon and turns indicators to the negative color.

Common per-server props: `type` (`local`/`remote`, required), `name`,
`hide-swap`.

Local server extra props: `cpu-temp-sensor`, `hide-mountpoints-by-default`,
`mountpoints` (map of path → `{ name, hide }`).

```yaml
- type: server-stats
  servers:
    - type: local
      hide-mountpoints-by-default: true
      mountpoints:
        "/": { name: Root, hide: false }
        "/mnt/data": { name: Data, hide: false }
        "/boot/efi": { hide: true }
```

Remote server props: `url` (required), `token`, `timeout` (default 3s).

---

## Repository
GitHub repo info + open PRs/issues/commits.

```yaml
- type: repository
  repository: glanceapp/glance
  pull-requests-limit: 5
  issues-limit: 3
  commits-limit: 3
```

| Name | Type | Default |
| ---- | ---- | ------- |
| repository | string (required) | |
| token | string | |
| pull-requests-limit | integer | 3 |
| issues-limit | integer | 3 |
| commits-limit | integer | -1 |

- Limits: `-1` to hide that section. `token`: `${GITHUB_TOKEN}` recommended.

---

## Bookmarks
Grouped link lists.

```yaml
- type: bookmarks
  groups:
    - links:
        - { title: Gmail, url: https://mail.google.com/mail/u/0/ }
    - title: Entertainment
      color: 10 70 50
      links:
        - { title: Netflix, url: https://www.netflix.com/ }
```

| Name | Type | Required |
| ---- | ---- | -------- |
| groups | array | yes |

Group props: `title`, `color` (HSL, default primary), `links` (required),
`same-tab`, `hide-arrow`, `target`.

Link props: `title` (required), `url` (required), `description`, `icon`,
`same-tab`, `hide-arrow`, `target`.

- `target`: `_blank`/`_self`/`_parent`/`_top` (takes precedence over
  `same-tab`). Group values apply to all links unless overridden per link.

---

## ChangeDetection.io
```yaml
- type: change-detection
  instance-url: https://changedetection.mydomain.com/
  token: ${CHANGE_DETECTION_TOKEN}
```

| Name | Type | Default |
| ---- | ---- | ------- |
| instance-url | string | https://www.changedetection.io |
| token | string | |
| limit | integer | 10 |
| collapse-after | integer | 5 |
| watches | array | |

- `token`: Settings → API. `watches`: list specific UUIDs (default all).
- `collapse-after`: `-1` never collapses.

---

## Clock
```yaml
- type: clock
  hour-format: 24h
  timezones:
    - { timezone: Europe/Paris, label: Paris }
    - { timezone: America/New_York, label: New York }
```

| Name | Type | Default |
| ---- | ---- | ------- |
| hour-format | string | 24h |
| timezones | array | |

- `hour-format`: `12h`/`24h`. Per-timezone: `timezone` (required, tz identifier
  like `Europe/London`), `label` (optional override).

---

## Calendar
```yaml
- type: calendar
  first-day-of-week: monday
```

| Name | Type | Default |
| ---- | ---- | ------- |
| first-day-of-week | string | monday |

- Any weekday name is valid. Cache updates on the hour.

## Calendar (legacy)
Deprecated. `- type: calendar-legacy` with `start-sunday` (boolean, default
false).

---

## Markets
Yahoo Finance quotes with a 21d chart.

```yaml
- type: markets
  markets:
    - { symbol: SPY, name: S&P 500 }
    - symbol: BTC-USD
      name: Bitcoin
      chart-link: https://www.tradingview.com/chart/?symbol=INDEX:BTCUSD
    - symbol: AAPL
      name: Apple
      symbol-link: https://www.google.com/search?tbm=nws&q=apple
```

| Name | Type | Required |
| ---- | ---- | -------- |
| markets | array | yes |
| sort-by | string | no |
| chart-link-template | string | no |
| symbol-link-template | string | no |

- `sort-by`: `change` (percent) or `absolute-change`.
- Templates use `{SYMBOL}`; per-market `chart-link`/`symbol-link` override them.
- Per-market: `symbol` (required), `name`, `symbol-link`, `chart-link`.

---

## Twitch Channels
```yaml
- type: twitch-channels
  channels: [jembawls, giantwaffle, asmongold]
```

| Name | Type | Default |
| ---- | ---- | ------- |
| channels | array (required) | |
| collapse-after | integer | 5 |
| sort-by | string | viewers |

- `sort-by`: `viewers` or `live`. `collapse-after`: `-1` never collapses.

## Twitch Top Games
```yaml
- type: twitch-top-games
  exclude: [just-chatting, music, art, asmr]
```

| Name | Type | Default |
| ---- | ---- | ------- |
| exclude | array | |
| limit | integer | 10 |
| collapse-after | integer | 5 |

- `exclude`: category slugs (from the category URL path).

---

## iframe
```yaml
- type: iframe
  source: <url>
  height: 400
```

| Name | Type | Default |
| ---- | ---- | ------- |
| source | string (required) | |
| height | integer | 300 |

- `height` minimum is 50.

---

## HTML
```yaml
- type: html
  source: |
    <p>Hello, <span class="color-primary">World</span>!</p>
```

- Use `|` for multi-line HTML.
