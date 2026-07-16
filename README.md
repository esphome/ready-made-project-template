# ESPHome Ready-Made Project Template

This is a [GitHub template repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository) for creating new ESPHome **ready-made project** repositories — repos that host YAML configurations and published firmware for multiple premade devices or kits (like [rf-proxies](https://github.com/esphome/rf-proxies), [bluetooth-proxies](https://github.com/esphome/bluetooth-proxies) or [media-players](https://github.com/esphome/media-players)), as opposed to single-firmware repos.

## Creating a new repository

1. Click **Use this template** -> **Create a new repository** and pick a name. The name drives everything: `rf-proxies` becomes the project title `Rf Proxies` and the firmware.esphome.io directory `rf-proxy` (plural repo name, singular firmware directory).
2. Ask an organization admin to add the new repository to the repository access list of the `ESPHOME_GITHUB_APP_PRIVATE_KEY` organization secret. The bootstrap workflow **fails with instructions** until this is done — re-run the failed jobs afterwards.
3. The [bootstrap workflow](.github/workflows/bootstrap.yml) runs on the initial push and opens a pull request that:
   - replaces the template placeholders (see below) with values derived from the repository name,
   - renames `README.template.md` to `README.md` (deleting this file),
   - deletes the bootstrap workflow itself,
   - creates the labels used by release-drafter, lock and stale.
4. Review and merge the bootstrap PR, then work through its **manual repository setup** checklist (branch/tag rulesets, `release` environment, squash-only merges, secret scanning, etc.). The rulesets ship as importable JSON in [.github/rulesets/](.github/rulesets/): **Settings -> Rules -> Rulesets -> New ruleset -> Import a ruleset**.
5. Replace `example-device/` with real device configurations. [build.yml](.github/workflows/build.yml) discovers every `*.factory.yaml` in the repository automatically — no workflow edits needed when adding or removing devices.

To override the derived firmware directory, set a `FIRMWARE_DIRECTORY` repository variable **before** the bootstrap workflow runs (or set it afterwards and fix the `update.source` URLs in the bootstrap PR — [publish.yml](.github/workflows/publish.yml) reads the variable at publish time).

The manual-setup checklist includes setting the repository homepage. The convention is `https://esphome.io/projects/?type=<type>`, where `<type>` is the project's filter value on the projects page: `voice`, `bluetooth`, `media`, `irrf` (shared by infrared and RF proxies) or `empty` (generic/esphome-web devices). This template repository itself uses the unfiltered `https://esphome.io/projects/`.

## Template placeholders

Placeholders only appear **outside** `.github/workflows/` — workflow files need no substitution, since `publish.yml` derives the firmware directory at runtime. This keeps the bootstrap commit free of workflow-file edits apart from deleting `bootstrap.yml` itself, which is why the ESPHome GitHub App token is required (the default `GITHUB_TOKEN` is refused pushes that modify `.github/workflows/`, and PRs it creates do not trigger CI).

| Placeholder              | Replaced with                                            |
| ------------------------ | -------------------------------------------------------- |
| `TEMPLATE-REPO-OWNER`    | The repository owner (organization or user)              |
| `TEMPLATE-REPO-NAME`     | The repository name, e.g. `rf-proxies`                   |
| `TEMPLATE-PROJECT-TITLE` | Title-cased repository name, e.g. `Rf Proxies`           |
| `TEMPLATE-FIRMWARE-DIR`  | Singularized repository name, e.g. `rf-proxy`            |

## Repository layout

One directory per device, containing two files:

| File                             | Purpose                                                             |
| -------------------------------- | ------------------------------------------------------------------- |
| `<device>/<device>.yaml`         | Core config users adopt via the ESPHome dashboard import            |
| `<device>/<device>.factory.yaml` | Factory wrapper (project metadata, improv, HTTP OTA) built by CI    |

## Release flow

1. Every push to `main` runs [release-drafter.yml](.github/workflows/release-drafter.yml), which maintains a CalVer (`YY.M.N`) draft release, and triggers [build.yml](.github/workflows/build.yml) to build all factory firmwares and attach them to the draft.
2. When ready to release, manually dispatch [release.yml](.github/workflows/release.yml). It patches the manifests with the final release notes and publishes the draft using the ESPHome GitHub App token (so downstream workflows trigger).
3. [publish.yml](.github/workflows/publish.yml) reacts to the published release: it uploads the firmware to R2, promotes it to the `beta` channel, and — for non-prereleases — to the `production` channel on firmware.esphome.io.

## Required secrets and variables

| Name                              | Kind         | Used by                                           |
| --------------------------------- | ------------ | ------------------------------------------------- |
| `ESPHOME_GITHUB_APP_CLIENT_ID`    | Org variable | `release.yml`, `bootstrap.yml`                    |
| `ESPHOME_GITHUB_APP_PRIVATE_KEY`  | Org secret   | `release.yml`, `bootstrap.yml`                    |
| `CLOUDFLARE_R2_ACCOUNT_ID`        | Org secret   | `publish.yml` via esphome/workflows               |
| `CLOUDFLARE_R2_ACCESS_KEY_ID`     | Org secret   | `publish.yml` via esphome/workflows               |
| `CLOUDFLARE_R2_SECRET_ACCESS_KEY` | Org secret   | `publish.yml` via esphome/workflows               |
| `CLOUDFLARE_R2_BUCKET`            | Org secret   | `publish.yml` via esphome/workflows               |

The `ESPHOME_GITHUB_APP_CLIENT_ID` org variable is available to every repository, but the `ESPHOME_GITHUB_APP_PRIVATE_KEY` org secret is only shared with an explicit repository list — the new repository must be added to it (used by `release.yml`, and by `bootstrap.yml` to delete itself and open a CI-triggering PR). `bootstrap.yml` fails with instructions until this is done; re-run the failed jobs afterwards. The `CLOUDFLARE_R2_*` secrets are consumed by the reusable [esphome/workflows](https://github.com/esphome/workflows) through `secrets: inherit` in `publish.yml` — verify they are shared with the repository too (the bootstrap PR checklist includes this).

## Maintenance

- All actions and reusable workflows are pinned to commit SHAs with version comments; [Dependabot](.github/dependabot.yml) keeps them updated weekly and groups `esphome/workflows` bumps.
- CI (yaml-lint and the PR build of `example-device`) runs on pull requests to this template, so template changes are validated before repos are generated from it. Release drafting is disabled in the template repository itself.
