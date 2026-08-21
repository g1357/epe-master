# Исследование №7: API, AI, Recipes и интеграции

- Статус: практический проход завершён
- Дата: 2026-08-21
- Baseline: Varbase 11.0.0-rc1 / Drupal 11.4.5 / PHP 8.4.22
- Среда: локальный DDEV, full-вариант Varbase
- Ограничения: без custom-кода, новой темы, новых contrib-модулей и реальных
  provider credentials

## 1. Цель и границы

Исследование проверяет, может ли будущий editorial AI-agent E+E Master работать
на штатной entity/API/workflow-инфраструктуре Varbase без собственной API
платформы. Production API не открывался. Consumer, machine user, OAuth secret и
production Recipe не создавались.

Единственное временное изменение — enabled resource override для
`node--research_technical_article`. После теста override удалён, cache rebuilt,
а endpoint снова подтверждён как 404. Webform submission API всё время оставался
закрыт.

Официальные источники:

- [Drupal core JSON:API](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module);
- [JSON:API revisions](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/revisions);
- [JSON:API translations](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/translations);
- [Simple OAuth](https://www.drupal.org/project/simple_oauth);
- [OpenAPI for JSON:API](https://www.drupal.org/project/openapi_jsonapi);
- [Drupal Recipes](https://www.drupal.org/docs/extending-drupal/drupal-recipes/how-to-download-and-apply-drupal-recipes);
- [ECA](https://www.drupal.org/project/eca).

## 2. Точный API baseline

| Компонент | Состояние | Версия |
| --- | --- | --- |
| JSON:API | enabled, read-only | Drupal core 11.4.5 |
| REST | enabled, resources не настроены | Drupal core 11.4.5 |
| Serialization | enabled | Drupal core 11.4.5 |
| JSON:API Extras | enabled | 8.x-3.28 |
| JSON:API Defaults | enabled | 8.x-3.28 |
| Consumers | enabled | 8.x-1.24 |
| Simple OAuth & OpenID Connect | enabled | 6.1.1 |
| OpenAPI | enabled | 8.x-2.3 |
| OpenAPI JSON:API | enabled | 3.0.6 |
| OpenAPI REST | enabled | 8.x-2.0 |
| OpenAPI UI / ReDoc / Swagger | enabled | 8.x-1.0-rc5 / 8.x-1.0-rc5 / 8.x-1.2 |
| REST UI | enabled | 8.x-1.22 |
| Admin Audit Trail | enabled | 1.0.10 |

Фактическая конфигурация:

```yaml
jsonapi.settings:
  read_only: true

jsonapi_extras.settings:
  path_prefix: api
  include_count: false
  default_disabled: true
  validate_configuration_integrity: true
```

Это более безопасная политика, чем default Drupal core: route существует только
для ресурса с явным enabled resource config, а mutation глобально запрещена.

## 3. Исходная закрытая модель

До эксперимента таблица Enabled Resources была пустой. Все resource types,
включая nodes, Media, taxonomy, users, consumers, OAuth tokens, Webforms и
Webform submissions, отображались как disabled by default.

Anonymous GET:

- `/api/node/research_technical_article` → 404;
- `/jsonapi/node/research_technical_article` → 404;
- `/api/webform_submission/research_06_contact_prototype` → 404.

`/api` — единственный префикс. Стандартный `/jsonapi` не используется. Header
`X-Consumer-Id: default_consumer` присутствует даже в anonymous response из-за
Consumers middleware, но не означает OAuth authentication.

REST UI показывает потенциальные plugins, но у всех operation — `Enable`;
active `rest.resource.*` config отсутствует. Следовательно, REST routes не
являются скрытым вторым API.

## 4. Временный read-only JSON:API эксперимент

### 4.1 Метод

Штатная форма JSON:API Extras дважды вернула на homepage и не создала config
entity. Глобальный `default_disabled` не менялся. Для воспроизводимого теста
стандартным Drupal config-entity API создан один override со всеми 41 текущими
полями bundle. Файлы и custom-код в проект не добавлялись.

После тестов удалён config
`jsonapi_extras.jsonapi_resource_config.node--research_technical_article`,
выполнен cache rebuild и endpoint снова дал 404.

### 4.2 Collection и entity

В коллекции anonymous появились только три опубликованные default revisions:

- EN node 15;
- EN node 19;
- RU node 20.

Draft EN/RU node 18 и Archived node 21 отсутствовали. Каждый resource получил:

- стабильный Drupal UUID в JSON:API `id`;
- `drupal_internal__nid` и `drupal_internal__vid`;
- `langcode`, `status`, `title`, timestamps;
- `moderation_state`;
- structured fields и SEO fields;
- relationship endpoints;
- `self` и `working-copy` links.

Collection filtering `filter[langcode]=ru` оставило RU representation.
`Accept-Language: ru` само по себе не отфильтровало коллекцию: в ответе остались
EN и RU entities. `/ru/api/...` дал 404, потому что принятую будущую RU `/`, EN
`/en` negotiation policy на текущем сайте ещё не применяли.

### 4.3 Relationships

OpenAPI и JSON:API описали relationships:

- `field_featured_image` → Media Image;
- `field_seo_image` → Media Image;
- `field_research_topics` → Research Topics;
- `field_research_technologies` → Research Technologies;
- author/revision author/node type/menu link.

Однако related resource types оставались disabled. У node 15 в БД есть Media
target 60, но JSON:API вернул `field_featured_image.data: null`. Значит, одного
article resource недостаточно: чтобы агент видел UUID разрешённых Media/terms,
нужно отдельно включить их read-only resource configs. Это правильный fail
closed результат.

## 5. Revisions, moderation и access

Node 15 имеет опубликованную default revision 15 и более новую forward Draft
revision 18. Anonymous результаты:

- обычный entity GET → 200, revision 15, `moderation_state=published`;
- `resourceVersion=rel:working-copy` → 403;
- `resourceVersion=id:18` → 403.

Таким образом, ссылка на working copy не раскрывает draft: Drupal revision
access проверяется при чтении. Это важнее сокрытия самой ссылки.

Core JSON:API рассматривает default revisions как resource versions, а
non-default revisions — как working copies. Для Node/Media версии можно читать
при наличии revision access. Если bundle настроен всегда создавать revisions,
PATCH автоматически создаёт revision. Практический write test не выполнялся,
потому что для него пришлось бы отключить глобальный `read_only`, то есть
разрешить mutation всем enabled resources. Это противоречит границам
исследования.

PATCH пустого document в текущем состоянии вернул 405 с объяснением, что API
принимает только read operations. Никакой content mutation не произошёл.

### 5.1 Moderation state через JSON:API

`moderation_state` присутствует в attributes и теоретически PATCHable. Реальный
transition всё равно проверяется Drupal permissions и workflow. Агенту нужны
только:

- create new draft;
- send to review.

`publish`, `archive`, restore и delete permissions не выдаются. Не нужен custom
workflow endpoint: сначала PATCH entity fields/draft, затем PATCH
`moderation_state=in_review`, с обработкой validation/access errors.

## 6. Multilingual contract

JSON:API использует Drupal language negotiation, а не собственную формальную
translation-модель. Core поддерживает PATCH существующего перевода и POST новой
entity с non-default langcode, но не POST дополнительного перевода к уже
существующей entity.

Практически подтверждено:

- RU и EN single-language published entities могут находиться в одной
  collection;
- language filter работает;
- draft translations не попадают anonymous;
- current `/ru/api` route не существует;
- published bilingual translation с shared UUID в наборе прототипов отсутствует,
  поэтому этот случай не симулировался.

Для E+E Master естественная корректировка workflow: человек или внутренний
Drupal process создаёт пустой translation, после чего внешний агент PATCHит
существующий перевод. Не писать custom translation endpoint только ради одного
удобного POST. Shared fields/references продолжают следовать field translation
policy из исследования №3.

## 7. Authentication, consumers и OAuth

### 7.1 Baseline

Simple OAuth использует dynamic entity scopes. Token endpoint `/oauth/token`
работает и без client parameters корректно возвращает `400 invalid_request`.
Password и implicit grants в Simple OAuth 6 удалены. Consumer form предлагает:

- Client Credentials;
- Authorization Code;
- Refresh Token;
- confidential/public flag;
- access-token lifetime, default нового consumer — 300 seconds;
- hashed secret: исходный secret модуль не хранит.

Scopes являются config entities и могут ссылаться на permissions/roles.
Tokens управляются отдельным административным списком и могут быть revoked;
consumer также можно disable. Client Credentials естественен для
machine-to-machine. Refresh token для него обычно не нужен: короткий access
token переиздаётся по client credentials.

### 7.2 Default Consumer

Есть один active `default_consumer` с UUID
`b8481e57-f4ef-4522-9d9e-25436fb90622`. Его grant types и lifetime не заполнены,
он не является готовой identity агента. Переиспользовать его или UID 1 нельзя.

Consumer/secret не создавались: production account запрещён границами задачи,
а генерация persistent credential не нужна для подтверждения архитектуры.
Secrets должны находиться только в secret manager/environment, никогда в Git и
не в config export.

### 7.3 Authentication methods

- OAuth Bearer — рекомендуемый machine authentication;
- cookie session + CSRF — для браузерного UI, не для background agent;
- anonymous — только явно открытые published read resources;
- Basic Auth module выключен;
- OpenID Connect доступен в Simple OAuth, но для автономного internal consumer
  Client Credentials проще.

Всегда использовать HTTPS; DDEV HTTP допустим только для локального теста.

## 8. Минимальная роль editorial agent

Production role/account не создавались. Рекомендуемая отдельная роль
`Editorial AI agent`:

| Разрешить | Не разрешать |
| --- | --- |
| `access content` | publish/archive/restore |
| create Technical Article | delete own/any content |
| edit own Technical Article | users, roles, permissions, configuration |
| view own unpublished content/revisions | administer content types/fields |
| create new draft transition | Canvas templates, patterns, global regions |
| review transition | Webforms/submissions/PII |
| create/edit own Image Media | create/delete taxonomy terms |
| use approved text format | menu, redirects, aliases administration |
| view/reference approved terms and Media | unrestricted Full HTML or PHP |

Нельзя брать штатный Content editor: он умеет publish/archive, delete any Media,
работать с Canvas/Webforms и значительно шире least privilege.

### 8.1 Существенное ограничение ownership

Требования «агент создаёт forward Draft для существующего опубликованного
материала» и «не редактирует чужой/Published content» конфликтуют. Bundle-level
core permissions не дают простого allowlist конкретных nodes: `edit any`
позволит агенту начать draft для любого материала bundle, а `edit own` — только
для материалов, owner которых сам агент.

Естественные варианты:

1. скорректировать требование и разрешить bundle-scoped `edit any`, но оставить
   только Draft/Review transitions и human publish;
2. агент работает только со своими materials;
3. позже оценить установленный/дополнительный access-control механизм для
   allowlist; custom node access — только последняя мера.

Это существенное решение следует оформить ADR после выбора пользователя.

## 9. Пошаговая модель editorial agent

| Шаг | Штатный механизм | Permission / boundary | Ограничение | Custom |
| --- | --- | --- | --- | --- |
| 1. Список разрешённых материалов | Search API View REST Export | view published + View access | endpoint ещё не настроен | нет |
| 2. Найти материал | Search API ranked query | language/access filters | JSON:API не ranked search | нет |
| 3. Прочитать published | JSON:API GET by UUID | access content | related types включаются отдельно | нет |
| 4. Создать forward Draft | JSON:API PATCH | edit own/any + create draft | глобальный read-only надо осознанно изменить | нет |
| 5. Обновить text/SEO/taxonomy | JSON:API PATCH | field/bundle access | validation и ownership | нет |
| 6. Приложить Media | Media JSON:API + file upload | create/edit own Media | multi-request, не одна transaction | нет |
| 7. Обновить перевод | negotiated PATCH | edit translation | дополнительный translation POST не поддержан | нет при adjusted workflow |
| 8. Отправить Review | PATCH moderation_state | review transition | transition error обрабатывается агентом | нет |
| 9. Human review | Drupal UI/revisions/diff | reviewer role | обязательный шаг | нет |
| 10. Human publish | Drupal moderation UI | publish transition только человеку | агенту запрещено | нет |

## 10. OpenAPI как machine-readable contract

OpenAPI endpoint требует `access openapi api docs`; anonymous получил 403.
Authenticated ReDoc после временного resource enable показал только один tag:
Technical Article, collection/entity GET и GET relationship routes. POST/PATCH
не показывались, что соответствует global read-only.

Schema отразила:

- фактический `/api/node/research_technical_article` server URL;
- UUID entity parameter;
- fields/relationships;
- filter/sort/page/include;
- `resourceVersion` и links на revision docs.

Ограничения baseline OpenAPI 2.0 contract:

- UI не описал OAuth/Bearer security scheme;
- translation/language negotiation не формализована;
- moderation/langcode не видны как понятные workflow concepts;
- generic samples слабо объясняют field validation;
- заголовок ReDoc говорит `Versioning not supported`, хотя query
  `resourceVersion` документирован;
- contract не описывает business sequence Draft → Review.

Вывод: schema полезна для генерации low-level client и обнаружения enabled
routes, но не является достаточным контрактом агента. Нужны versioned
integration tests и короткая project API policy поверх generated OpenAPI. Это
не оправдывает custom API.

## 11. Media

Для будущего агента нужен минимальный набор enabled resources: Article, Image
Media, underlying File и approved taxonomy vocabularies. Media reuse выполняется
relationship UUID. Core JSON:API поддерживает binary upload к file/image field,
но это не проверялось из-за global read-only.

Правила:

- сначала создать/проверить Media, затем PATCH article relationship;
- разрешать только Image Media и ограничения extension/size;
- alt/title принадлежат image field и должны следовать multilingual Media policy;
- public file URL публичен; private files требуют отдельной access проверки;
- не давать delete any Media;
- не создавать отдельный media service;
- при failed upload не менять article и очищать orphan только контролируемой
  задачей/редактором.

## 12. Taxonomy

Vocabulary/terms доступны как JSON:API entities только после явного resource
enable. Agent должен читать UUID и переведённые labels и ссылаться на
существующие terms. Permission создавать/редактировать/delete terms не выдаётся.

Автоматическое создание taxonomy агентом неестественно: оно порождает дубли,
нарушает hierarchy и governance. Новые terms проходят отдельный редакционный
процесс. Текущий baseline позволяет реализовать это только permissions/config.

## 13. Retrieval и Search API

JSON:API умеет filters/sort, но не relevance ranking, stemming, facets и
полнотекстовый retrieval. Текущий Search API View имеет page `/search` и block,
но REST Export display отсутствует.

Естественный future retrieval:

1. добавить к существующему Search API View отдельный read-only REST Export;
2. вернуть только UUID, language, title, excerpt, URL и минимум metadata;
3. применить access и language filters из исследования №4;
4. по UUID получить полную entity через JSON:API.

REST Export — штатный Views/Core REST механизм; custom search endpoint не
нужен. Его contract и ranking нужно тестировать отдельно. Search index не должен
содержать forward drafts для read-only consumer.

## 14. Webform и PII

Все Webform submission resources видны в disabled resource inventory, но active
config для них отсутствует. После эксперимента feedback/contact endpoints дают
404. REST resource Webform Submission также выключен.

Editorial agent по умолчанию не получает:

- Webform permissions;
- submission resources;
- email/IP/consent/message PII;
- raw export.

Если позже агент triage Corrections/Questions, нужен отдельный минимизированный
administrative View/representation с dedicated permission и redacted fields.
Не открывать generic Webform Submission entity API.

## 15. AI baseline

### 15.1 Enabled

| Возможность | Компонент | Версия | Состояние |
| --- | --- | --- | --- |
| Framework/providers | AI Core | 1.4.6 | enabled |
| Assistants/Chatbot | AI Assistant API / Chatbot | 1.4.6 | enabled |
| Field automators | AI Automators | 1.4.6 | enabled |
| CKEditor | AI CKEditor | 1.4.6 | enabled |
| Agents | AI Agents | 1.3.4 | enabled |
| Dashboard | AI Dashboard | 1.0.3 | enabled |
| Image alt | AI Image Alt/Bulk Alt | 1.0.2 | enabled |
| Canvas | Drupal Canvas AI | 1.10.1 | enabled |
| ECA bridge | AI Integration - ECA | 1.0.0-rc3 | enabled |
| Providers | OpenAI / Anthropic / amazee.ai | 1.2.4 / 1.2.2 / 1.4.1 | modules enabled, setup отсутствует |

Provider UI показывает `Setup` для всех трёх: usable API key/default model не
настроены. Реальные prompts/calls не запускались.

Enabled AI Agents configs включают Canvas orchestrator/components/page/template,
content type, field и taxonomy configuration tools. Они полезны Site Builder,
но слишком мощны для editorial agent и не должны входить в его tool allowlist.

### 15.2 Disabled

- AI API Explorer, Content Suggestions, Search, Translate, Logging,
  Observability, Validations;
- AI ECA core integration `ai_eca`;
- AI Integration ECA Agents/Automators submodules;
- AI Agents Explorer/Extra/Extra Tools/Form Integration;
- AI Agent Modes 1.0.0-alpha4;
- Context Control Center 1.0.0-beta4;
- Canvas OAuth/Dev AI;
- Varbase AI Figma 1.0.0-rc1;
- CKEditor Premium AI.

Отдельного MCP/tool-server integration в baseline не обнаружено. Не следует
включать перечисленное только из-за наличия package: сначала use case, security
review и provider/data policy.

### 15.3 Maturity

Core API/revisions/workflow — наиболее стабильная основа. Simple OAuth 6.1.1 —
stable/security-covered. AI ecosystem содержит stable, RC, beta и alpha
компоненты одновременно; Canvas AI и agents активно развиваются. Любой AI
production baseline требует pinned versions, evaluation corpus, cost/privacy
limits и audit, а не только enabled module.

## 16. Внутренний и внешний AI

| Критерий | A. Внутри Drupal | B. Внешний через OAuth/API |
| --- | --- | --- |
| Security | наследует Drupal session/role; риск слишком широких tools | явная machine identity/scope, но появляется secret perimeter |
| Workflow | естественный Entity/Moderation access | нужен корректный JSON:API sequence |
| Provider portability | AI provider abstraction уже есть | зависит от orchestration platform, API Drupal остаётся стабильным |
| Audit | entity revision user, AI logs при включении | dedicated OAuth user + request logs |
| Testing | UI/plugins сложнее изолировать | contract/e2e tests проще |
| Operations | меньше отдельной инфраструктуры | отдельный runtime, retries, secret rotation |
| Cost | provider calls из Drupal | provider + agent runtime |
| Lock-in | Drupal AI/ECA configs | orchestration vendor, но JSON:API стандартен |

Естественный результат — не бинарный выбор, а разделение:

- внутренний AI для human-triggered CKEditor/alt/tag suggestions;
- внешний agent для контролируемого retrieval и Draft → Review orchestration
  под отдельной OAuth identity;
- publish всегда человек.

Это существенная архитектурная рекомендация. Предлагается ADR
«Editorial AI boundary: assistive internal AI + external draft-only agent», но
ADR в этом исследовании не создаётся.

## 17. Recipes

### 17.1 Что применено

`project_browser.applied_recipes` содержит 53 path entries. Среди них:

- `varbase_starter` и базовые Users/Admin/Security/Media/Editor/Content/
  Workflow/SEO/Webform/Page/Blog/Performance/Demo recipes;
- `varbase_dev_base`, `varbase_i18n_base`, `varbase_api_base`,
  `varbase_auth_base`;
- `drupal_cms_ai`, `varbase_ai_image_alt`,
  `varbase_ai_editor_assistant`, `varbase_ai_taxonomy_tagging`,
  `varbase_ai_base`;
- core recipes для roles/media.

Некоторые paths повторяются, например `content_editor_role` и Image Media.
Это реальное свидетельство повторного композиционного применения sub-recipes.

`varbase_api_base` установил API/OAuth/OpenAPI modules, задал `/api`,
`default_disabled=true`, integrity validation и permission Site Admin.
`varbase_ai_base` композиционно применил Drupal CMS AI и Varbase AI recipes.

### 17.2 Где хранятся

- unpacked contributed recipes: project-root `/recipes/...`;
- core recipes: `/web/core/recipes/...`;
- Composer управляет package versions; `drupal/core-recipe-unpack` распаковывает
  recipe packages в проект.

### 17.3 Semantics

Recipe — применяемый набор install/config/actions, не постоянно установленный
plugin. После применения модули и active config живут самостоятельно.

- generic uninstall/rollback recipe отсутствует;
- удаление package не откатывает config/modules;
- applied-path state Project Browser — журнал удобства, не transaction ledger;
- idempotency зависит от action: `grantPermissions`/`createIfNotExists`
  устойчивы, произвольный import/update может конфликтовать;
- dependencies фиксируются в recipe и resulting config dependencies;
- update YAML recipe не применяет изменения автоматически к уже работающему
  сайту;
- active config не сохраняет универсального ownership recipe.

Поэтому повторное применение допустимо только после review конкретных actions и
config diff, а не как универсальный update mechanism.

### 17.4 `epe_master_base`

После утверждения модели recipe имеет смысл для bootstrap чистой среды:

- languages;
- content types/fields/taxonomies;
- workflow;
- Views/Search config;
- Webforms;
- SEO defaults;
- initial roles/permissions.

Граница:

- Recipe описывает начальную композицию и безопасные create/grant actions;
- обычный config management остаётся authoritative для изменений существующего
  сайта и deployment между environments;
- migrations/update hooks нужны только при будущих data/schema transforms,
  которых config actions не выражают.

Production recipe сейчас не создавался. Решение следует оформить ADR после
утверждения content/role/multilingual архитектуры.

## 18. ECA

ECA 3.1.5 и UI включены. Активны Content, Form, Language, Log, Queue, Views,
Webform, Metatag и другие integrations; ECA Workflow submodule выключен.
Baseline содержит 13 config models, включая sitemap settings, SEO fields,
Media permissions, unpublished 404 и Canvas behavior.

Оценка сценариев без production model:

| Сценарий | Естественный механизм | Вывод |
| --- | --- | --- |
| Draft → Review notification | Content Moderation Notifications уже установлен; ECA альтернативно | сначала профильный module |
| Publication trigger | ECA Content event + conditions/actions/queue | возможно без hook |
| Webform Correction notify | Webform email handler проще; ECA Webform для сложных условий | custom не нужен |
| Outdated translation reminder | ECA Content/Language + cron/queue/View | вероятно возможно, нужен prototype |
| AI task after event | AI Integration ECA существует, но provider и submodules не готовы | отдельное согласование |

ECA models хранятся в config и deploy обычным config management. ECA способен
заменить значительную часть будущих glue hooks, но model complexity, recursion,
retries и audit должны тестироваться. `eca_endpoint` не следует использовать
для создания собственного API, пока JSON:API/Views достаточны.

## 19. Identity и audit

Требуемая модель:

- отдельный Drupal user `editorial_ai_agent`, не human owner и не UID 1;
- отдельная минимальная role;
- отдельный confidential Consumer и Client Credentials;
- short-lived token;
- revision author = machine user;
- обязательный revision log с job/correlation ID без prompt/PII;
- Admin Audit Trail Node/Media/Taxonomy/Workflow уже enabled;
- OAuth/token/application logs с контролируемой retention;
- human reviewer/publisher остаётся отдельным revision author.

JSON:API response имеет `revision_uid` relationship, но user resource был
disabled и data вернулась null. Audit остаётся в entity/revision DB; exposing
user API для этого не требуется.

AI actions никогда не должны выполняться под аккаунтом владельца или shared
Content editor: иначе revision/audit не отличит человека от машины.

## 20. Failure и rollback

| Failure | Штатная реакция |
| --- | --- |
| Validation/access PATCH error | 4xx, entity не сохраняется; исправить request, не обходить access |
| Wrong AI edit | оставить/reject forward Draft; human не публикует; создать новую revision/revert |
| Partial multi-step process | фиксировать job state; выполнять Media до article PATCH; cleanup orphan отдельно |
| Failed Media upload | article relationship не менять; retry/idempotency по checksum/UUID |
| Token leak/revocation | revoke tokens, disable consumer, rotate secret, short TTL |
| Concurrent edit | заново GET default/working copy, не перетирать слепо; отдельный conflict policy |
| Failed Review transition | Draft остаётся Draft; alert/retry после permission/validation review |
| Published error | человек откатывает на предыдущую revision; agent не получает publish |

JSON:API не является cross-entity transaction system. Собственную transaction
платформу проектировать не нужно: entity validation + revisions + idempotent
job orchestration достаточны для editorial workflow.

## 21. Функции, запрещённые агенту

- Publish/Archive/Delete;
- users, roles, permissions, consumers, scopes и secrets;
- modules, recipes, config import и updates;
- content types/fields/workflows;
- Canvas templates/patterns/global regions;
- taxonomy term creation по умолчанию;
- delete/edit any Media;
- Webform submissions и PII;
- redirects/menu/global SEO settings;
- provider key management;
- запуск широких Canvas/content-type/field AI tools;
- обход access/revision/moderation checks.

## 22. Классификация

| Требование | Классификация | Вывод |
| --- | --- | --- |
| Closed-by-default entity API | Varbase штатно | `/api`, default disabled, read-only |
| Published entity read | Drupal core | JSON:API + Entity access |
| UUID/fields/relationships | Drupal core | подтверждено |
| Resource allowlist/aliases | Входит contrib | JSON:API Extras/Defaults |
| OAuth machine identity | Входит contrib | Consumers + Simple OAuth |
| Generated API schema | Входит contrib | полезна, но неполна как agent contract |
| Draft revision via PATCH | Drupal core | после отдельного решения об отключении global read-only |
| Draft → Review | Drupal core | moderation field + transition permission |
| Additional translation POST | Лучше скорректировать требование | human/internal process создаёт translation, agent PATCHит |
| Per-node agent allowlist | Возможный custom | сначала adjusted ownership или contrib access control |
| Ranked retrieval | Входит contrib | Search API + Views REST Export |
| Media/file upload | Drupal core | resource/permission config, не отдельный service |
| Existing taxonomy only | Drupal core | permissions + read-only terms |
| PII-free boundary | Varbase штатно | Webform resources остаются disabled |
| Internal editor AI | Varbase штатно | provider/data policy ещё не настроена |
| External draft-only agent | Входит contrib | OAuth + JSON:API + Search/View |
| Event automation | Входит contrib | ECA/config models |
| Reproducible bootstrap | Drupal core | Recipes + Composer |
| Recipe rollback/update ownership | Лучше скорректировать требование | config management остаётся authoritative |

## 23. Рекомендуемая минимальная архитектура

1. Dedicated machine user + narrow role.
2. Confidential Simple OAuth consumer, Client Credentials, 300-second token.
3. Explicit JSON:API resource allowlist: Article + минимальные Media/File/terms.
4. Search API View REST Export для retrieval; JSON:API для entity fetch.
5. PATCH создаёт forward Draft; отдельный PATCH переводит только в Review.
6. Human reviewer проверяет diff, language, Media, SEO и taxonomy.
7. Publish доступен только человеку.
8. Revision author/log + Admin Audit Trail различают AI и человека.
9. Revisions являются rollback layer; orchestration retries идемпотентны.
10. Webform/Canvas/config/users/taxonomy creation остаются вне agent boundary.

Собственный API на этом этапе не требуется. Реальные ограничения находятся в
translation creation, per-node ownership и полноте OpenAPI, а не в базовом
entity CRUD.

## 24. Рекомендуемые ADR, но не созданные

1. Editorial AI boundary: internal assistive AI + external draft-only agent.
2. Machine identity, OAuth Client Credentials и human-only publish.
3. Recipe versus config-management ownership для `epe_master_base`.

Каждый ADR существенно влияет на security/operations и требует отдельного
согласования пользователя.

Исследование №7 завершено. Исследование №8 автоматически не начинается.
