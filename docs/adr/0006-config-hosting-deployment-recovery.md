# ADR-0006: Configuration ownership, hosting, deployment and recovery

- Status: Accepted
- Date: 2026-08-21

## Context

The project must be reproducible locally and deployable to Beget while Varbase
11 is currently on an RC baseline. Configuration, dependencies, mail and backup
ownership must not depend on manual production changes.

## Decision

The WSL/DDEV checkout is the sole active development checkout; the Windows
checkout remains present but inactive. Composer lock owns dependencies and
versioned `config/sync` owns Drupal configuration. Secrets stay in environment
settings, never exported configuration.

Continue development on the tested Varbase 11 RC baseline. Before production,
review the available stable release and upgrade when compatible and appropriate.
Use Beget shared-first after qualification; move to VPS only on a confirmed
trigger. Production mail uses authenticated SMTP, domain From and visitor email
only as Reply-To; SPF, DKIM and DMARC are mandatory. Deploy reviewed tags from
`main`, run Composer from lock, database updates, config import, cache rebuild
and smoke tests. Back up DB, files and required secrets, and test restores.

## Alternatives considered

- Keep Windows and WSL as equal writable checkouts.
- Keep config in ignored public files or edit production manually.
- Wait for stable before any development.
- Choose VPS immediately.
- Use visitor email as From or unauthenticated local mail.

## Consequences

Builds and configuration become reviewable and portable. Shared hosting remains
conditional on PHP/MySQL/CLI/cron/path/resource qualification. RC risk is
explicit until the pre-production stable review. Recovery may require DB/files
restore, not merely a Git rollback.

## Deferred triggers

Move to VPS for Solr/Redis/workers, unsupported stack, recurring resource or
cron failures, or measured SLO failure. Add persistent staging or tighter
RPO/RTO only when release frequency or business impact justifies it.

