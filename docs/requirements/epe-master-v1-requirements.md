# E+E Master v1: требования к публичному запуску

- Статус: Architecture review complete; owner approvals pending
- Дата: 2026-08-21
- Реализация: не начата
- Источник истины: восемь документов `docs/research/` и capability map

## 1. Цель v1

Запустить двуязычный корпоративный и экспертный сайт E+E Master, который:

- ясно представляет компанию и два направления — управление активами и
  консалтинг;
- публикует экспертные технические и управленческие статьи;
- показывает текущий объект недвижимости и допускает добавление объектов;
- позволяет найти опубликованный материал и отправить приватное обращение;
- индексируется поисковыми системами;
- управляется небольшой доверенной редакцией через штатный workflow Varbase;
- остаётся воспроизводимым через Composer, Git, config management и backup.

v1 не является порталом, community, личным кабинетом или AI-продуктом.

## 2. Статусы review

- **Keep as-is** — требование естественно покрывается baseline;
- **Simplify** — v1 получает более узкий вариант;
- **Adjust to Varbase** — формулировка приведена к естественной модели;
- **Defer** — осознанно перенесено после запуска;
- **Drop** — исключено из roadmap v1;
- **Requires later decision** — нельзя утвердить без владельца/данных.

## 3. Консолидированный review требований

| Требование | Статус | Решение и основание |
| --- | --- | --- |
| Корпоративный сайт E+E Master | Keep as-is | Canvas Pages и structured content покрывают сценарий без custom-кода |
| Управление активами | Keep as-is | Раздел Canvas + Asset entities/View; подтверждено исследованием №2 |
| Консалтинг верхнего уровня | Keep as-is | Раздел Canvas, раскрывающий предложения и expertise |
| IT / industrial software / обучение | Simplify | Одна тематическая ветвь консалтинга, не отдельная подсистема |
| Project management для SMB | Simplify | Вложенная страница/секция и при необходимости Service record |
| Business consulting | Simplify | Объединить в ветвь Business Consulting / Interim Management |
| Interim management | Adjust to Varbase | Не создавать отдельный тип; представить контентом/услугой |
| Текущий объект недвижимости | Keep as-is | Один Asset допустим; модель сохраняет рост каталога |
| Новые объекты | Keep as-is | Asset + Views масштабируются без новой архитектуры |
| Экспертные статьи | Keep as-is | Technical Article — главный повторяемый тип |
| Видео | Defer | Remote Video/Video Media уже доступны; использовать только при контенте |
| Код в статьях | Adjust to Varbase | Семантический code block в v1; syntax highlighting не обещать |
| Автоматическое оглавление | Defer | Штатно не подтверждено; ручные anchors допустимы |
| RU/EN | Keep as-is | Независимая публикация переводов подтверждена |
| Обязательный перевод | Drop | Противоречит принятому правилу и усложняет workflow |
| Поиск | Adjust to Varbase | Search API DB, current language, без Solr/facets/autocomplete |
| SEO | Keep as-is | Metatag/Pathauto/Redirect/Sitemap/Schema уже входят |
| Contact | Keep as-is | Reusable Webform + mail/anti-spam |
| Questions/corrections | Adjust to Varbase | Приватный Webform, не public Comment |
| Public comments | Defer | Нет подтверждённого community use case; spam/moderation cost |
| Workflow | Keep as-is | Draft → In review → Published → Archived подтверждён |
| Author/publisher split | Requires later decision | Baseline роли publish; split делается configuration-only |
| Registration | Defer | Нет полезного кабинета/закрытого контента; лишняя attack surface |
| Разграниченный доступ | Defer | Нет Member/Internal published-content use case |
| Личный кабинет | Defer | Не определены данные, действия и business value |
| Community/forum | Defer | Отдельный продуктовый и moderation scope |
| Editorial AI-agent | Defer | Публичному launch не нужен; API/security boundary готова концептуально |
| Beget | Adjust to Varbase | Shared-first только после qualification; VPS по triggers |
| Git/GitHub | Keep as-is | Source control и short-branch workflow обязательны |
| Документация | Keep as-is | Репозиторий — authority; GitHub Wiki может быть только представлением |
| Platform first, custom last | Keep as-is | Принято ADR-0001 и подтверждено всеми исследованиями |
| E-commerce | Drop from this site | Если появится — отдельный сайт/subsite и отдельная архитектура |

