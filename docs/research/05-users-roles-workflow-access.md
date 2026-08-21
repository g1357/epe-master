# Исследование №5: пользователи, роли, workflow и access control

- Статус: практический проход завершён
- Дата: 2026-08-21
- Baseline: Varbase 11.0.0-rc1 / Drupal 11.4.5 / PHP 8.4.22
- Среда: локальный DDEV, full-вариант Varbase
- Ограничения: без custom-кода, новой темы и новых contrib-модулей

## 1. Цель и границы

Исследование проверяет, насколько штатная модель Varbase 11 подходит для
редакционной и пользовательской архитектуры E+E Master. Созданные пользователи,
роли и материалы являются временными прототипами. Конфигурация production-ролей
в этом проходе намеренно не менялась.

Важное различие терминов:

- **route/admin permissions** открывают административные страницы и операции;
- **content permissions** разрешают create/edit/delete для bundle;
- **moderation permissions** разрешают переходы workflow, но сами по себе не
  создают закрытый раздел сайта;
- **node access/grants** ограничивают view/update/delete конкретных опубликованных
  nodes;
- **API access** требует отдельного permission boundary и не должен наследовать
  возможности интерактивного администратора.

Официальная документация Drupal описывает роли как набор permissions, а опасные
permissions — как доступ только для trusted roles: [Roles and Permissions](https://www.drupal.org/docs/roles-and-permissions).

## 2. Методика

Проверены:

1. фактическая конфигурация ролей и permissions через Drush;
2. административные маршруты под разными штатными ролями через Masquerade;
3. полный редакционный цикл временного материала;
4. публичная выдача старой опубликованной ревизии при наличии новой Draft;
5. архивирование, прямой URL и Search API;
6. RU/EN moderation rows и revision history;
7. Views access settings, `node_access`, Media, Taxonomy и Canvas permissions;
8. user registration, anti-abuse, audit и security baseline;
9. состав уже включённых модулей и их конфигурация.

## 3. Baseline roles Varbase

### 3.1 Инвентаризация

| Роль | Permissions | Основное назначение baseline | Существенные наблюдения |
| --- | ---: | --- | --- |
| Anonymous | 7 | Публичный просмотр | `access content`, `view media`; нет редакционных прав |
| Authenticated | 15 | Базовый вошедший пользователь | Профиль, собственный unpublished content, own files; не является уровнем закрытого node access |
| Content editor | 135 | Ежедневное создание контента | Может publish/archive, работать с Media, menu, aliases и Full HTML; не управляет users/config/content types/templates |
| Content admin | 161 | Расширенное управление контентом | Имеет все transitions, taxonomy administration, patterns и content locks; не управляет users/permissions/config |
| SEO admin | 106 | SEO и контентная оптимизация | Может менять и публиковать Blog/Page, Media, menu/redirect/aliases; это не read-only SEO-роль |
| Site admin | 214 | Операционное администрирование сайта | Users, content, Canvas governance; нет `administer permissions`, site configuration и content types |
| Super Admin (`administrator`) | специальная | Полное администрирование | `is_admin: true`, получает все permissions автоматически |

Все четыре штатные административные контентные роли — `content_editor`,
`content_admin`, `seo_admin`, `site_admin` — имеют transitions:

- Create New Draft;
- Send to review;
- Publish;
- Archive;
- Restore from archive.

Следовательно, baseline не разделяет автора и издателя. Это допустимо для
маленькой доверенной команды, но не обеспечивает обязательный принцип
«четырёх глаз».

### 3.2 Практический доступ к административным страницам

| Проверка | Content editor | SEO admin | Site admin |
| --- | --- | --- | --- |
| `/admin/people` | 403 | 403 | Доступ |
| `/admin/people/permissions` | 403 | 403 | 403 |
| `/admin/structure/types` | 403 | 403 | 403 |
| Site information | 403 | 403 | 403 |
| Создание Blog | Доступ | Доступ | Доступ |
| Publish/Archive | Доступ | Доступ | Доступ |

`Site admin` имеет `assign roles`, но текущий `roleassign.settings` не содержит
разрешённых к назначению ролей. В форме пользователя отображается секция
Assignable roles без флажков: роль может блокировать/активировать аккаунты, но
фактически не может назначать роли. Это безопасный baseline, хотя название
permission без проверки конфигурации вводит в заблуждение.

### 3.3 Новые content types

Исследовательские bundles из исследования №2 не были автоматически добавлены в
permissions штатных ролей. Content editor получил 403 на создание и
редактирование `research_technical_article`. Это правильное безопасное поведение,
но означает обязательный production-шаг после создания любого bundle:
явно проверить create/edit/delete/revision/moderation permissions каждой роли.

### 3.4 Насколько роли пригодны для E+E Master

- **Content editor** можно переиспользовать почти без изменений только в очень
  маленькой доверенной редакции. Права publish/archive, delete-any Media,
  меню/aliases и Full HTML шире минимально необходимых автору.
- **Content admin** естественно подходит владельцу контентной модели и
  редакционного процесса, но `administer patterns` и taxonomy требуют обучения.
- **SEO admin** подходит только доверенному SEO-редактору: роль может менять и
  публиковать контент, а не только метаданные.
- **Site admin** подходит операционному администратору/владельцу сайта, особенно
  для Canvas governance, но не заменяет Super Admin для permissions/configuration.
- **Super Admin** нужен крайне ограниченному числу доверенных технических лиц.

Классификация: **Varbase штатно**. Если разделение author/publisher станет
обязательным, сначала достаточно конфигурации штатных roles/transitions; custom
не требуется.

## 4. Principle of least privilege

### 4.1 Избыточные права baseline

Наиболее заметные широкие permissions:

- все контентные роли могут Publish и Archive;
- Content editor может `administer menu`, aliases/redirects, создавать terms,
  редактировать и удалять любые Media поддерживаемых типов;
- Content admin может администрировать taxonomy и patterns;
- SEO admin может редактировать body и менять состояние публикации;
- Site admin управляет users и ключевыми Canvas templates/global regions;
- все перечисленные контентные роли используют формат Full HTML.

Full HTML в этой установке всё же проходит через включённый `filter_html` с
allowlist элементов; PHP-фильтра нет, ACE code filter отключён. Риск ниже, чем у
нефильтрованного HTML, но право остаётся trusted: разрешены embeds, classes и
широкая редакторская разметка.

### 4.2 Рекомендация

Не «исправлять» baseline заранее. Для первой маленькой команды можно начать со
штатных ролей, документировать доверие и наблюдать реальные обязанности. Перед
production выполнить permission review и убрать только подтверждённо лишнее.

Классификация: **Лучше скорректировать требование** — не вводить сложную матрицу
ролей до появления реального разделения обязанностей.

## 5. Editorial workflow

### 5.1 Конфигурация

Активен `varbase_editorial_workflow`:

| State | Published | Default revision |
| --- | --- | --- |
| Draft | Нет | Нет |
| In review | Нет | Нет |
| Published | Да | Да |
| Archived | Нет | Да |

Transitions:

- Draft/Review/Published/Archived → Draft;
- Draft/Review → In review;
- Draft/Review/Published → Published;
- Published → Archived;
- Archived → Published.

Workflow применяется к Blog, Page и трём временным research types.

### 5.2 Полный практический цикл

Временный материал `Workflow Access Research Prototype` прошёл:

1. revision 29 — Draft;
2. revision 30 — In review;
3. revision 31 — Published;
4. revision 32 — новая Draft с отличимым маркером;
5. revision 33 — повторная публикация;
6. revision 34 — Archived.

Revision UI показал автора, время, состояние и revision log. Drupal core
поддерживает именно такую модель: опубликованная версия остаётся live, а новая
рабочая копия проходит проверку отдельно ([Content Moderation overview](https://www.drupal.org/docs/8/core/modules/content-moderation/overview)).

Практические результаты:

- при существовании revision 32 Draft анонимный URL возвращал прежний
  опубликованный текст revision 31;
- новый Draft-маркер не появился в публичном Search API;
- после публикации revision 33 новый текст стал текущим;
- после Archive прямой публичный URL возвращал 404;
- после Archive поиск не выдавал ни title, ни snippet материала.

Требование «предыдущая published revision остаётся публичной до публикации новой»
полностью подтверждено.

Классификация: **Drupal core + Varbase штатно**.

### 5.3 Content lock

Включён `content_lock` 3.0.0, для nodes активна блокировка с timeout 30 минут.
Модуль предназначен для предотвращения одновременного редактирования
([Content locking](https://www.drupal.org/project/content_lock)). В UI ранее
наблюдались lock controls. Полноценную гонку двух независимых браузерных сессий
в этом проходе не моделировали; её следует включить в acceptance test редакции.

Классификация: **Входит contrib**.

### 5.4 Scheduling

Scheduler и `scheduler_content_moderation_integration` 3.0.5 включены. Для Blog и
Page scheduling publish/unpublish активен, для временных research types — нет.
Интеграция с moderation присутствует, но это bundle-level configuration, а не
автоматическая возможность каждого нового типа.

Классификация: **Входит contrib**.

### 5.5 Preview и temporary unpublished access

Включён `access_unpublished` 1.9.0. Он выдаёт временную уникальную ссылку на
unpublished entity при наличии соответствующего permission; стандартный сценарий
описан на [странице проекта](https://www.drupal.org/project/access_unpublished).

Текущая конфигурация:

- query key: `auHash`;
- lifetime: 172800 секунд (48 часов);
- permissions для anonymous/authenticated выданы только Blog и Page;
- таблица tokens пуста;
- временные research bundles не получили эти permissions автоматически.

Это механизм proofread/preview, а не способ создать Member/Internal-раздел.
Перед production требуется отдельная проверка того, кто может генерировать токены,
их отзыва и cache behavior.

Классификация: **Входит contrib**.

## 6. RU/EN workflow

### 6.1 Что подтверждено

Для bilingual Service практически существует сочетание:

- EN original — Published;
- RU translation — Draft.

В `content_moderation_state_field_revision` хранятся отдельные строки по revision
и `langcode`. Для нескольких прототипов подтверждены независимые EN/RU revision
rows. Правило проекта «материал может быть опубликован на одном языке без второго
перевода» сохраняется и на уровне moderation.

### 6.2 Ограничение эксперимента

Полный RU цикл Draft → Review → Published → Archived не был безопасно завершён.
Из-за уже зафиксированной в исследовании №3 неопределённости language negotiation
RU edit links разрешались в EN form, а `/ru/...` возвращал 404. Маршрут добавления
перевода предлагал создать RU заново, несмотря на существующую RU revision.

Изменение default language/URL negotiation существенно влияет на архитектуру и
не было выполнено без решения пользователя. Существующие данные не перезаписывались.

### 6.3 Language-specific permissions

Штатные content permissions ориентированы на entity/bundle/operation, а не на
правило «пользователь может редактировать RU, но не EN». Естественного permission
baseline для языкового разделения редакторов не найдено.

Рекомендация: не разделять редакторов по языкам в первой версии; использовать
общие роли и редакционный процесс. Если это станет юридическим или организационным
требованием, сначала оценить зрелый contrib-механизм.

Классификация: независимая moderation переводов — **Drupal core**; editor-by-language
— **Требуется дополнительный contrib** либо **Лучше скорректировать требование**.

## 7. Public и Authenticated baseline

Anonymous имеет `access content` и `view media`. Authenticated добавляет управление
профилем, собственным unpublished content/files и несколько editor helpers.

Практически важно: core-role Authenticated не означает, что опубликованный node
доступен только вошедшим. В `node_access` присутствует только default row:

```text
nid=0, realm=all, gid=0, grant_view=1
```

Drupal node grants могут управлять view/update/delete конкретных nodes; default
row и принцип realms/grants описаны в [Node access API](https://www.drupal.org/docs/develop/drupal-apis/writing-a-module-that-handles-node-access).

В текущем baseline нет включённого механизма role-restricted published content:

- Groups отсутствует;
- Content Access отсутствует;
- Permissions by Term отсутствует;
- `eca_node_access` выключен;
- `taxonomy_access_fix` регулирует операции с terms, а не видимость nodes.

Поэтому безопасно смоделированы public, draft и archived states, но не создан
искусственный «authenticated-only published node».

Классификация: публичный baseline — **Drupal core**; опубликованный role-restricted
content — **Требуется дополнительный contrib**, только если бизнес-требование
сохранится. Custom node-access module сейчас не обоснован.

## 8. Будущие уровни доступа

Не рекомендуется хранить числовое поле `access_level = 1..4` и строить на нём
самодельный доступ. Естественнее описывать capabilities:

- Public — отсутствие требования к роли;
- Registered — core Authenticated плюс конкретные permissions;
- Member — будущая бизнес-роль с подтверждёнными capabilities;
- Internal — отдельная внутренняя роль;
- доступ к nodes — grants-механизм, если появится закрытый опубликованный контент.

Одних roles достаточно для routes/operations, но недостаточно для выборочной
видимости опубликованных nodes без grants provider. Решение о contrib-модуле нужно
отложить до конкретного сценария: какие материалы закрыты, кто их видит, нужны ли
переводы, Search API и cache variations.

Классификация: **Лучше скорректировать требование** — говорить о ролях и
capabilities, а не об абстрактных уровнях.

## 9. Restricted content, Search и Views

### 9.1 Search

Практически подтверждено:

- Draft не попал в anonymous search;
- новая Draft поверх Published не раскрыла новый title/body/snippet;
- прежняя Published revision оставалась доступной;
- Archived исчез из прямого URL и search results.

Search API View настроен с `bypass_access=false`, `skip_access=false`; cache
contexts включают `user.node_grants:view` и `user.permissions`. Это правильная
готовность к будущему grants provider, но не доказательство role-restricted модели,
которой пока нет.

### 9.2 Views

Исследовательские Views используют `status=1`, permission `access content`,
`disable_sql_rewrite=false` и те же grants/permissions cache contexts. Drupal
может применять query rewrite, когда реальный node access provider будет включён.

Для будущей модели обязателен regression suite:

1. direct canonical URL;
2. listing и related-content Views;
3. teaser, title и snippet;
4. Search API после полной reindex;
5. menu links и active trail;
6. breadcrumbs;
7. RU/EN translation каждого состояния;
8. anonymous/authenticated/member/internal;
9. warm и cold cache.

Нельзя считать доступ безопасным только по 403 на canonical URL.

Классификация: access-aware Views/Search — **Drupal core + входит contrib**;
role-restricted regression отложен до появления реальной модели.

## 10. Canvas permissions

| Операция | Content editor | Content admin | SEO admin | Site admin |
| --- | --- | --- | --- | --- |
| Создать/редактировать Canvas Page | Да | Да | Нет | Да |
| Удалить Canvas Page | Да | Нет | Нет | Да |
| Администрировать patterns | Нет | Да | Нет | Да |
| Content Templates | Нет | Нет | Нет | Да |
| Global regions | Нет | Нет | Нет | Да |
| Components/design system | Нет | Нет | Нет | Да |
| Canvas Override | Нет | Нет | Нет | Нет в baseline matrix |

Отдельные permissions для Canvas Override существуют, включая bundle-specific,
но штатным не-Super ролям не назначены. Предварительная идея подтверждена:
обычный редактор может работать с содержимым, но не должен менять shared Content
Templates, global regions и per-node layout override. Site admin естественно
является design-governance ролью.

Нюансы для будущего review: Content editor имеет delete Canvas Page, а Content
admin — administer patterns. Эти права шире или уже, чем можно ожидать по названию
ролей.

Классификация: **Varbase штатно**.

## 11. Media permissions

В baseline Content editor может:

- загружать и использовать все включённые Media types;
- редактировать own и any Media;
- удалять own и any Media.

Content admin и Site admin имеют ещё более широкое управление. SEO admin может
редактировать any Image/Remote video и own другие Media, то есть способен менять
SEO-значимые alt/caption/description.

Все 63 фактически управляемых файла используют `public://`; private files сейчас
не применяются. Следовательно, Media permission не следует путать с защитой файла
от прямого скачивания.

Рекомендуемая граница:

- editor: upload, reuse и edit own;
- content admin: edit any, управление metadata и замена;
- delete any: только content/site admin;
- protected downloads: отдельное исследование только при реальном требовании.

Классификация: **Drupal core + Varbase штатно**.

## 12. Taxonomy permissions

Content editor и SEO admin имеют generic `create any term`, edit Tags и reorder
Blog categories. Content admin и Site admin могут administer taxonomy и
update/delete any terms.

Generic permission распространяется и на новые vocabularies, поэтому новые
Topics/Technologies могут быстро превратиться в неуправляемый набор дублей.

Рекомендуемая политика:

- editors выбирают существующие Topics/Technologies;
- content admin или taxonomy steward создаёт, объединяет и удаляет terms;
- свободное создание тегов разрешать только для специально выбранного словаря;
- регулярно контролировать multilingual labels и hierarchy.

Это можно реализовать configuration-only. `taxonomy_access_fix` помогает
разграничить управление terms, но не является node access системой.

Классификация: **Drupal core + входит contrib**.

## 13. Registration, login и profiles

### 13.1 Текущий baseline

- self-registration: `admin_only`;
- email verification: выключена, что безопасно только при admin-created accounts;
- password reset lifetime: 24 часа;
- отмена аккаунта: block;
- login flood: 5 попыток/IP за 1800 секунд, 4/user за 1800 секунд;
- password policy: минимум 8 символов, upper/lower/numeric/special, username
  запрещён;
- profile fields: picture и технические password-policy поля; бизнес-профиля нет;
- Persistent Login включён;
- blocked users и password reset доступны штатно.

### 13.2 Anti-abuse

- CAPTCHA points настроены для login/reset/registration;
- FriendlyCaptcha включён как default challenge;
- Honeypot применяется к registration/password и некоторым формам, но не ко всем;
- Antibot охватывает registration/password/contact/comment/webform patterns;
- reCAPTCHA keys не настроены;
- Social Auth и provider-модули присутствуют, но рабочие provider credentials не
  обнаружены;
- MFA/TFA не установлены.

### 13.3 Рекомендация первой версии

Оставить registration disabled (`admin_only`). Когда появится реальный кабинет,
сначала выбрать approval flow и включить email verification, проверить abuse,
privacy и account lifecycle. Open registration сейчас создаёт поверхность атаки
без подтверждённой ценности.

Классификация: baseline — **Varbase штатно + входит contrib**; требование открыть
регистрацию сразу — **Лучше скорректировать требование**.

## 14. Comments/questions

Core Comment выключен, comment fields на baseline content types не используются.
Форума нет. Anti-spam modules уже содержат настройки для будущих comment forms,
но это не делает комментарии включённой функцией.

Рекомендация: оставить comments выключенными. После подтверждения сценария
«вопрос/исправление» сравнить moderated comments с Webform/contact workflow.

Классификация: Comments — **Drupal core**, сейчас не включено; spam protection —
**Входит contrib**.

## 15. Auditability

Доступны:

- core revisions с author/time/log;
- moderation states/history;
- Database Logging (`dblog`, лимит 1000 rows);
- Admin Audit Trail 1.0.10 с неограниченным текущим table limit;
- submodules для auth, file, media, menu, node, taxonomy, user и workflows.

Практический audit log зафиксировал login/logout/masquerade, создание и изменения
nodes, translations и workflow events. Revision history показал авторов всех
этапов исследовательского цикла. Страница проекта перечисляет поддерживаемые CUD
events и role assignment events: [Admin Audit Trail](https://www.drupal.org/project/admin_audit_trail).

Ограничения:

- в текущей выборке не было подтверждённого события изменения permission matrix;
- core user record хранит last login, а не полный immutable login ledger;
- database logs может изменить пользователь с доступом к БД;
- retention/backup/PII policy не настроена.

Для малого бизнеса baseline достаточен для операционного расследования, но не для
регулируемого/неизменяемого аудита. Дополнительный audit stack рассматривать
только при формальном compliance-требовании.

Классификация: **Drupal core + входит contrib**.

## 16. Security conclusions

1. User 1 — recovery/bootstrap account, не повседневная учётная запись. Drupal
   отдельно рекомендует не делиться UID 1 и максимально ограничивать его
   использование ([Administration guide](https://www.drupal.org/docs/administering-a-drupal-site/getting-started-with-drupal-administration)).
2. Super Admin role имеет `is_admin: true`, поэтому получает все новые permissions
   автоматически. Назначать её единицам.
3. Site admin не может менять permissions/config/content types, но управляет users
   и Canvas governance — это хороший слой между редакцией и Super Admin.
4. Publishing, delete any, user administration, role assignment, Full HTML,
   taxonomy/menu administration и Canvas templates/global regions — trusted
   permissions.
5. MFA отсутствует; перед production с административным доступом это отдельное
   security-решение, а не повод писать custom-код.
6. Private Media и role-restricted published content отсутствуют; публичный URI
   нельзя защитить только entity permissions.

Классификация: role separation — **Varbase штатно**; MFA и специальный protected
content могут потребовать **дополнительный contrib** после отдельного решения.

## 17. Future AI agent

Нельзя переиспользовать Content editor: эта роль умеет publish/archive, delete any
Media, менять menu/aliases и использовать широкий Full HTML.

Будущая техническая роль должна иметь только:

- чтение опубликованного разрешённого контента;
- API authentication через отдельный OAuth consumer/technical account;
- создание Draft только нужных bundles;
- редактирование собственных Draft;
- transition Draft → In review;
- upload/reuse только разрешённых Media types;
- при необходимости создание переводов тех же Draft.

Запретить:

- Publish, Archive и Restore;
- delete any;
- users, roles, permissions и configuration;
- taxonomy creation/administration;
- menus, aliases и redirects;
- Canvas templates, global regions, patterns и overrides;
- Full HTML и произвольные embeds;
- view any unpublished content.

API по принятому ранее правилу остаётся закрытым до отдельного ADR. AI-пользователь
в этом исследовании не создавался.

Классификация: техническая роль — **Drupal core configuration**; OAuth —
**Входит contrib**; custom не требуется до обнаружения конкретного ограничения.

## 18. Рекомендуемая минимальная модель E+E Master

### Первая версия

| Потребность | Рекомендация |
| --- | --- |
| Public visitor | Anonymous baseline |
| Зарегистрированный пользователь | Не вводить до появления кабинета; core Authenticated затем |
| Editor | Начать с Content editor для доверенной малой команды; провести точечный permission review перед production |
| Content owner | Content admin |
| SEO specialist | SEO admin только как trusted content editor, не как «только метаданные» |
| Site owner/operator | Site admin |
| Technical recovery/configuration | Очень ограниченный Super Admin + UID1 recovery |
| Member/Internal | Не создавать до подтверждения закрытого published-content сценария |
| AI agent | Не создавать до API ADR; позже отдельная least-privilege роль |

### Если обязательна проверка перед публикацией

Сначала сконфигурировать штатный Content editor без Publish/Archive и оставить
эти transitions Content admin. Это корректировка роли, а не новая workflow-система
и не custom-код.

### Если появляется закрытый раздел

Зафиксировать use cases и выбрать зрелый grants provider. Не имитировать доступ
через Draft, taxonomy field, скрытый URL или исключение из View.

Итог: Varbase baseline покрывает примерно 90–95% потребностей небольшой
редакционной команды. Естественный путь — адаптировать процесс под штатные роли и
workflow, а не разрабатывать собственную access/workflow систему.

## 19. Сводная классификация

| Требование | Классификация | Вывод |
| --- | --- | --- |
| Baseline editorial roles | Varbase штатно | Богатые, но доверенные роли |
| Draft/Review/Published/Archived | Drupal core + Varbase штатно | Практически подтверждено |
| Published + forward Draft | Drupal core | Старая revision остаётся live |
| Revisions и logs | Drupal core | Подтверждено |
| Content locking | Входит contrib | Включён, 30 минут |
| Scheduling + moderation | Входит contrib | Включено, требует bundle config |
| Temporary unpublished preview | Входит contrib | Access Unpublished, 48 часов |
| Независимые EN/RU states | Drupal core | EN Published + RU Draft подтверждено |
| Editor only RU, not EN | Лучше скорректировать требование | Иначе дополнительный contrib |
| Published authenticated/member/internal node | Требуется дополнительный contrib | Текущего grants provider нет |
| Search/Views respect grants | Drupal core + входит contrib | Настройки готовы; regression нужен после grants |
| Canvas governance | Varbase штатно | Site admin отделён от editor |
| Media permissions | Drupal core + Varbase штатно | Baseline editor слишком широк для строгого least privilege |
| Taxonomy governance | Drupal core + входит contrib | Ограничить term creation конфигурацией |
| Registration disabled | Drupal core | Рекомендовано для v1 |
| Anti-spam | Входит contrib | Baseline присутствует |
| Comments | Drupal core | Выключены; пока не нужны |
| Audit trail | Drupal core + входит contrib | Достаточно операционно, не compliance-grade |
| MFA | Требуется дополнительный contrib | Только после отдельного security-решения |
| AI least-privilege role | Drupal core configuration | Создавать позже вместе с API ADR |

## 20. Открытые решения и следующие проверки

Исследование №5 не запускает следующие этапы. Перед production понадобятся
отдельные решения:

1. default language и URL negotiation — блокирует безопасное завершение RU
   moderation regression;
2. нужен ли обязательный author/publisher split;
3. существует ли реальный Member/Internal published-content use case;
4. registration/approval/email verification при появлении кабинета;
5. MFA для административных ролей;
6. точечный permission review после утверждения окончательных bundles;
7. двухсессионный content-lock test;
8. полный grants/cache/search regression после выбора access-механизма.

Ни один из этих пунктов не требует custom-кода на текущем этапе.
