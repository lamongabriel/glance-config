# glance-config

This repository builds a customizable [Glance](https://github.com/glanceapp/glance) dashboard configuration (`glance.yml`) that reads environment variables and supports modular includes.

## Agent skills

Reusable knowledge for authoring Glance configs lives under `.agents/skills/`. Each skill is a `SKILL.md` with YAML frontmatter (`name`, `description`) plus optional `references/`, `scripts/`, and `assets/` directories.

Kilo discovers these skills via the `skills.paths` entry in `.kilo/kilo.jsonc`, which points at `./.agents/skills`.

Available skills:

- **glance-config-basics** — config file structure, auto reload, environment variables & secrets, `$include`, icons, config schema, authentication, server, document, branding, theme, and pages/columns.
- **glance-widgets** — reference for every Glance widget (RSS, videos, hacker-news, reddit, monitor, docker-containers, server-stats, clock, calendar, markets, bookmarks, group, split-column, iframe, html, and more).
- **glance-custom-api** — the `custom-api` widget: Go templates, gjson selectors, options, parameters, subrequests, and template helper functions.
- **glance-community-widgets** — the community widgets collection (custom-api & extension widgets for Immich, Jellyfin, *arr, Proxmox, sports scores, and more); how to find and install them.

When working on `glance.yml` or any included config file, load the relevant skill(s) before making changes.

## Layout

```
repo/
  AGENTS.md
  .kilo/
    kilo.jsonc
  .agents/
    skills/
      glance-config-basics/
        SKILL.md
      glance-widgets/
        SKILL.md
      glance-custom-api/
        SKILL.md
      glance-community-widgets/
        SKILL.md
```