## 4. Приоритеты

### Must have for v1

- Главная, О компании, Управление активами, Консалтинг, Статьи, Объекты,
  Контакты, Search, Privacy/legal pages;
- понятное представление четырёх consulting competencies;
- Technical Article, Asset и утверждённый способ представления Services;
- RU default и EN optional translations;
- Image/Document/Remote Video media policy;
- Search API DB с current-language выдачей;
- SEO metadata, aliases, redirects, sitemap и schema mappings;
- Contact и private Article Feedback Webforms;
- штатные роли и editorial workflow;
- registration/comments/API mutations выключены;
- production readiness fixes, Composer/Git/config/backup/cron/minimal CI.

### Should have shortly after launch

- golden query set и корректировка search relevance;
- первые Remote Video материалы, если есть редакционный план;
- улучшение редакторского руководства и переводческого контроля;
- измеренная Webform retention/anti-spam настройка;
- временный staging restore перед первым update;
- MFA, если владелец явно не включит её в launch scope;
- оценка постоянного staging после появления регулярных обновлений.

### Future backlog

Полный backlog вынесен в
[epe-master-future-backlog.md](epe-master-future-backlog.md). Будущая функция
не должна расширять v1 без отдельного owner decision и acceptance criteria.

## 5. Основные разделы и page-building boundary

| Раздел | Рекомендуемый механизм | Решение |
| --- | --- | --- |
| Главная | Canvas Page | Уникальная композиция, patterns, Views blocks, CTA |
| О компании | Canvas Page | Уникальная evergreen page |
| Управление активами | Canvas Page + Asset View | Narrative landing + структурированный каталог |
| Консалтинг | Canvas Page | Landing и навигация по competencies |
| IT Consulting | Вложенная Canvas Page или Service | Service только если нужна общая серия/связи |
| Project Management Consulting | Вложенная Canvas Page или Service | Не создавать новый bundle |
| Business Consulting / Interim Management | Одна вложенная Canvas Page или Services | Объединить близкие предложения в IA |
| Статьи / Knowledge Base | View + Technical Article template | Не свободная Canvas Page на каждую статью |
| Объекты | View + Asset template | Каталог/filters должны идти из fields |
| Контакты | Canvas Page + reusable Webform | Поля формы не копировать в Canvas |
| Search | Search API View | Отдельно от Asset filters |
| Privacy/legal | Обычная Page или простая Canvas Page | Выбрать один простой механизм без special bundle |

IT/PM/Business pages не обязаны быть тремя production Content Types или даже
тремя отдельными routes. На старте это вложенные Canvas Pages/секции. Service
entities добавляются только если опубликовано несколько повторяемых предложений
с единым набором полей, related Articles и общим listing/template.

## 6. Production Content Types

### 6.1 Technical Article — обязателен

Назначение: основной экспертный и knowledge-base контент.

| Аспект | Требование v1 |
| --- | --- |
| Поля | title; summary/description; formatted body; featured Image; Topics; Technologies; author; created/changed; SEO fields |
| RU/EN | text/SEO/alias translatable; taxonomy/reference/Media/author/date shared |
| Revisions | обязательны, revision log |
| Moderation | полный общий workflow; translations publish independently |
| SEO | TechArticle, canonical, OG/Twitter, hreflang |
| Search | title boost, summary/body/rendered content, topics, technologies, dates/langcode |
| Display | один Canvas Content Template; override только исключительной trusted role |
| Sitemap | каждый опубликованный перевод; drafts/archived исключены |
| API | disabled в v1; позже explicit read/write allowlist для AI |

