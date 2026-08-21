# ADR-0004: Editorial access, feedback and public participation boundary

- Status: Accepted
- Date: 2026-08-21

## Context

The small launch team needs moderated revisions and private visitor feedback,
without registration, community moderation or unnecessary personal data.

## Decision

Use Draft → In review → Published → Archived. No mandatory four-eyes rule is
imposed at launch; a trusted Content editor may Publish. Administrative duties
remain least-privilege, with mandatory MFA for Site Admin and Super Admin before
production. Registration, member access, comments and public API are disabled.

Contact uses optional name/email/organization plus required message/consent.
Article Feedback email is optional. Submissions are private and excluded from
Search/API/AI. IP is not retained unless anti-abuse demonstrates a need. The
provisional retention baseline is 180 days pending legal review.

## Alternatives considered

- Mandatory author/publisher separation at launch.
- Public Comments or a forum.
- Required visitor identity fields and indefinite retention.
- Treat Authenticated as a business access tier.

## Consequences

Editorial UX remains proportionate to team size while revisions and review are
available. Publish authority and MFA require a permission/security acceptance
test. The 180-day period cannot become final policy without legal approval.

## Deferred triggers

Revisit four-eyes when staffing, compliance or risk changes. Add CAPTCHA,
accounts, member access or community features only for measured abuse or an
approved product scenario.

