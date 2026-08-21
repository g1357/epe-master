# E+E Master: backlog архитектурных решений

- Статус: proposed backlog; ADR не приняты этим review
- Дата: 2026-08-21

## Принцип

ADR нужен для решения, которое трудно отменить, затрагивает несколько
подсистем или задаёт долгосрочную boundary. Настройки отдельных полей, labels,
menu links, View displays и Webform handlers не требуют отдельных ADR.

## Уже принято

### ADR-0001 — Varbase 11 / platform first, custom last

Статус: Accepted. Сохраняется. Перед production требуется не новый platform
ADR, а review фактической стабильной версии Varbase.

## Рекомендуемые объединённые ADR

### ADR-0002 — Content model, Canvas boundary и multilingual presentation

Объединяет:

- Technical Article / Asset / conditional Service; no News;
- Canvas Page versus Content Type + Canvas Content Template;
- patterns/global regions/per-node override governance;
- RU default, `/` + `/en`, optional translations;
- translated text versus shared facts/references/Media;
- symmetric Canvas layout.

Почему один ADR: content identity, translation и presentation template должны
изменяться согласованно. Отдельные ADR создали бы противоречивые boundaries.

Owner decisions: Service Content Type, Asset Status representation,
language-switcher missing-translation UX.

### ADR-0003 — Search backend and retrieval policy

Фиксирует Search API abstraction, Database backend for v1, structured fields +
rendered content, current-language default и acceptance triggers for Solr.
Также отделяет human Search View, Asset Views filters и future agent retrieval.

Не включать конкретный golden query set или View field weights в ADR — это
testable configuration.

### ADR-0004 — Editorial access, feedback and public participation boundary

Объединяет:

- штатные roles и least privilege;
- Draft → Review → Published → Archived;
- author/publisher decision;
- registration/cabinet/member access deferred;
- private Webform feedback versus disabled public Comments;
- submissions/PII excluded from Search/API/AI.

Почему один ADR: identity, publication authority и participation surface — одна
security/product boundary. Forum/community позже потребует новый ADR только
при реальном проекте.

Owner decisions: four-eyes split, MFA timing, privacy retention/IP policy.

### ADR-0005 — AI/API boundary and machine identity

Фиксирует двухуровневую модель:

- internal human-triggered assistive AI;
- future external draft-only agent;
- Search API/View retrieval + explicit JSON:API allowlist;
- OAuth Client Credentials, dedicated machine user;
- human-only Publish, revisions/audit;
- forbidden capabilities and PII boundary.

Создавать только перед AI implementation, не сейчас. Owner decisions:
ownership `edit own` versus constrained `edit any`, provider/data policy и
business ROI.

### ADR-0006 — Configuration ownership, hosting, deployment and recovery

Объединяет:

- Recipes = bootstrap/composition;
- exported config = authoritative lifecycle state;
- `epe_master_base` timing;
- Beget shared-first qualification and VPS triggers;
- Composer from lock, Git branches/PR/tags, no develop;
- Local / temporary Staging / Production;
- deploy/update/rollback order;
- backup scope, RPO 24h/RTO 4–8h.

Почему объединить hosting/deployment/backup: выбор shared/VPS определяет
release mechanics и recovery capability; раздельные ADR будут неполными.

Owner decisions: shared qualification acceptance, RC/stable release risk,
mail transport, RPO/RTO acceptance.

## Что не требует ADR сейчас

- конкретные Topics/Technologies terms;
- Pathauto patterns и aliases;
- Schema field mappings;
- Webform labels/handlers;
- cron interval внутри диапазона 5–15 минут;
- image style dimensions;
- Content editor permission diff после утверждения owner model;
- minimal CI implementation details;
- temporary fixes Tour orphan/Canvas drift/Composer advisory — это blockers,
  а не альтернативные архитектуры.

## Порядок

1. Owner approvals из architecture review.
2. ADR-0002 и ADR-0004 до production content model/permissions.
3. ADR-0006 до production configuration/deployment.
4. ADR-0003 до финальной search configuration; можно принять вместе с v1.
5. ADR-0005 только когда AI входит в roadmap.

Не создавать ADR автоматически из этого backlog. Каждый требует отдельного
review и explicit acceptance владельца.
