# E+E Master: backlog архитектурных решений

- Статус: active backlog; ADR-0002/0003/0004/0006 приняты
- Дата: 2026-08-21

## Принцип

ADR нужен для решения, которое трудно отменить, затрагивает несколько
подсистем или задаёт долгосрочную boundary. Настройки отдельных полей, labels,
menu links, View displays и Webform handlers не требуют отдельных ADR.

## Уже принято

### ADR-0001 — Varbase 11 / platform first, custom last

Статус: Accepted. Сохраняется. Перед production требуется не новый platform
ADR, а review фактической стабильной версии Varbase.

## Принятые объединённые ADR

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

Статус: Accepted; см. [ADR-0002](../adr/0002-content-model-canvas-multilingual.md).

### ADR-0003 — Search backend and retrieval policy

Фиксирует Search API abstraction, Database backend for v1, structured fields +
rendered content, current-language default и acceptance triggers for Solr.
Также отделяет human Search View, Asset Views filters и future agent retrieval.

Не включать конкретный golden query set или View field weights в ADR — это
testable configuration.

Статус: Accepted; см. [ADR-0003](../adr/0003-search-backend-retrieval.md).

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

Статус: Accepted; см. [ADR-0004](../adr/0004-editorial-access-feedback.md).

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

Статус: Accepted; см. [ADR-0006](../adr/0006-config-hosting-deployment-recovery.md).

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

ADR-0002, ADR-0003, ADR-0004 и ADR-0006 приняты. ADR-0005 остаётся deferred и
создаётся только когда AI входит в roadmap после отдельного review.
