# Varbase 11: initial capability assessment

**Status:** in progress  
**Verified:** 2026-08-18  
**Scope:** upstream capabilities and risks only; no E+E Master features

## Executive summary

Varbase is a Drupal distribution intended to reduce repeated site-building work
through preconfigured modules, recipes, themes, and editorial tooling. Varbase 11
targets new Drupal 11 projects and is the appropriate line to evaluate for E+E
Master. As of this review, the newest listed release is `11.0.0-rc1`; therefore,
production adoption must remain behind an explicit readiness decision.

## Confirmed baseline

| Area | Current upstream position | Project implication |
| --- | --- | --- |
| Project template | `drupal/varbase_project` 11.0.6 | Keep the upstream Composer and DDEV baseline. |
| Distribution | `vardot/varbase` 11.0.0-rc1 | Treat production readiness as an explicit gate. |
| Core | Drupal 11.4.5 | Use DDEV project type `drupal11`. |
| PHP | 8.4.22 in the verified container | Keep PHP 8.4 as the research baseline. |
| CLI | Composer 2.10.2 and Drush 13.7.6 | Use only container-provided project tooling. |
| Document root | `web` | Keep repository layout Composer-compatible. |
| Front end | Bootstrap 5-based responsive foundation | Compare with E+E design-system needs before adoption. |
| Site building | Recipes and preconfigured features | Test composability, reversibility, and configuration ownership. |
| Editorial | CKEditor, media tooling, dashboards, inline editing | Validate workflows with representative editor tasks. |
| Layout | Layout Builder / landing-page tooling; Canvas is advertised in 11.x | Prototype both paths and avoid assuming they are interchangeable. |
| SEO | Metatag, sitemap, redirect, analytics-oriented defaults | Audit privacy, regional compliance, and unnecessary modules. |
| AI | Full install enables Varbase AI integrations | Never use real data before security, cost, and data-flow review. |
| Security | Security-oriented recipes and SecKit guidance | Treat defaults as a starting point, not a completed threat model. |

## Candidate capabilities to investigate

1. Recipe-first installation and the Varbase Starter Recipe.
2. Drupal Canvas maturity and coexistence with Layout Builder.
3. Content modelling, moderation, revisions, and multilingual defaults.
4. Media library, responsive images, focal point, and Drimage behavior.
5. Search, SEO, redirects, sitemap, and structured metadata.
6. Admin dashboard and permissions for distinct editorial roles.
7. Varbase ECA for visual workflows and its configuration portability.
8. Theme extension strategy, UI Patterns, Storybook, and Bootstrap coupling.
9. Update mechanics, contributed patches, recipes, and configuration drift.
10. Performance, cacheability, security baseline, and operational footprint.

## Risks and open questions

- `11.0.0-rc1` is a release candidate, not a stable release.
- The distribution includes a broad feature surface; unused defaults can increase
  maintenance, security, and editorial complexity.
- The package graph and patch set are captured in `composer.lock` and
  `patches.lock.json`; their breadth and pre-release dependencies require audit.
- Canvas, recipes, and other fast-moving Drupal capabilities require hands-on
  compatibility tests.
- Upgrade and rollback behavior must be demonstrated before production approval.
- Licensing, privacy, telemetry, AI providers, and external-service dependencies
  need a separate review.
- The full recipe provisioned an anonymous amazee.ai trial account when no
  OpenAI key was supplied. This automatic external interaction requires an early
  privacy and secrets review.
- Several Russian contrib translations are incomplete or unavailable.

## Research acceptance gate

Varbase 11 can be selected only after:

- a reproducible DDEV/Composer installation is documented;
- the package and patch inventory is reviewed;
- critical editorial journeys are prototyped;
- upgrade, configuration export/import, and rollback are tested;
- security and performance baselines are recorded;
- a stable-release policy or explicit pre-release exception is approved.

## Primary sources

- [Varbase project page](https://www.drupal.org/project/varbase)
- [Varbase releases](https://www.drupal.org/project/varbase/releases)
- [Varbase requirements](https://docs.varbase.vardot.com/developers/installing-varbase/requirements)
- [Varbase 11 documentation](https://docs.varbase.vardot.com/11.0.x)
- [Varbase project template](https://www.drupal.org/project/varbase_project)
- [Varbase patches](https://github.com/Vardot/varbase-patches)
