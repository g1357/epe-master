# Карта возможностей Varbase 11

- Статус: структура исследования
- Baseline: Varbase 11.0.0-rc1 / Drupal 11.4.5
- Дата начала: 2026-08-18

## Цель

Проверить возможности чистой full-установки Varbase 11 до проектирования
E+E Master и определить минимальный production baseline без преждевременного
custom-кода.

## Классификация

Для каждого проверенного требования используется одна основная классификация:

- **Varbase штатно**;
- **Drupal core**;
- **Входит contrib**;
- **Требуется дополнительный contrib**;
- **Лучше скорректировать требование**;
- **Возможный custom**.

`Возможный custom` допустим только после фиксации воспроизводимого ограничения,
рассмотрения альтернатив и отдельного архитектурного решения.

## Метод исследования

Для каждой области фиксируются:

1. пользовательский сценарий и ожидаемый результат;
2. фактические модули, recipes и конфигурация;
3. пошаговый эксперимент на чистом baseline;
4. основная классификация;
5. ограничения, риски и maturity;
6. влияние на multilingual, permissions, cache и upgrades;
7. вывод и следующий эксперимент;
8. ссылки на первичные источники.

## Матрица

| Область | Вопросы первого исследования | Статус | Классификация | Evidence |
| --- | --- | --- | --- | --- |
| Page building / Canvas | Canvas pages, patterns, slots, редакторский UX, ограничения структуры | Первый практический проход завершён | Varbase штатно | [Исследование Canvas](page-building-canvas-design-system.md) |
| Content types | Типы, поля, revisions, display modes, Canvas versus node | Практический проход завершён | Drupal core + Varbase штатно | [Структурированный контент](02-structured-content.md) |
| Media | Media types, library, bulk upload, focal point, responsive images, Drimage | Media entity/Library/embed/focal point проверены; pipeline требует исследования | Drupal core + Varbase штатно + входит contrib | [Структурированный контент](02-structured-content.md) |
| Views | Listings, exposed filters, reusable displays, multilingual filtering | Asset catalog и reverse reference View проверены | Drupal core; входит contrib для улучшенного UX | [Структурированный контент](02-structured-content.md) |
| Search | Search API baseline, indexing, Canvas/node coverage, RU/EN relevance, filters и путь к Solr | Практический проход завершён: DB search работает, Canvas/node/draft boundary подтверждены; RU morphology, language filter, facets и tech tokens не готовы | Varbase штатно + входит contrib; дополнительный contrib для RU/Solr/facets | [Исследование поиска](04-search.md) |
| Taxonomy | Иерархии, management UI, multilingual terms, access | RU translation term подтверждён; shared identity/hierarchy рекомендованы | Drupal core | [Мультиязычность](03-multilingual.md) |
| Multilingual | RU/EN, content without translation, language negotiation, fallback | Практический проход завершён; single-language publication подтверждена, negotiation требует решения | Drupal core | [Мультиязычность](03-multilingual.md) |
| Workflow / moderation | Draft/review/published, scheduling, notifications, per-language moderation | Полный EN цикл Draft → Review → Published → новая Draft → Published → Archived подтверждён; published revision остаётся live; EN Published + RU Draft подтверждено | Drupal core + Varbase штатно + входит contrib | [Роли, workflow и access](05-users-roles-workflow-access.md) |
| Users / roles / permissions | Штатные роли, least privilege, delegation, audit | Практический проход завершён: штатные роли богаты, но author/publisher не разделены; Site Admin отделён от permissions/config | Varbase штатно; configuration для точного least privilege | [Роли, workflow и access](05-users-roles-workflow-access.md) |
| SEO | Metatag, schema, sitemap, redirects, hreflang, robots, Yoast | Практический проход завершён: metadata и 301 проверены; JSON-LD не настроен, sitemap пока только Blog/Page, найден конфликт canonical homepage | Varbase штатно + входит contrib; включение `schema_service` требует согласования | [SEO, формы и feedback](06-seo-forms-feedback.md) |
| Forms | Webform recipes, spam protection, multilingual forms, mail delivery | Contact и article feedback prototypes отправлены; submissions, Mailpit, translation, access, Canvas embedding и privacy boundary проверены | Varbase штатно + входит contrib | [SEO, формы и feedback](06-seo-forms-feedback.md) |
| Comments | Core comments, moderation, notifications, anti-spam, необходимость функции | Core Comment выключен; private Webform feedback признан естественным v1, public comments отложены до community use case | Лучше скорректировать требование; Drupal core + входит contrib | [SEO, формы и feedback](06-seo-forms-feedback.md) |
| Layout and theming | Vartheme BS5, SDC, UI Patterns, Storybook, upgrade-safe extension | Инвентаризация завершена, extension не исследован | Varbase штатно | [Исследование Canvas](page-building-canvas-design-system.md) |
| Reusable components | Canvas patterns, SDC components, governance and reuse | Первый практический проход завершён | Varbase штатно | [Исследование Canvas](page-building-canvas-design-system.md) |
| Recipes | Composition, idempotency, config ownership, uninstall/rollback | Практический проход завершён: 53 applied path entries, Varbase API/AI recipes разобраны; generic rollback и ongoing ownership отсутствуют, config management остаётся authoritative | Drupal core + Varbase штатно | [API, AI, Recipes и интеграции](07-api-ai-recipes-integrations.md) |
| API / integrations | JSON:API, OpenAPI, OAuth, consumers, data exposure | Практический проход завершён: closed-by-default и read-only подтверждены; published/forward Draft access проверен; OpenAPI отражает enabled GET routes, но auth/multilingual contract неполон | Drupal core + Varbase штатно + входит contrib | [API, AI, Recipes и интеграции](07-api-ai-recipes-integrations.md) |
| Performance / cache | Drupal cache layers, BigPipe, images, cron, queues, baseline metrics | Практический проход завершён: page/dynamic cache, aggregation, cron/queues и DDEV measurements подтверждены; Redis/CDN не нужны для baseline; image test не воспроизвёл 196 s | Drupal core + Varbase штатно + входит contrib; Hosting/configuration | [Production readiness и Beget](08-production-readiness-beget.md) |
| Security | SecKit, password policy, permissions, dependency/patch risk, secrets | Роли/login/access и form abuse проверены; production status/settings/audit исследованы; Composer advisory, отсутствующие production secrets/settings и MFA требуют решения до launch | Varbase штатно + входит contrib; Hosting/configuration; дополнительный contrib для MFA при решении | [Роли, workflow и access](05-users-roles-workflow-access.md), [SEO, формы и feedback](06-seo-forms-feedback.md), [Production readiness](08-production-readiness-beget.md) |
| Deployment | Composer build, config flow, Beget constraints, cron and releases | Практический проход завершён: shared-first квалификация, простой tag workflow и deploy order предложены; ignored config sync, failed import (orphaned Tour config), Canvas drift и две рассинхронизированные working copies — blockers | Drupal core + Composer + Hosting/configuration | [API, AI, Recipes и интеграции](07-api-ai-recipes-integrations.md), [Production readiness и Beget](08-production-readiness-beget.md) |
| Backup / restore | Database/files/config scope, DDEV restore, Beget recovery targets | DB export → marker → import → cache rebuild → HTTP 200 подтверждён; files/config archive и checksums созданы вне Git; рекомендованы RPO 24 h / RTO 4–8 h | Drupal core + DDEV + Hosting/configuration | [Production readiness и Beget](08-production-readiness-beget.md) |

## Исследовательский backlog

Приоритет 1 — Page building / Canvas: это центральное нововведение Varbase 11 и
оно влияет на content model, reusable components, multilingual, permissions и
theming.

Завершены восемь запланированных практических проходов исследовательской фазы.
Следующего исследования нет: переход к реализации, production architecture или
новым ADR возможен только по отдельному решению пользователя.

## Важное ограничение

Наличие пакета или включённого модуля не считается подтверждением возможности.
Статус меняется только после воспроизводимого эксперимента и записи evidence.