CKEditor v1: headings, tables, links, quotes, embedded Media и semantic code
blocks. Syntax highlighting и automatic TOC отложены.

### 6.2 Service — условно нужен

Назначение: повторяемое структурированное предложение компании, связанное со
статьями. Не использовать как замену каждой маркетинговой Canvas Page.

| Аспект | Требование v1 |
| --- | --- |
| Поля | title; short/full description; Service Category; featured Image; related Technical Articles; SEO |
| RU/EN | text/SEO/alias translatable; category/articles/Media shared |
| Revisions/moderation | обязательны, общий workflow |
| SEO | Schema Service после включения установленного plugin; bundle defaults |
| Search | индексировать только если Service entities утверждены |
| Display | общий Canvas Content Template; marketing landing может ссылаться на entities |
| Sitemap | включить при наличии production Services |
| API | disabled в v1 |

Owner должен подтвердить: есть ли на launch серия отдельных услуг, которой
нужны listing, related Articles и одинаковый lifecycle. Если нет — **не создавать
Service в v1**, использовать Canvas Pages.

### 6.3 Asset — обязателен

Назначение: структурированная карточка объекта и источник каталога.

| Аспект | Требование v1 |
| --- | --- |
| Поля | title; description; area decimal; address text; Asset Status; multiple Images; SEO |
| RU/EN | title/description/SEO/alias translatable; area/address/status/images shared по умолчанию |
| Revisions/moderation | обязательны, общий workflow |
| SEO | RealEstateListing + page/Organization references |
| Search | title/description/status/area; каталог фильтруется Views, не full text |
| Display | общий Canvas Content Template; gallery/media from fields |
| Sitemap | опубликованные доступные страницы объектов |
| API | disabled в v1 |

Адрес остаётся строкой. Address/Geofield/map/geocoding не нужны без требований
к компонентам адреса, карте или поиску по расстоянию.

### 6.4 News — не нужен

Статус: **Drop from v1**. Новости компании можно публиковать как Technical
Article с Topic/форматом, если это экспертный материал. Короткие announcements
до появления регулярного news lifecycle размещаются Canvas content. Отдельный
News bundle создаст дублирующие fields, Views, SEO и workflow без контента.

Trigger пересмотра: регулярная новостная лента с отличным авторским процессом,
датой события/press metadata и отдельной syndication policy.

### 6.5 Page и Canvas Page

- **Canvas Page** — default для уникальных corporate/landing pages.
- Обычный **Page** сохранять только для простых evergreen/legal pages, если
  plain body workflow действительно проще Canvas.
- Не создавать новый custom Page type. Использовать уже естественный baseline,
  но до content migration выбрать один механизм для legal pages.
- Canvas Page: title, alias, language, SEO/social image и component inputs;
  общий symmetric RU/EN tree; revisions/moderation; sitemap только для
  публичных evergreen pages; API disabled.

## 7. Taxonomy v1

| Vocabulary | v1 | Иерархия | Multilingual | Governance |
| --- | --- | --- | --- | --- |
| Topics | Да | максимум 2 уровня после реальной потребности | shared term identity/tree, translated name/description | Content Admin; editor выбирает existing terms |
| Technologies | Да для технических статей | flat | shared identity, translated label | Content Admin; controlled vocabulary; tech tokens сохранять явно |
| Service Categories | Только если Service CT утверждён | flat на v1 | shared identity, translated label | Content Admin |
| Asset Status | Да, если больше одного состояния | flat controlled enum | shared business state, translated label | Content Admin/Site Admin, editors не создают terms ad hoc |

Если Asset Status имеет 2–4 неизменяемых значения и не требует editorial term
management, Options field проще taxonomy. Owner/content owner выбирает после
утверждения статусов. Не создавать отдельные RU/EN terms.

