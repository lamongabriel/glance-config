---
name: glance-config-basics
description: >-
  Fundamentals for authoring a Glance dashboard config file (glance.yml):
  config file structure and auto reload, inserting environment variables and
  secrets (${ENV}, ${secret:}, ${readFileFromEnv:}), including other config
  files with $include, specifying icons, the config schema, authentication
  (users, hashed passwords, brute-force protection), the server block
  (host/port/proxied/base-url/assets-path), document head injection, branding,
  theme (HSL colors, presets, custom CSS), and organizing content with pages
  and columns. Load this whenever creating or editing glance.yml or any included
  config file that is not purely a single widget definition.
---

# Glance config basics

Glance is configured through a single YAML file, conventionally `glance.yml`.
This skill covers everything structural in that file. For individual widget
options load `glance-widgets`; for the `custom-api` widget load
`glance-custom-api`.

## The config file

### Auto reload
Config changes take effect on save without restarting the container/service.
Caveats:

- Changing **environment variables** does NOT trigger reload; restart manually.
- Deleting a config file stops it from being watched even if recreated.
- Starting with an invalid config makes Glance exit with an error.
- If a running instance is edited into an invalid state, the error is logged
  and Glance keeps the old config until the file is valid again.
- Reloading clears cached data, so data is re-fetched — reloading too often can
  trigger API rate limits.

### Environment variables
Insert env vars anywhere with `${ENV_VAR}` syntax. A missing variable is an
error (Glance won't start / won't load new config).

```yaml
server:
  host: ${HOST}
  port: ${PORT}
```

Works mid-string and for non-string values:

```yaml
- type: rss
  title: ${RSS_TITLE}
  feeds:
    - url: http://domain.com/rss/${RSS_CATEGORY}.xml
  limit: ${RSS_LIMIT}
```

Escape a literal `${NAME}` with a leading backslash:

```yaml
something: \${NOT_AN_ENV_VAR}
```

#### Secrets: other ways to provide tokens/passwords

Docker secrets — replaced with contents of `/run/secrets/<name>`:

```yaml
token: ${secret:github_token}
```

Load contents of a file whose path is in an env var (leading/trailing
whitespace is stripped):

```yaml
token: ${readFileFromEnv:TOKEN_FILE}
```

```yaml
# docker-compose.yml
services:
  glance:
    image: glanceapp/glance
    environment:
      - TOKEN_FILE=/home/user/token
    volumes:
      - /home/user/token:/home/user/token
```

### Including other config files
Use the `$include` directive with a relative (to the main config) or absolute
path. Included files support env vars and their changes trigger auto reload.

```yaml
pages:
  - $include: home.yml
  - $include: videos.yml
  - $include: homelab.yml
```

Rules:
- The included file's values must be at the **top level** (no extra
  indentation); correct indentation is added automatically based on where it is
  included.
- `$include` can appear anywhere, but must be on its own line with correct
  indentation.
- Because includes are resolved **before** YAML parsing, reported error line
  numbers are often wrong. Debug with:

```sh
glance --config /path/to/glance.yml config:print | less -N
# Docker:
docker run --rm -v ./glance.yml:/app/config/glance.yml glanceapp/glance config:print | less -N
```

Example — an included widget fragment (`rss.yml`, values at top level):

```yaml
- type: rss
  title: News
  feeds:
    - url: ${RSS_URL}
```

## Icons
Widgets that accept an `icon` property take either a URL to an image or a
prefixed icon name:

```yaml
icon: si:immich   # Simple icons        https://simpleicons.org/
icon: sh:immich   # selfh.st icons      https://selfh.st/icons/
icon: di:immich   # Dashboard icons     https://github.com/homarr-labs/dashboard-icons
icon: mdi:camera  # Material Design      https://pictogrammers.com/library/mdi/
```

Icons are loaded from `cdn.jsdelivr.net`; you can download and self-host them
instead. Simple icons and Material Design icons auto-invert for light/dark
themes. Force inversion for other icons (expects a black icon) with the
`auto-invert` prefix:

```yaml
icon: auto-invert https://example.com/path/to/icon.png
icon: auto-invert sh:glance-dark
```

## Config schema
For IDE autocompletion/validation, use the community schema:
<https://github.com/not-first/glance-schema>.

## Top-level structure
A `glance.yml` is built from these top-level properties:

