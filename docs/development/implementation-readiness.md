# E+E Master implementation readiness

- Status: **Not ready for implementation**
- Date: 2026-08-21
- Active checkout: `/home/dukar/projects/epe-master`
- Active branch: `codex/implementation-baseline`

## Owner decisions accepted

All nine owner decisions are recorded: Service Content Type; trusted editor
Publish; RC development with stable pre-production review; admin MFA; minimal
PII and provisional 180-day retention; authenticated SMTP and domain
authentication; qualified Beget shared-first; missing-translation switcher to
the selected-language homepage with manual freshness control; Asset Status as
Options enum. The 180-day retention remains subject to legal review.

## ADRs accepted

- ADR-0002 — content model, Canvas boundary and multilingual presentation.
- ADR-0003 — Search API Database backend and retrieval policy.
- ADR-0004 — editorial access, private feedback and public participation boundary.
- ADR-0006 — configuration ownership, hosting, deployment and recovery.

ADR-0005 was deliberately not created because AI is outside launch scope.

## Blockers resolved

- WSL/DDEV is the sole active development checkout. The Windows checkout at
  `C:\Users\Dukar\Documents\ChatGPT\Drupal` is retained but inactive.
- Authoritative configuration moved from ignored
  `web/sites/default/files/sync` to versioned `config/sync`.
- Versioned `settings.php` selects `config/sync`; DDEV runtime settings remain
  in the environment-specific include. No secret was exported.
- `tour.tour.honeypot` was traced to Honeypot optional onboarding configuration
  for the absent Tour module. It was removed as orphan configuration; Honeypot
  remains enabled and Tour was not installed merely for onboarding UI.
- The eight Canvas differences were serialization/order migration, not content
  loss. A platform-only `cex → cim → cex → cim` cycle converged to no import or
  status differences; no `config_ignore` or custom code was used.
- `paragonie/sodium_compat` was updated from 2.5.0 to 2.5.2. Composer reports no
  security advisories. A clean `--no-dev` install applied all Varbase patches,
  platform requirements passed, and Drupal booted with HTTP 200.
- `oomphinc/composer-installers-extender` remains an accepted upstream warning:
  `vardot/varbase-patches 11.0.38` requires it and offers no replacement.
- The existing DDEV database imports the versioned configuration cleanly and
  reports no config differences.

## Remaining pre-production blockers

- **Disposable installation is not reproducible yet.** Direct
  `site:install --existing-config` fails on ECA configuration actions referring
  to entities not yet imported, a runtime Easy Encryption key, and Password
  Policy field ordering. Installing the Varbase profile first succeeds, but a
  subsequent import fails UUID/content deletion and Canvas in-use safeguards.
  The original DDEV DB was restored from snapshot and verified after both tests.
- GitHub default `main` remains behind the latest linear research/architecture
  commits. Updating the protected/default branch requires explicit owner
  authorization; the active WSL branch is based on the latest architecture
  commit instead of the remote `main` tip.
- Before production: review available stable Varbase and upgrade if appropriate;
  qualify the concrete Beget account; configure/test SMTP and SPF/DKIM/DMARC;
  add MFA for Site Admin/Super Admin; complete permission, privacy/legal,
  backup/restore and production smoke acceptance.

## Development source-of-truth

Only `/home/dukar/projects/epe-master` may receive development changes. Its
origin is `https://github.com/g1357/epe-master.git`. The Windows clone is a
read-only inactive reference and must not be independently committed or pushed.
Short branches and reviewed integration into `main` remain the release model.

## Config lifecycle

`config/sync` is authoritative and committed. Normal change flow is: make a
reviewed configuration change in DDEV, `drush cex`, inspect the diff,
`drush cim`, `drush config:status`, then repeat `cex` to prove convergence.
Production imports reviewed config and never exports back into Git. Secrets and
runtime service credentials remain outside exported configuration.

## CI gate

`.github/workflows/baseline.yml` defines two minimal jobs: locked clean Composer
installation with validation, audit and platform requirements; and Drupal
installation against disposable MySQL followed by config import/status, boot
and cache smoke. No code-style/static-analysis stack is added while custom code
does not exist. The Drupal job is intentionally a blocking gate and is expected
to fail until the disposable-install blocker above is solved.

## Ready / Not ready for implementation

**NOT READY.** Versioned configuration, existing-site import, Composer security,
clean dependency build and local runtime are stable. Production content-model
implementation must wait until disposable installation/config import is
reproducible and the latest baseline is integrated into remote `main`.