## 8. Media v1

| Media type | Policy |
| --- | --- |
| Image | Основной тип; shared file/reference, alt/caption translatable при необходимости |
| Remote Video | YouTube/Vimeo-подобные embeds только при реальном контенте/privacy review |
| Video file | Не baseline; включать только при подтверждённом self-hosting use case |
| Document/File | PDF/technical files; public/private decision per document |
| SVG | Только trusted upload, sanitization/security review и реальная brand/diagram need |
| Audio/прочее | Defer |

Production image policy: long edge обычно <=2560 px, <=5 МБ; hero заранее
оптимизировать; upload limit начать с 20–32 МБ; lowercase ASCII filenames;
responsive styles, WebP и lazy loading; AVIF не обещать до pipeline test.
Один файл используется для RU/EN. Language-specific creative — отдельный Media
только при подтверждённой маркетинговой необходимости.

## 9. Multilingual acceptance policy

- default language — RU;
- RU URLs без prefix: `/...`;
- EN URLs: `/en/...`;
- browser language не делает обязательный redirect;
- перевод необязателен, публикация оригинала независима;
- public Views/Search показывают только published current language;
- direct original-language URL разрешён, даже если пользователь пришёл из
  другого языка; не создавать fake fallback translation;
- text, labels, body, summary, SEO, alias — translatable;
- numbers, dates, author, taxonomy/entity/Media references — shared;
- taxonomy tree и term identity shared, name/description translated;
- Media file/reference shared, textual metadata translated при необходимости;
- Canvas tree symmetric; inputs translated; разные RU/EN layouts не являются
  поддерживаемой нормой;
- self canonical; hreflang только между опубликованными переводами.

Unresolved: точный language-switcher UX при отсутствии перевода и
редакционный процесс пометки устаревшего перевода.

## 10. Search v1

- Search API остаётся abstraction;
- backend — Database;
- один index для published nodes и Canvas Pages;
- индексировать structured fields и curated rendered content;
- View по умолчанию фильтрует current language;
- optional all-languages mode — только после UX decision;
- Asset catalog filters остаются Views filters, не site-search facets;
- RU morphology и `ё/е` baseline ограничены;
- `C#`, `CI/CD`, `ИТ` и подобные tokens не обещать без analyzer/golden tests;
- Solr не принят.

Solr trigger: обязательная RU morphology/protected vocabulary, facets,
autocomplete/spellcheck, неудовлетворительный p95/relevance или search load на
основную DB. Solr также означает VPS/managed service decision.

## 11. Roles, workflow и registration

| Role | v1 назначение |
| --- | --- |
| Anonymous | published content, search и allowed Webforms |
| Content editor | создаёт/редактирует материалы и отправляет In review; trusted small-team baseline |
| Content admin | владеет content, workflow, taxonomy, patterns и submissions |
| SEO admin | trusted SEO/content role, не read-only metadata specialist |
| Site admin | users, operational site и Canvas governance без permissions/config authority |
| Super Admin | крайне ограниченный technical configuration/recovery |

Не создавать Member/Internal/Customer roles. Authenticated — техническая core
роль, не business access tier. UID 1 — break-glass, не ежедневный аккаунт.

Workflow: **Draft → In review → Published → Archived**. Revisions и revision log
обязательны. Split author/publisher для v1 требует owner approval: если review
обязателен, убрать Publish/Archive у Content editor и оставить Content admin;
новая workflow-система/custom не нужны.

Registration остаётся `admin_only`; personal cabinet отсутствует. Registration
пересматривается только при наличии хотя бы одного утверждённого сценария:

- пользователь сохраняет материалы/объекты или управляет подписками;
- существует закрытый paid/member/internal content;
- пользователь видит историю собственных обращений/заказов;
- community требует persistent identity;
- есть owner, privacy/account lifecycle и support process.

