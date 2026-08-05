# EduDemo — Drupal Educational Platform (Pantheon Custom Upstream)

**EduDemo** is a simple, fictional mock demo website for an online educational
platform, built with **Drupal 11** and packaged as a **Pantheon Custom
Upstream**. It exists to demonstrate what a course-catalog / education marketing
site looks like when spun up from a reusable upstream on Pantheon.

> ⚠️ This is a demonstration site. All content is placeholder text — no real
> courses are offered and no real people are on the other end of the contact
> form.

## What this is

This repository is a **Pantheon Custom Upstream**: a shared starting point that
one or more Pantheon sites can be created from. It is based on Pantheon's
official [`drupal-composer-managed`](https://github.com/pantheon-upstreams/drupal-composer-managed)
upstream, with the Drupal core constraint moved up to Drupal 11 and a small
amount of demo configuration and content layered on top.

To learn how custom upstreams work and how to register one, see Pantheon's guide:
**https://docs.pantheon.io/guides/custom-upstream/create-custom-upstream**

## What the demo includes

- **Landing page** (set as the site front page) with a hero, value proposition,
  four feature highlights, calls-to-action, and a footer.
- **Courses** page with several placeholder course entries.
- **About** page describing the (fictional) platform and its mission.
- **Contact** page using Drupal core's built-in site-wide contact form.
- A **main navigation** menu linking Home / Courses / About / Contact.
- The **Olivero** front-end theme (Drupal core default).

## How it is structured

This upstream keeps the Pantheon `drupal-composer-managed` layout intact:

| Path | Purpose |
| --- | --- |
| `composer.json` | Composer project definition (Drupal 11, Drush, Pantheon integrations). |
| `pantheon.upstream.yml` | Pantheon platform settings (PHP 8.3, MariaDB 10.6, build step). |
| `config/` | Exported Drupal **configuration** (`drush config:export` output). On Pantheon this is the config sync directory. |
| `web/` | Drupal web root (core and contrib are Composer-managed and git-ignored). |
| `web/modules/custom/edudemo_demo/` | Small custom module that ships the demo **content** (pages, aliases) and the main-menu links. |

### Configuration vs. content

Drupal's `config:export` captures **configuration** (site settings, content
types, fields, the contact form, menus, blocks, theme, permissions) as YAML in
`config/`. It does **not** capture **content** (nodes), because nodes are not
configuration. So the demo pages are shipped by the `edudemo_demo` module, which
creates them in its install hook. A new site gets the pages when that module is
installed — most directly by installing the site **from the existing
configuration** (`edudemo_demo` is enabled in `config/core.extension.yml`).

## Deploying a site from this upstream

1. Register this repository as a custom upstream (see the manual step below).
2. Create a new site on Pantheon using the **EduDemo** upstream.
3. Install Drupal and apply the exported configuration and demo content. Run
   these against the target environment (`dev` shown here) with terminus:

   ```bash
   # Install core, then import the EduDemo configuration.
   terminus drush <site>.dev -- site:install standard -y
   terminus drush <site>.dev -- config:set system.site uuid \
     $(grep '^uuid:' config/system.site.yml | awk '{print $2}') -y
   terminus drush <site>.dev -- config:import -y

   # Populate the demo page bodies (see "Configuration vs. content" below),
   # then rebuild caches.
   terminus drush <site>.dev -- php:eval 'edudemo_demo_create_pages();'
   terminus drush <site>.dev -- cr
   ```

   Notes:
   - The `config:set ... uuid` step aligns the new site's UUID with the exported
     config so `config:import` will run (Drupal refuses to import config from a
     different site UUID).
   - `edudemo_demo` is enabled by the config import. Because it can be enabled
     before the page `body` field is attached, the pages may first be created
     empty; `edudemo_demo_create_pages()` fills them in and is safe to re-run.

## Keeping core up to date

The official Pantheon Drupal upstream is tracked as a git remote so future core
updates can be merged in:

```bash
git remote -v
# pantheon-drupal-composer-managed  https://github.com/pantheon-upstreams/drupal-composer-managed.git
```

When you want to pull in upstream improvements:

```bash
git fetch pantheon-drupal-composer-managed
git merge pantheon-drupal-composer-managed/main
```

## Required manual follow-up (not automatable)

Connecting this repository as a custom upstream must be done in the Pantheon
Dashboard and cannot be scripted:

> **Pantheon Dashboard → Organization → Upstreams → Add Upstream**, pointed at
> `https://github.com/aaronfox1/edu-demo-site`.

## Local development

There is intentionally **no persistent local environment** for this repo. The
demo configuration and content were generated once in a throwaway Drupal
instance and captured into `config/` and the `edudemo_demo` module; that
environment was then discarded. Standard Pantheon/Drupal workflows (Multidev,
DDEV, Lando, etc.) can be used if local work is ever needed.