```yaml
auth: ...        # optional, authentication
server: ...      # optional, server settings
document: ...    # optional, custom <head> HTML
branding: ...    # optional, logos/footer/app name
theme: ...       # optional, colors & presets
pages: ...       # required, the dashboard content
```

## Authentication
Set via top-level `auth`. A `secret-key` is required.

```yaml
auth:
  secret-key: # generate with: glance secret:make
  users:
    admin:
      password: 123456
    svilen:
      password: 123456
```

Generate a secret key:

```sh
./glance secret:make
docker run --rm glanceapp/glance secret:make
```

### Hashed passwords
Avoid plaintext by hashing and using `password-hash`:

```sh
./glance password:hash mysecretpassword
docker run --rm glanceapp/glance password:hash mysecretpassword
```

```yaml
auth:
  secret-key: ...
  users:
    admin:
      password-hash: $2a$10$o6SXqiccI3DDP2dN4ADumuOeIHET6Q4bUMYZD6rT2Aqt6XQ3DyO.6
```

### Brute-force protection
Glance blocks an IP after 5 failed auth attempts in 5 minutes. Behind a reverse
proxy (nginx, Traefik, NPM, ...) it needs the real client IP, so set
`server.proxied: true` so it trusts `X-Forwarded-For`.

## Server
Top-level `server` block.

```yaml
server:
  host: localhost      # default: all interfaces
  port: 8080           # 1..65535, default 8080
  proxied: true        # use X-Forwarded-* headers, default false
  base-url: /glance    # when hosted under a subpath behind a proxy
  assets-path: /home/user/glance-assets
```

| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| host | string | no | (all interfaces) |
| port | number | no | 8080 |
| proxied | boolean | no | false |
| base-url | string | no | |
| assets-path | string | no | |

Notes:
- `base-url` needs a leading `/` unless a full domain+path is given. Strip the
  prefix before forwarding to Glance (Caddy `handle_path` / `uri strip_prefix`).
- `assets-path` is served under `/assets/`. In Docker, mount the host path to
  the same path in the container and point `assets-path` there:

```yaml
# host: /home/user/glance-assets  ->  container: /app/assets
# mount: /home/user/glance-assets:/app/assets
assets-path: /app/assets
# then reference:
icon: /assets/gitea-icon.png
```

## Document
Insert custom HTML into `<head>` for all pages:

```yaml
document:
  head: |
    <script src="/assets/custom.js"></script>
```

## Branding
Top-level `branding` block.

```yaml
branding:
  hide-footer: false
  custom-footer: |
    <p>Powered by <a href="https://github.com/glanceapp/glance">Glance</a></p>
  logo-text: "G"
  logo-url: /assets/logo.png
  favicon-url: /assets/logo.png
  app-name: "My Dashboard"
  app-icon-url: "/assets/app-icon.png"
  app-background-color: "#151519"
```

| Name | Type | Default | Notes |
| ---- | ---- | ------- | ----- |
| hide-footer | bool | false | |
| custom-footer | string | | HTML for the footer |
| logo-text | string | G | Text in nav |
| logo-url | string | | Image in nav; overrides logo-text |
| favicon-url | string | | |
| app-name | string | Glance | Browser tab / PWA name |
| app-icon-url | string | Glance default | 512x512 PNG for PWA/tab |
| app-background-color | string | Glance default | Valid CSS color for PWA |

## Theme
Top-level `theme`. Colors are **HSL** as three space-separated numbers
(no `%`), e.g. `43 50 70`. Use <https://hslpicker.com/> to convert.

```yaml
theme:
  light: false
  background-color: 100 20 10
  primary-color: 40 90 40
  positive-color: 61 66 44   # defaults to primary-color
  negative-color: 6 96 59
  contrast-multiplier: 1.1
  text-saturation-multiplier: 1
  custom-css-file: /assets/my-style.css
  disable-picker: false
  presets:
    gruvbox-dark:
      background-color: 0 0 16
      primary-color: 43 59 81
      positive-color: 61 66 44
      negative-color: 6 96 59
    zebra:
      light: true
      background-color: 0 0 95
      primary-color: 0 0 10
      negative-color: 0 90 50
```

