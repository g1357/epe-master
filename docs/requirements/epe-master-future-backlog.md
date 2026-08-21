# E+E Master: future backlog после v1

- Статус: Deferred; не входит в scope реализации v1
- Дата: 2026-08-21

## Правило входа в roadmap

Пункт становится проектом только после появления owner, user story, данных,
acceptance criteria, privacy/security оценки и эксплуатационного бюджета.
Наличие модуля в full Varbase не является основанием включить функцию.

## Should have shortly after launch

| Пункт | Trigger | Первый штатный путь |
| --- | --- | --- |
| Search quality iteration | накоплен corpus и golden queries | Search API processors/View/structured fields |
| MFA | административный production access | зрелый Drupal 11 contrib после отдельного выбора |
| Translation freshness process | регулярные RU/EN обновления | workflow/report/training; ECA только при сложности |
| Webform retention/anti-spam tuning | реальный volume и legal policy | Webform limits/purge/configuration |
| First update staging clone | первый core/Varbase update | restore production backup во временную среду |
| Production performance review | появились реальные traffic/cache metrics | core caches и hosting tuning |

## Product backlog

### Registration и personal cabinet

Не включать ради «готовности». Trigger: утверждённые пользовательские действия
(saved content, subscriptions, обращения, закрытые материалы), account support,
verification, privacy и deletion lifecycle. Сначала core user/profile и
configuration; business role — только по реальному access model.

### Member/Internal access

Trigger: published content, который должен видеть определённый круг. До выбора
grants provider описать группы, владельцев, приглашение/отзыв доступа и
Search/Views/cache regression. Не использовать Draft, скрытый URL или taxonomy
как access control.

### Public comments/community

Trigger: компания готова поддерживать публичную дискуссию, moderation SLA,
notifications, spam/privacy policy и community ownership. До этого private
Article Feedback остаётся достаточным.

### Forum

Отдельный продуктовый scope. Trigger: устойчивое сообщество и команда
модерации. Сравнить Drupal solution с внешней forum platform; не считать forum
естественным продолжением Comments автоматически.

### News

Trigger: регулярная news/press лента с отличным lifecycle и metadata. До этого
использовать Technical Article или Canvas announcement.

### Video library

Trigger: утверждённый видеоплан, provider/privacy/caption policy. Начать с
Remote Video; self-hosting только при доказанной необходимости.

### Syntax highlighting и automatic TOC

Trigger: достаточный объём длинных технических публикаций и редакторская
приёмка semantic code/manual anchors как недостаточных. Сначала зрелый contrib,
не custom CKEditor plugin.

### Maps/geospatial Assets

Trigger: поиск по расстоянию, карта, структурированные адреса или интеграция.
Тогда оценить Address/Geofield/geocoder contrib и privacy/provider terms.

### Advanced analytics

Trigger: утверждённые KPI, consent/legal policy и владелец аналитики. Не
встраивать trackers «по умолчанию».

### E-commerce

Если появится, рассматривать отдельный сайт/subsite и ADR: payments, orders,
personal data, support и security радикально меняют risk profile. Не добавлять
commerce в corporate v1.

## Search/infrastructure backlog

### Solr

Trigger: DB backend не проходит обязательную RU morphology, protected technical
tokens, facets/autocomplete/spellcheck, relevance/p95 или load criteria. Нужны
VPS/managed service, configset, backup, update и monitoring.

### Redis/Memcached

Trigger: измеренный cache/DB bottleneck после core-cache tuning. Не является
обязательным признаком production Drupal.

### CDN/Cloudflare

Trigger: значимый географический latency/media traffic, DDoS/WAF requirement
или origin offload. Перед включением определить TLS, cache invalidation,
cookies и trusted proxy headers.

### Permanent staging

Trigger: регулярные releases, несколько разработчиков, API/AI integrations или
production-like acceptance team. До этого создавать temporary staging из
backup перед updates.

### VPS

Trigger: Solr/Redis, long-running workers/AI, process or memory limits, cron
failures, отсутствие MySQL 8/PHP 8.4/symlink/private path либо measured traffic.

## AI backlog

### Internal assistive AI

Human-triggered CKEditor, alt-text и taxonomy suggestions. До включения нужны
provider/data policy, cost limits, evaluation, privacy gate и audit. AI output
всегда проверяет редактор.

### External editorial agent

Не входит в launch. Trigger: стабильные production content model, config,
workflow и API contract, достаточный corpus и конкретный editorial ROI.

Планируемая boundary:

- Search API/View retrieval;
- explicit JSON:API resource allowlist;
- Simple OAuth Client Credentials;
- dedicated machine user и narrow role;
- только собственные или явно разрешённые Draft;
- Draft → In review; human-only Publish;
- revisions, revision logs, correlation ID и audit.

Запрещены Publish/Archive/Delete, users/roles/config/secrets, recipes/updates,
content model, Canvas governance, taxonomy creation, Webform/PII, redirects,
menus, global SEO и provider key management.

Unresolved для будущего ADR: edit-own versus bundle-wide edit-any ownership,
создание новых translations, API mutation boundary и OpenAPI contract.

### Semantic/vector search

Только после измеримого lexical baseline. Embeddings не должны маскировать
некачественный language/access/content model.

## Platform backlog

### `epe_master_base` recipe

Имеет смысл после стабилизации и versioning production config, когда нужно
воспроизводимо bootstrap-ить чистую среду или новый installation. Recipe может
составить languages, bundles, vocabularies, workflow, Views/Search, Webforms,
SEO и roles. Оно не владеет дальнейшими изменениями и не является rollback.

### ECA automation

Использовать выборочно для сложных cross-module events. Простые задачи сначала
решаются профильным механизмом: Webform handler, Scheduler, Content Moderation
Notifications, Search API cron. ECA model требует tests, recursion/retry/audit
review.

### Generated project theme

Не создавать только ради цветов. Trigger: branding нельзя выразить устойчиво
через Vartheme BS5/Varbase Design System/Canvas configuration и нужен
versioned design layer для logo, tokens, typography, component variants,
header/footer или spacing. Решение — отдельный architecture/design review.

## Security/operations backlog

- permanent external observability — только по SLA/compliance;
- RPO 1h/RTO 1h — только если leads/content loss оправдывают automation cost;
- immutable compliance audit — только при формальном требовании;
- advanced disaster recovery/standby — не требуется малому corporate site;
- API publication — только по конкретному consumer contract.
