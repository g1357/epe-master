# E+E Master v1: целевая архитектура

- Статус: Approved implementation baseline
- Дата: 2026-08-21
- Реализация: не начата
- Baseline: Varbase 11.0.0-rc1 / Drupal 11.4.5 / PHP 8.4

## 1. Architecture outcome

E+E Master v1 — Composer-managed Varbase/Drupal monolith с двумя слоями
публикации:

```text
Unique corporate narrative
  Canvas Page → approved patterns/SDC → global header/footer

Repeatable domain content
  Technical Article / optional Service / Asset
    → typed fields + Media + Taxonomy
    → Views/Search/Workflow/SEO
    → shared Canvas Content Template
```

Это сохраняет структурированные данные для поиска, multilingual, SEO и
будущего API, не превращая каждую страницу в custom layout. Custom module,
theme, component и API для v1 не требуются.

## 2. Architecture principles

1. Varbase 11 и Drupal 11 — platform baseline; ADR-0001 сохраняется.
2. Platform first, custom last: core → included contrib → mature contrib →
   adjustment of requirement → custom только после доказанного gap и ADR.
3. Content model — источник business data; Canvas — presentation/composition.
4. Один source of truth для code/config/docs; production не редактирует
   authoritative config вручную.
5. Public-by-design v1: нет registration, member access, comments и open API.
6. Перевод optional; язык не должен блокировать публикацию оригинала.
7. Infrastructure добавляется по measured trigger, не «на будущее».

## 3. Information architecture

Верхний уровень:

- Главная;
- О компании;
- Управление активами;
- Консалтинг;
- Статьи / Knowledge Base;
- Объекты;
- Контакты;
- Search;
- Privacy / legal.

Внутри Consulting:

- IT / industrial software development / training;
- Project Management Consulting for SMB;
- Business Consulting / Interim Management.

Эти элементы — navigation concepts, не обязательные отдельные bundles. Они
могут быть секциями/вложенными Canvas Pages. Не проектировать детальные URL до
content inventory и approved naming.

## 4. Canvas и structured-content boundary

### Canvas Page

Использовать для home, company, direction landings, contact composition и
уникальных campaign/evergreen pages. Canvas Page не хранит площадь, статус,
отношения и другие queryable business facts.

### Canvas Content Template

Один централизованный template на bundle/view mode для Technical Article,
Asset и Service, если он утверждён. Fields связываются с component props.
Изменение template обновляет presentation всех entities.

### Patterns

Approved повторно вставляемые композиции: hero, intro, cards, FAQ, CTA, contact,
listings. Pattern после вставки — независимая копия; изменение исходного pattern
не является массовым content update. Новые patterns создавать только после
подтверждения, что 14 baseline patterns не покрывают композицию.

### Global regions

Header/footer и общесайтовая navigation structure. Управляют Site Admin и
ограниченный Content Admin; обычный Content editor не меняет global regions.

### Per-node Canvas Override

Исключение для подтверждённой presentation need. Право только trusted
Content Admin/Site Admin; каждое override документируется. Массовые overrides
считаются признаком неверного template/content model.

### Governance rule

Общий layout задаётся централизованно. Обычный редактор работает с fields,
body, Media и approved Canvas Pages/pattern instances, но не меняет shared
templates/global regions. Это правило подтверждено исследованиями №1, №2 и №5.

## 5. Design and theming path

Рекомендуемый путь:

```text
Vartheme BS5
  → Varbase Design System / existing SDC
  → Storybook states
  → Canvas patterns/templates
  → E+E Master branding configuration
```

Сначала применяются logo, palette, typography и композиция существующими
механизмами. Новая тема не создаётся только ради смены цветов.

Generated project theme становится оправданной, когда несколько изменений
должны быть versioned и upgrade-safe как единый design layer:

