---
name: glance-custom-api
description: >-
  Guide for the Glance custom-api widget, which renders data from any JSON API
  using a Go html/template. Covers widget properties (url, headers, method,
  body/body-type, parameters, subrequests, options, frameless, allow-insecure,
  skip-json-validation), gjson-based JSON accessors (.JSON.String/Int/Float/
  Bool/Array/Exists with dot/index paths), looping and context, .Options for
  configurable widgets, .Response access, JSON Lines (.JSONLines), in-template
  HTTP requests (newRequest/getResponse), time handling, and the full list of
  template helper functions. Load this whenever building, editing, or debugging
  a custom-api widget or its template.
---

# Glance custom-api widget

`custom-api` renders data from a JSON API using a Go `html/template`. It relies
on Go's [text/template](https://pkg.go.dev/text/template) for logic and
[tidwall/gjson](https://github.com/tidwall/gjson) for JSON selection. Requires
basic knowledge of HTML/CSS, Go templates, and Glance's CSS utility classes.

## Minimal example
```yaml
- type: custom-api
  title: Random Fact
  cache: 6h
  url: https://uselessfacts.jsph.pl/api/v2/facts/random
  template: |
    <p class="size-h4 color-paragraph">{{ .JSON.String "text" }}</p>
```

## Properties
| Name | Type | Required | Default |
| ---- | ---- | -------- | ------- |
| url | string | no | |
| headers | key/value | no | |
| method | string | no | GET |
| body-type | string | no | json |
| body | any | no | |
| frameless | boolean | no | false |
| allow-insecure | boolean | no | false |
| skip-json-validation | boolean | no | false |
| template | string | yes | |
| options | map | no | |
| parameters | key/value (string or array) | no | |
| subrequests | map of requests | no | |

- `url` — must be reachable from the Glance server. Can be omitted if the
  request is built entirely in the template with `newRequest`.
- `headers` — request headers, e.g. `x-api-key: ${IMMICH_API_KEY}`.
- `method` — `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`, `HEAD`.
- `body-type` — `json` or `string`.
- `body` — map (for `json`) or string (for `string`).
- `frameless` — remove border/padding.
- `allow-insecure` — ignore invalid/self-signed certs.
- `skip-json-validation` — required for JSON Lines/NDJSON responses.
- `parameters` — query params; overrides any query already in `url`. Values may
  be strings or arrays.
- `subrequests` — extra requests run concurrently, available via
  `.Subrequest "key"`. Support all main properties except nested `subrequests`.

```yaml
headers:
  x-api-key: ${API_KEY}
  Accept: application/json

body-type: json
body:
  key1: value1
  multiple-items: [item1, item2]

parameters:
  param1: value1
  param2: [item1, item2]
```

### Subrequests
```yaml
- type: custom-api
  cache: 2h
  url: https://uselessfacts.jsph.pl/api/v2/facts/random
  subrequests:
    another-one:
      url: https://uselessfacts.jsph.pl/api/v2/facts/random
  template: |
    <p>{{ .JSON.String "text" }}</p>
    <p>{{ (.Subrequest "another-one").JSON.String "text" }}</p>
    {{ $s := .Subrequest "another-one" }}
    <p>{{ $s.Response.StatusCode }}</p>
```

### Options — reusable configurable widgets
Read config values in the template with `.Options` so one template serves many
widgets:

```yaml
custom-widgets:
  - &example-widget
    type: custom-api
    template: |
      <ul class="list list-gap-10 collapsible-container"
          data-collapse-after="{{ .Options.IntOr "collapse-after" 5 }}">
        {{ if (.Options.BoolOr "show-thumbnails" true) }}
          <li><img src="{{ .JSON.String "thumbnail" }}" /></li>
        {{ end }}
      </ul>

pages:
  - name: Home
    columns:
      - size: full
        widgets:
          - <<: *example-widget
            options: { collapse-after: 3, show-thumbnails: false }
          - <<: *example-widget
            options: { collapse-after: 8, show-thumbnails: true }
```

`.Options` methods: `StringOr`, `IntOr`, `FloatOr`, `BoolOr`, and `JSON`.

## Template guide & function reference
The template language, gjson accessors, looping/context rules, time handling,
in-template HTTP requests, JSON Lines, and the complete list of helper
functions are documented in
[references/templates.md](references/templates.md). Consult it whenever writing
or debugging a template so accessor names, function signatures, and gotchas are
correct.

### Quick reminders
- Access fields via typed accessors: `.JSON.String "key"`, `.JSON.Int "key"`,
  `.JSON.Float`, `.JSON.Bool`, `.JSON.Array`, `.JSON.Exists`.
- Use dot/index paths: `.JSON.String "user.address.city"`,
  `.JSON.String "users.0.name"`.
- Inside `range .JSON.Array "posts"` the context is the element; use `$` to
  reach the top level (`$.JSON.String "author"`); for arrays of scalars use the
  empty key: `range .JSON.Array ""` then `.String ""`.
- Relative time must be an HTML attribute:
  `<span {{ .String "date" | parseTime "rfc3339" | toRelativeTime }}></span>`.
- Response info: `.Response.StatusCode`, `.Response.Header.Get "Content-Type"`.
