# custom-api templates — full reference

Templates use Go's `html/template` for logic and tidwall/gjson for JSON
selection. This file covers accessors, looping, time, in-template requests,
JSON Lines, and every helper function.

## Accessing fields
```json
{ "title": "My Title", "content": "My Content" }
```
```html
<div>{{ .JSON.String "title" }}</div>
<div>{{ .JSON.String "content" }}</div>
```

## Looping over an array of objects
```json
{ "author": "John Doe", "posts": [ { "title": "A" }, { "title": "B" } ] }
```
```html
{{ range .JSON.Array "posts" }}
  <div>{{ .String "title" }}</div>
{{ end }}
```
Inside `range`, the context becomes the current element, so `.JSON` is dropped.
Reach the top-level context with `$`:
```html
{{ range .JSON.Array "posts" }}
  <div>{{ .String "title" }}</div>
  <div>{{ $.JSON.String "author" }}</div>
{{ end }}
```

## Arrays of scalars
When the context is a non-object scalar, specify its type with an empty key:
```json
["Apple", "Banana", "Cherry"]
```
```html
{{ range .JSON.Array "" }}
  <div>{{ .String "" }}</div>
{{ end }}
```
Access by index directly: `{{ .JSON.String "0" }}` → `Apple`.

## Nested paths and indexes
Dot notation and indexes work anywhere in the path:
```html
<div>{{ .JSON.String "user.address.city" }}</div>
<div>{{ .JSON.String "users.0.name" }}</div>
```

## Existence check
```html
{{ if .JSON.Exists "user.age" }}
  <div>{{ .JSON.Int "user.age" }}</div>
{{ else }}
  <div>Age not provided</div>
{{ end }}
```

## Math
```html
<div>{{ sub (.JSON.Int "price") (.JSON.Int "discount") }}</div>
```
Operations: `add`, `sub`, `mul`, `div`, `mod`. If both operands are ints the
result is an int, otherwise a float. Divide by zero → `0`. Non-numeric → `NaN`.

## Time / relative time
```html
{{ range .JSON.Array "posts" }}
  <div>{{ .String "title" }}</div>
  <div {{ .String "date" | parseTime "rfc3339" | toRelativeTime }}></div>
{{ end }}
```
- `parseTime layout string` — layout can be `RFC3339`, `RFC3339Nano`,
  `DateTime`, `DateOnly`, `TimeOnly`, `unix`, or a custom Go layout.
- `toRelativeTime` output **must** be used as an HTML tag attribute; Glance
  populates and updates it client-side.

## Response info
```html
{{ if eq .Response.StatusCode 200 }}<p>Success!</p>{{ else }}<p>Failed</p>{{ end }}
<div>{{ .Response.Header.Get "Content-Type" }}</div>
```

## JSON Lines / NDJSON
Set `skip-json-validation: true`, then iterate with `.JSONLines`:
```html
{{ range .JSONLines }}
  <p>{{ .String "name" }} is {{ .Int "age" }} years old</p>
{{ end }}
```
gjson JSON-Lines selectors also work, e.g. all names:
```html
{{ range .JSON.Array "..#.name" }}<p>{{ .String "" }}</p>{{ end }}
```

## In-template HTTP requests (chained calls)
Use the result of one call in another:
```yaml
- type: custom-api
  url: https://api.example.com/get-id-of-something
  template: |
    {{ $theID := .JSON.String "id" }}
    {{
      $something := newRequest (concat "https://api.example.com/something/" $theID)
        | withParameter "key" "value"
        | withHeader "Authorization" "Bearer token"
        | getResponse
    }}
    {{ $something.JSON.String "title" }}
```
Omit `url` entirely and build the whole request in the template when parameters
are dynamic:
```yaml
- type: custom-api
  title: Events from the last 24h
  template: |
    {{
      $events := newRequest "https://api.example.com/events"
        | withParameter "after" (offsetNow "-24h" | formatTime "rfc3339")
        | getResponse
    }}
    {{ if eq $events.Response.StatusCode 200 }}
      {{ range $events.JSON.Array "events" }}
        <div>{{ .String "title" }}</div>
        <div {{ .String "date" | parseTime "rfc3339" | toRelativeTime }}></div>
      {{ end }}
    {{ else }}
      <p>Failed: {{ $events.Response.Status }}</p>
    {{ end }}
```
Always check the status code manually. Functions: `newRequest`, `withParameter`,
`withHeader`, `getResponse`.

---

## Functions

### On the `JSON` object
- `String(key) string`
- `Int(key) int`
- `Float(key) float`
- `Bool(key) bool`
- `Array(key) []JSON`
- `Exists(key) bool`

### On the `Options` object
- `StringOr(key, default) string`
- `IntOr(key, default) int`
- `FloatOr(key, default) float`
- `BoolOr(key, default) bool`
- `JSON(key) JSON` — stringified JSON object; errors if key missing

### Glance helpers
- `toFloat(i int) float`
- `toInt(f float) int`
- `toRelativeTime(t) template.HTMLAttr` — use as an HTML attribute
- `now() time.Time`
- `offsetNow(offset string) time.Time` — e.g. `3h`, `-1h`, `2h30m10s`
- `duration(str) time.Duration` — e.g. `1h`, `24h`, `5h30m`
- `parseTime(layout, s) time.Time` — layouts as above (or `unix`, `RFC3339`, …)
- `formatTime(layout, s)` — formats a time using same layouts
- `parseLocalTime(layout, s)` — uses local timezone when none present
- `parseRelativeTime(layout, s)` — shorthand for `parseTime | toRelativeTime`
- `add(a,b)`, `sub(a,b)`, `mul(a,b)`, `div(a,b)` (float), `mod(a,b)` (int)
- `formatApproxNumber(n int) string` — `1000` → `1k`
- `formatNumber(n) string` — `1000` → `1,000`
- `trimPrefix(prefix, str)`, `trimSuffix(suffix, str)`, `trimSpace(str)`
- `replaceAll(old, new, str)`
- `replaceMatches(pattern, replacement, str)` — regex
- `findMatch(pattern, str)`, `findSubmatch(pattern, str)` — regex
- `sortByString(key, order, arr)` — order `asc`/`desc`
- `sortByInt(key, order, arr)`
- `sortByFloat(key, order, arr)`
- `sortByTime(key, layout, order, arr)`
- `concat(strings ...string) string`
- `unique(key, arr) []JSON`
- `percentChange(current, previous) float`
- `startOfDay(t) time.Time`, `endOfDay(t) time.Time`

### Go text/template built-ins
- `eq`, `ne`, `lt`, `le`, `gt`, `ge`
- `and(...)`, `or(...)`, `not(a)`
- `index(a, b)` — value at index
- `len(a)` — length
- `printf(format, ...)`