## 12. Contact, feedback и anti-spam

### Webform Contact

Имя, email, тема, сообщение, consent; anonymous access; reusable Webform Block
на Canvas Contact page. Domain From, visitor email только Reply-To.

### Private Article Feedback

Тип: Question, Correction, Suggestion, Comment/other; message; source article;
email optional where follow-up is not promised; consent. Anonymous allowed.
Форма встраивается в Technical Article template, чтобы source entity был
надёжно передан.

Comments disabled. Submissions исключены из Search, public API и AI. Results и
exports доступны только Content Admin/Site Admin.

Anti-spam minimum: Antibot + Honeypot; per-IP/total Webform limits после
реальных volume expectations; FriendlyCaptcha только при abuse; не включать
несколько CAPTCHA одновременно. Attachments выключены в v1.

Privacy retention и хранение IP требуют owner/legal decision; 180 дней — лишь
исследовательский ориентир, не утверждённая политика.

## 13. SEO acceptance policy

- Metatag — title/description/canonical/hreflang;
- Pathauto — стабильные bundle patterns без изменчивых taxonomy terms;
- Redirect — автоматический single-hop 301 при изменении alias;
- Simple XML Sitemap — только утверждённые bundles/pages и published
  translations;
- Schema Metatag mappings: TechArticle, Service (если Service утверждён),
  RealEstateListing, Organization, BreadcrumbList;
- Open Graph/Twitter Cards — translated text, shared/appropriate image;
- Yoast/Realtime SEO — редакторская подсказка, не publication gate.

Must-fix production configuration:

1. устранить canonical conflict `/home` versus `/`;
2. включить production bundles в sitemap и проверить hreflang;
3. убрать misleading canonical с 404 и при необходимости поставить noindex;
4. согласовать включение уже установленного `schema_service`;
5. настроить фактические Schema mappings и проверить rendered JSON-LD.

## 14. Non-functional v1 requirements

- Composer install строго из `composer.lock`, `--no-dev`, platform checks;
- один source-of-truth checkout;
- versioned Drupal config, повторяемые export/import/status;
- Git `main` + short branches + PR + release tag; без `develop`;
- cron каждые 5–15 минут и контроль stalled queues;
- проверяемый DB/public/private backup и offsite copy;
- целевой RPO 24 часа, RTO 4–8 часов;
- temporary staging restore перед update;
- minimal CI: validate, audit, no-dev build, config validation, smoke;
- no Redis/CDN/Solr by default;
- no custom modules/theme/code without proven gap and ADR.

## 15. Traceability к исследованиям

| Решение v1 | Evidence |
| --- | --- |
| Canvas Pages, templates, patterns и governance | [Исследование Canvas](../research/page-building-canvas-design-system.md), [Structured Content](../research/02-structured-content.md) |
| Technical Article / Service / Asset и Views | [Structured Content](../research/02-structured-content.md) |
| RU `/`, EN `/en`, optional translation, shared facts | [Multilingual](../research/03-multilingual.md) |
| Search API DB, current language, Solr triggers | [Search](../research/04-search.md) |
| Roles, workflow, registration off, AI role boundary | [Users/Roles/Workflow](../research/05-users-roles-workflow-access.md) |
| SEO, private feedback, anti-spam и privacy gaps | [SEO/Webform/Feedback](../research/06-seo-forms-feedback.md) |
| Closed API, draft-only agent, Recipes/config/ECA | [API/AI/Recipes](../research/07-api-ai-recipes-integrations.md) |
| Beget, config blockers, Composer, backup/RPO/RTO | [Production Readiness](../research/08-production-readiness-beget.md) |

Полная сводка статусов находится в
[capability map](../research/varbase-capability-map.md).

## 16. Acceptance boundary

Этот документ не создаёт content types/config и не утверждает owner-dependent
решения. Реализация v1 может начаться только после устранения development
blockers из architecture document и подтверждения owner decisions.