- logo/brand assets требуют template/library integration;
- palette и typography становятся design tokens;
- button/card variants отсутствуют среди SDC;
- header/footer требуют markup/behavior, недоступный Canvas regions;
- spacing/rhythm нельзя выразить approved utilities;
- нужен controlled component extension или visual regression ownership.

Trigger — документированный gap после brand prototype, а не сам факт наличия
брендбука.

## 6. Content/data architecture

Production model описан в
[v1 requirements](../requirements/epe-master-v1-requirements.md):

- Technical Article — обязателен;
- Asset — обязателен;
- Service — отдельный Content Type в v1;
- News — исключён из v1;
- Canvas Page — уникальные pages;
- Page — только простые legal/evergreen pages при явном UX преимуществе.

Taxonomy ограничена Topics, Technologies и Service Categories. Asset Status —
фиксированный Options enum, не Taxonomy. Media — Image, Remote Video и
Document; SVG trusted-only.

## 7. Multilingual architecture

Окончательная рекомендуемая policy:

- RU default; RU без prefix; EN `/en`;
- translation optional, independent moderation/revisions;
- public listings/search — current published language;
- direct original-language URL fallback, но не fallback row в localized list;
- textual and SEO fields translated;
- factual scalar, author, dates, taxonomy/entity/Media references shared;
- terms share identity/tree and translate name/description;
- one Media/file shared; metadata translated only when needed;
- Canvas component tree, templates и regions symmetric; inputs translated;
- self canonical; hreflang только для существующих published translations.

Не поддерживать разный RU/EN Canvas layout как норму. Если campaigns реально
различаются, это отдельные Canvas Pages, а не обход symmetric translation.

Если перевода текущей страницы нет, language switcher ведёт на главную
выбранного языка. Устаревшие переводы в v1 отслеживаются редакционно вручную.
Финальные Pathauto patterns утверждаются при реализации content model.

## 8. Search architecture

```text
Content entities + Canvas Page
  → Search API index
  → Database backend
  → current-language Search View
```

Index включает published/access-allowed content, title, summary/body/rendered
content, bundle, langcode, changed, Topics, Technologies, category и Asset
fields. Asset catalog использует Views filters; не смешивать его с global
relevance search.

DB backend принимается для launch при real-corpus acceptance. Известные gaps:
RU morphology/`ё-е`, protected technical tokens и отсутствие facets,
autocomplete/spellcheck. Сначала golden RU/EN query set и structured fields.
Solr только по критериям из requirements; custom search запрещён до оценки
Search API/Solr.

## 9. Access and editorial architecture

```text
Anonymous → published content/search/forms
Content editor → authoring + Review + Publish для trusted launch team
Content admin → content/workflow/taxonomy/pattern/submission owner
SEO admin → trusted content/SEO role
Site admin → users/operations/Canvas governance
Super Admin → configuration/recovery; minimal assignment
UID 1 → break-glass
```

Workflow: Draft → In review → Published → Archived. Published revision остаётся
live, пока следующая Draft/In review не опубликована. Revisions, Diff, content
lock и audit сохраняются.

Не создавать business роли Member/Internal и не считать core Authenticated
закрытым access tier. Registration `admin_only`, no personal cabinet.
Обязательного four-eyes разделения на launch нет; trusted Content editor может
публиковать. Переходы workflow остаются Draft → In review → Published →
Archived, а ужесточение permissions возможно позже без смены архитектуры.

## 10. Feedback, forms and privacy boundary

- reusable Contact Webform на Canvas Contact page;
- private Article Feedback в Technical Article template;
- Question, Correction, Suggestion, Comment/other;
- anonymous access, optional email where appropriate;
- Antibot + Honeypot; limits and FriendlyCaptcha by measured abuse;
- no attachments in v1;
- comments off;
- submissions private and excluded from Search/JSON:API/AI;
- authenticated SMTP, domain From, visitor email only Reply-To;
- access to Results/Exports only Content Admin/Site Admin.

