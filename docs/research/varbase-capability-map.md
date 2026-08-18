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
| Content types | Типы, поля, revisions, display modes, Canvas versus node | Не начато | Не определено | — |
| Media | Media types, library, bulk upload, focal point, responsive images, Drimage | Не начато | Не определено | — |
| Views | Listings, exposed filters, reusable displays, multilingual filtering | Не начато | Не определено | — |
| Search | Search API baseline, indexing, Canvas/node coverage, путь к Solr | Не начато | Не определено | — |
| Taxonomy | Иерархии, management UI, multilingual terms, access | Не начато | Не определено | — |
| Multilingual | RU/EN, content without translation, language negotiation, fallback | Не начато | Не определено | — |
| Workflow / moderation | Draft/review/published, scheduling, notifications, per-language moderation | Не начато | Не определено | — |
| Users / roles / permissions | Штатные роли, least privilege, delegation, audit | Не начато | Не определено | — |
| SEO | Metatag, schema, sitemap, redirects, hreflang, robots, Yoast | Не начато | Не определено | — |
| Forms | Webform recipes, spam protection, multilingual forms, mail delivery | Не начато | Не определено | — |
| Comments | Core comments, moderation, notifications, anti-spam, необходимость функции | Не начато | Не определено | — |
| Layout and theming | Vartheme BS5, SDC, UI Patterns, Storybook, upgrade-safe extension | Инвентаризация завершена, extension не исследован | Varbase штатно | [Исследование Canvas](page-building-canvas-design-system.md) |
| Reusable components | Canvas patterns, SDC components, governance and reuse | Первый практический проход завершён | Varbase штатно | [Исследование Canvas](page-building-canvas-design-system.md) |
| Recipes | Composition, idempotency, config ownership, uninstall/rollback | Не начато | Не определено | — |
| API / integrations | JSON:API, OpenAPI, OAuth, consumers, data exposure | Не начато | Не определено | — |
| Performance / cache | Drupal cache layers, BigPipe, images, cron, queues, baseline metrics | Не начато | Не определено | — |
| Security | SecKit, password policy, permissions, dependency/patch risk, secrets | Не начато | Не определено | — |
| Deployment | Composer build, config flow, Beget constraints, cron and releases | Не начато | Не определено | — |
| Backup / restore | Database/files/config scope, DDEV restore, Beget recovery targets | Не начато | Не определено | — |

## Исследовательский backlog

Приоритет 1 — Page building / Canvas: это центральное нововведение Varbase 11 и
оно влияет на content model, reusable components, multilingual, permissions и
theming.

Затем: Multilingual → Content types → Media → Workflow → Layout/theming → Search
→ SEO → Operations/security.

## Важное ограничение

Наличие пакета или включённого модуля не считается подтверждением возможности.
Статус меняется только после воспроизводимого эксперимента и записи evidence.