| Name | Type | Default |
| ---- | ---- | ------- |
| light | boolean | false |
| background-color | HSL | 240 8 9 |
| primary-color | HSL | 43 50 70 |
| positive-color | HSL | same as primary-color |
| negative-color | HSL | 0 70 70 |
| contrast-multiplier | number | 1 |
| text-saturation-multiplier | number | 1 |
| custom-css-file | string | |
| disable-picker | bool | false |
| presets | object | |

Property notes:
- `light` — inverts text colors for light backgrounds; does not change
  `background-color`.
- `primary-color` — used across the page, largely for unvisited links.
- `positive-color` / `negative-color` — up/down, online/offline, live, etc.
- `contrast-multiplier` — `1.3` makes text 30% lighter/darker; raise it if text
  is hard to read.
- `text-saturation-multiplier` — `0.5` halves saturation, `1.5` raises it 50%.
- `custom-css-file` — external path or one under `assets-path`. Each widget has
  a `widget-type-{name}` class (e.g. `.widget-type-rss a { font-size: 1.5rem; }`),
  and every widget supports a `css-class` property for per-instance styling.
- `disable-picker` — hides the theme picker; forces everyone to the default.
- `presets` — extra selectable presets with the same properties (except
  `custom-css-file`). Use keys `default-dark` / `default-light` to override the
  built-in defaults.

Pre-made themes to copy: <https://github.com/glanceapp/glance/blob/main/docs/themes.md>.

## Pages & columns
Content is organized into `pages`, each with up to **3 columns**, each column
holding any number of widgets. The first page is the home page; pages appear in
the nav bar in definition order.

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets: ...
      - size: full
        widgets: ...
      - size: small
        widgets: ...
```

### Page properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| name | string | yes | |
| slug | string | no | (from name) |
| width | string | no | default |
| desktop-navigation-width | string | no | |
| center-vertically | boolean | no | false |
| hide-desktop-navigation | boolean | no | false |
| show-mobile-header | boolean | no | false |
| head-widgets | array | no | |
| columns | array | yes | |

- `slug` — URL-friendly name; auto-generated from `name` if omitted.
- `width` — `default` (1600px), `slim` (1100px), or `wide` (1920px). `slim`
  allows a maximum of 2 columns.
- `desktop-navigation-width` — same values; keeps nav width steady across pages
  of differing widths.
- `center-vertically` — vertically centers content if it fits the viewport.
- `hide-desktop-navigation` — hide the top nav links on desktop.
- `show-mobile-header` — show a tall page-name header on mobile.
- `head-widgets` — widgets shown above the columns spanning their combined
  width. Best suited to `markets`, `rss` with `horizontal-cards`, and `videos`.

Head widgets example:

```yaml
pages:
  - name: Home
    head-widgets:
      - type: markets
        hide-header: true
        markets:
          - symbol: SPY
            name: S&P 500
          - symbol: BTC-USD
            name: Bitcoin
    columns:
      - size: small
        widgets:
          - type: calendar
      - size: full
        widgets:
          - type: hacker-news
      - size: small
        widgets:
          - type: weather
            location: London, United Kingdom
```

### Columns
Two sizes: `small` (fixed 300px) and `full` (fills remaining width). Up to 3
columns per page, and you must have **1 or 2 full columns**.

| Name | Type | Required |
| ---- | ---- | -------- |
| size | string | yes |
| widgets | array | no |

Common layouts:

```yaml
# small | full | small
columns:
  - { size: small, widgets: [...] }
  - { size: full,  widgets: [...] }
  - { size: small, widgets: [...] }

# full | small
columns:
  - { size: full,  widgets: [...] }
  - { size: small, widgets: [...] }

# full | full
columns:
  - { size: full, widgets: [...] }
  - { size: full, widgets: [...] }
```

For splitting a full column into more sub-columns (masonry, 3/4/5 column
layouts) use the `split-column` widget — see `glance-widgets`.

## Shared widget properties
Every widget supports these (details in `glance-widgets`):

| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| type | string | yes | |
| title | string | no | (widget default) |
| title-url | string | no | (widget default) |
| hide-header | boolean | no | false |
| cache | string | no | |
| css-class | string | no | |

- `cache` — a number followed by `s`/`m`/`h`/`d` (e.g. `30s`, `5m`, `2h`, `1d`).
  Not all widgets honor it; calendar and weather update on the hour and cannot
  be changed.
- `hide-header` — cannot hide the header of a `group` widget; also hides the
  red error dot shown when a widget fails to update.