PII минимизируется. Contact: name, email и organization optional, message и
consent required; Article Feedback email optional. IP не хранится, если этого
не потребует anti-abuse. Retention baseline — 180 дней, предварительно и до
отдельной legal review. No PII is sent to external AI.

## 11. SEO architecture

| Layer | Mechanism |
| --- | --- |
| Metadata/canonical/hreflang | Metatag |
| Aliases | Pathauto |
| Changed URLs | Redirect, one-hop 301 |
| Discovery | Simple XML Sitemap + robots sitemap directive |
| Structured data | Schema Metatag |
| Social | Open Graph/Twitter Cards |
| Editorial hints | Yoast/Realtime SEO |

Schema graph: Organization/WebSite globally; BreadcrumbList; TechArticle;
RealEstateListing; Service после проверки и явного включения установленного
`schema_service`. Fix `/home` canonical, 404 canonical,
sitemap bundles and rendered JSON-LD before launch.

## 12. AI and API boundary

AI не входит в public launch.

### Layer A: internal assistive AI

Human-triggered suggestions in CKEditor/Media/taxonomy. Оно наследует human
session и требует provider/privacy/cost/evaluation policy. Не публикует
автономно.

### Layer B: external editorial draft-only agent

Будущий separate runtime:

```text
Search API View retrieval → JSON:API entity read/write
  → OAuth Client Credentials
  → dedicated machine user / narrow role
  → Draft → In review
  → human review and Publish
```

Required audit: revision author, revision log/correlation ID, OAuth/application
logs, Admin Audit Trail. Agent запрещены Publish/Archive/Delete, users, roles,
permissions, config, secrets, modules, recipes, updates, schema, workflow,
Canvas templates/patterns/regions/overrides, taxonomy creation, Webforms/PII,
menus, aliases, redirects, global SEO and provider keys.

JSON:API остаётся `default_disabled` и globally read-only в v1. Future
resources/mutations включаются explicit allowlist и ADR. Search API/View даёт
ranked retrieval; JSON:API не заменяет search endpoint.

## 13. Recipes, config and ECA

- Recipes = bootstrap/composition, не ongoing ownership и не rollback;
- exported Drupal config = authoritative lifecycle state;
- Composer lock = authority для code dependencies;
- ECA = selective config automation для сложных events;
- simple task сначала решает профильный mechanism: Webform handler, Scheduler,
  Content Moderation Notifications, Search API cron.

`epe_master_base` имеет смысл позже, после approved bundles/fields/languages,
workflow, Views/Search, Webforms, SEO, roles и стабильного versioned config. Оно
поможет bootstrap чистую среду, но дальнейшие изменения deploy-ятся config
management. Data/schema transformations при необходимости идут migrations или
update hooks, не повторным Recipe apply.

## 14. Production architecture

### Hosting

Принято: Beget shared-first после qualification. Обязательно подтвердить
PHP 8.4 web/CLI и extensions, MySQL 8 (не 5.7), Composer 2/build strategy, SSH,
cron, symlink/DocumentRoot `web`, private path, 256 МБ+ memory, ImageMagick,
SSL/mail/backups. Если обязательный пункт не проходит — небольшой Beget VPS.

VPS triggers: Solr, Redis, long-running workers/AI, recurrent cron/Composer/updb
limits, memory/CPU/image failures, отсутствие required stack или measured
traffic/SLO failure.

### Environments and Git

- Local: DDEV;
- temporary Staging: production backup clone before update;
- Production: Beget;
- persistent Staging только по trigger;
- `main` + short branch + PR + review + tag + deploy; no `develop`;
- deploy exact tag, Composer install from lock, updb, cim, cr, cron, smoke;
- DB-changing rollback = previous code + DB/files restore, не только Git switch.

### Runtime baseline

- page/dynamic/render/Views caches and aggregation;
- cron every 5–15 minutes, queues monitored;
- no Redis/Memcached/CDN/reverse proxy/Solr by default;
- authenticated SMTP with domain From, visitor Reply-To only,
  SPF/DKIM/DMARC and delivery test;
- verbose errors/assertions/dev services off;
- trusted hosts, hash salt, private/temp paths and permissions;
- secrets outside Git/config export.

### Backup and recovery

Backup DB, public/private files, config/release metadata, secrets/OAuth keys
separately. Provider backup + nightly DB/daily files + encrypted offsite; 7
daily, 4 weekly, 3 monthly; quarterly restore drill and before releases.
Target: RPO 24h, RTO 4–8h. Local DB restore experiment passed.

## 15. Delivery blockers and sequencing

### Must fix before development continues

1. **Single source-of-truth checkout.** Windows Git branch и active WSL DDEV
   являются разными clones; выбрать/синхронизировать один рабочий checkout.
2. **Versioned config sync.** Текущий sync находится в ignored public files;
   перенести в repository path и определить settings override.
3. **Importable config.** Удалить/исправить orphaned `tour.tour.honeypot` после
   review; `drush cim` сейчас validation-fails.
4. **Canvas config convergence.** Восемь patterns/template показывают drift
   после export; найти штатную normalization/serialization cause.
5. Owner approvals зафиксированы ADR-0002, ADR-0003, ADR-0004 и ADR-0006.

Эти пункты нужны до production entities: иначе config нельзя безопасно
review/deploy и две рабочие копии будут расходиться.

### Must fix before production

- продолжать на текущем Varbase 11 RC baseline; перед production проверить
  доступный stable и обновиться, если это совместимо и оправдано;
- устранить Composer advisory `paragonie/sodium_compat` и оценить abandoned
  package; `composer audit` должен быть clean/accepted;
- production settings/secrets/trusted hosts/private files/permissions;
- qualify конкретный Beget shared account или выбрать VPS;
- permissions review каждого final bundle;
- внедрить обязательную MFA для Site Admin / Super Admin;
- финальная RU/EN negotiation, Views/Search filters и switcher UX;
- SEO fixes: `/home`, 404, sitemap, Schema, domain robots;
- legal review предварительного retention 180 дней и production SMTP test;
- cron/queue/logging/backup/restore/smoke;
- minimal CI.

### Can defer

- registration/cabinet/member access/community/forum;
- AI/API mutation, Solr/facets/autocomplete;
- Redis/CDN/permanent staging/external observability;
- custom/generated theme until branding trigger;
- syntax highlighting/TOC/maps/video library;
- RPO 1h/RTO 1h.

## 16. Minimal CI and release gates

1. `composer validate --no-check-publish`;
2. `composer audit --locked`;
3. clean `composer install --no-dev` + platform checks;
4. disposable Drupal boot + `updb` + `cim` + clean config status;
5. smoke: homepage, RU/EN, Article, Canvas, Asset, Search, forms/mail, login,
   workflow, sitemap, cron, API closed;
6. code standards only after custom-code appears.

## 17. Architecture quality attributes

- **Maintainability:** platform configuration before code; centralized layouts;
- **Security:** public-only launch, least privilege, closed API, no secrets Git;
- **Recoverability:** tagged releases and tested DB/files restore;
- **Portability:** Composer/config build works on qualified shared or VPS;
- **Accessibility/performance:** existing SDC/responsive media/core cache, then
  test actual pages; Canvas editor stability requires smoke coverage;
- **Auditability:** revisions, moderation, named users and Admin Audit Trail.

## 18. Owner decisions accepted

Девять owner decisions приняты 2026-08-21 и отражены в ADR-0002, ADR-0003,
ADR-0004 и ADR-0006. Единственный оставшийся внешний gate среди них — legal
review предварительного retention baseline 180 дней. Эти решения разрешают
implementation baseline, но не расширяют утверждённый scope v1.
