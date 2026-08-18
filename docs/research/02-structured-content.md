# Исследование 02: структурированный контент Varbase 11

- Дата: 2026-08-18
- Baseline: Varbase 11.0.0-rc1, Drupal 11.4.5, full installation
- Статус: второй практический проход завершён
- Scope: Content Types, Fields, Media, Taxonomy, Views и Canvas Content Templates

## Ограничения исследования

Все сущности и конфигурация ниже — временные локальные прототипы, а не модель
E+E Master. Не создавались custom-модули, custom-компоненты, custom-тема и
custom-код; тема не изменялась. Дополнительные пакеты не устанавливались.
Изменения выполнены через административный UI и остаются в активной локальной
конфигурации/БД. В Git фиксируется только документация исследования.

## Краткий вывод

Для повторяющихся и пригодных для поиска данных естественная архитектура
Varbase 11 — обычный Drupal Content Type с типизированными полями, Media,
Taxonomy и Views, а Canvas Content Template отвечает за единое компонентное
представление материалов. Canvas Page лучше оставить для уникальных,
композиционно свободных страниц, где нет каталога однотипных записей.

Все три сценария реализуемы без custom-кода на уровне прототипа:

- Technical Article — Content Type + CKEditor 5 + Taxonomy + Media + Content
  Template;
- Service — Content Type + entity reference на Technical Article;
- Asset — Content Type с числовыми/текстовыми/таксономическими полями + Media,
  а каталог и фильтрация строятся Views.

Главные нерешённые вопросы перед production-моделью: governance словарей,
окончательный набор полей, языковая политика каждого поля, API exposure,
поисковый индекс и допустимость индивидуальных Canvas overrides.

## Инвентаризация baseline

В full-установке уже включены необходимые механизмы:

- Drupal core: Node/Field UI, Media/Media Library, Taxonomy, Views/Views UI,
  Language/Content Translation, Workflows/Content Moderation, CKEditor 5,
  JSON:API;
- contrib в установке: Canvas и Canvas Override, Tagify, Better Exposed
  Filters, Search API + Database, Metatag/Schema/Pathauto/Simple Sitemap/Yoast,
  JSON:API Extras/Defaults, OpenAPI JSON:API, Scheduler и integration с
  moderation;
- Varbase: переиспользуемые поля контента и SEO, Media UX, готовый editorial
  workflow и Vartheme/Design System components.

Наличие модулей не принималось за доказательство: ниже перечислены фактические
эксперименты.

## Созданные прототипы

### Research Technical Article

Создан тип `research_technical_article` с revisions, выбором языка, content
translation и разрешённым per-content Canvas override.

Поля:

| Поле | Тип/источник | Назначение |
| --- | --- | --- |
| Title, author, created/changed, language | Drupal base fields | Заголовок, авторство, даты, язык |
| `field_description` | переиспользованное Varbase text field | Аннотация |
| `field_content` | переиспользованное Varbase formatted long text | Основной текст |
| `field_article_content` | временное long text with summary | Сравнение варианта со summary; дублирует модель и не рекомендуется оставлять |
| `field_featured_image` | Media reference | Главное изображение |
| `field_research_topics` | Taxonomy reference | Тематика |
| `field_research_technologies` | Taxonomy reference | Технологии |
| Canvas/SEO fields | автоматически добавлены Varbase | Layout override, title/description/image/analysis |

Созданы два материала. Для первого создана вторая draft-ревизия поверх
опубликованной версии. Для второго создан необязательный русский перевод в
состоянии Draft; английский и русский варианты имеют независимые поля
перевода, но общий node identity.

### Research Service

Создан тип `research_service` с revisions, переводами, moderation и Canvas
override.

Поля: title, `field_description`, `field_content`, `field_featured_image`,
`field_research_service_category`, `field_related_technical_articles`,
Canvas/SEO fields. Поле связи — многозначный entity reference только на
Research Technical Article.

Практическая особенность: штатный для этой установки contrib-виджет Tagify
принимает ссылку только при выборе результата из dropdown. Простое программное
заполнение видимого/исходного input создаёт визуальный tag, но не сохраняет
target ID. После выбора подсказки запись `Service 16 -> Article 15` сохранилась.
Это UX-требование к обучению редакторов, а не ограничение entity reference.

### Research Asset

Создан тип `research_asset` с revisions, переводами, moderation и Canvas
override.

Поля: title, formatted description, decimal area (scale 2, minimum 0, suffix
`m²`), address, taxonomy status, multiple Image Media reference, Canvas/SEO.
Типизированных полей достаточно для каталога без custom-кода. Адрес пока
обычная строка: переход к Address/Geofield/геокодированию нельзя делать до
подтверждения требований к картам, компонентам адреса и поиску по расстоянию.

## Taxonomy

Созданы revisionable/translatable словари:

- Research Topics;
- Research Technologies;
- Research Service Categories;
- Research Asset Statuses.

Добавлены тестовые термины, включая Available/Occupied. Taxonomy естественна
для управляемых, переиспользуемых классификаторов и фильтров. Не следует делать
словарём уникальный адрес, площадь или свободное описание. Для небольшого
закрытого enum (например, неизменяемые машинные статусы) Options field может
оказаться проще; решение зависит от необходимости переводов, иерархии и
редакторского управления терминами.

## Media и Media Library

Создан Media Image `Research E+E placeholder image` из существующего
репозиторного `logo.png`, заполнен alt text, сохранена focal point 50/50. Media
Library показала существующие demo media, позволила найти и выбрать новый Media
item в node field. Та же сущность переиспользована несколькими материалами.

Практически подтверждены:

- отдельная Media entity вместо копирования файлов — **Drupal core**;
- Media Library widget и CKEditor embed — **Drupal core**;
- focal point, bulk/upload/edit и расширенный UX — **Входит contrib / Varbase
  штатно**;
- Varbase Media types Image, Document, Video, Remote Video, Audio, SVG Image —
  **Varbase штатно**.

Загрузка малого локального изображения заняла около 196 секунд. Это не принято
за норму и требует отдельного performance/media pipeline исследования
(ImageMagick, responsive derivatives, Drimage, файловая система DDEV).

## Views: практические результаты

### Asset catalog

Создан View `research_asset_catalog`, page `/research/assets`:

- базовый Content filter ограничивает список Research Asset;
- отображается созданный Asset;
- exposed taxonomy select `Asset status` содержит Any/Available/Occupied;
- exposed sort `Area` предоставляет Asc/Desc;
- штатные pager, cache и access settings доступны.

Вывод: базовый каталог недвижимости с фильтром по статусу и сортировкой по
площади удобен без custom-кода — **Drupal core**, с доступными улучшениями
Better Exposed Filters — **Входит contrib**. Для production надо проверить
несколько десятков/сотен записей, множественные фильтры, диапазон площади,
локализованные URL и UX мобильной формы. Карта, георадиус и зависимые фильтры
могут потребовать дополнительный mature contrib, но сейчас не устанавливались.

### Service -> Technical Article и обратный список

Создана прямая многозначная entity reference и View
`research_services_by_article`, page `/research/services-by-article`.
Exposed filter по target ID `15` вернул связанный Service. Таким образом,
обратное отображение возможно штатными Views на основе reference field.

Для итогового UX числовой ID в exposed form непригоден. Естественный вариант —
скрытый contextual filter от текущего Article и block/display, либо
autocomplete/select с уже имеющимися Views/Better Exposed Filters механизмами.
Это конфигурационная задача, не основание для custom-кода.

## Canvas Content Templates

Для full view mode Research Technical Article через Canvas UI опубликован общий
Content Template:

- Heading prop связан с node `Article title`;
- Rich Text prop связан с `Content` (`field_content`).

Второй Article без отдельной разметки автоматически получил тот же template и
вывел собственный title. Это подтверждает модель «один шаблон — много
материалов». Canvas документирует content template как дерево компонентов,
привязанное к entity type, bundle и view mode, с props из полей сущности.

Маршрут `/node/18/canvas` открыл отдельный Canvas editor для материала, а рядом
доступны Canvas Override и Reset Canvas layout. Следовательно, индивидуальная
композиция возможна, не меняя общий шаблон и другие nodes — **Входит contrib
(Canvas Override) / Varbase штатно**. Однако массовые overrides разрушают
предсказуемость общего дизайна, поиска и поддержки. Рекомендация для E+E
Master: разрешать override только ограниченной роли и только как исключение;
обычным редакторам — поля + общий template.

## Revisions, moderation и multilingual

Все типы revisionable. На момент проверки:

| Node | Сущность | Revisions | Состояние |
| --- | --- | ---: | --- |
| 15 | Technical Article | 2 | default Published, latest Draft |
| 16 | Service | 4 | Published |
| 17 | Asset | 1 | опубликован до первой moderated revision |
| 18 | второй Technical Article | 2 | EN Draft + RU Draft |

К трём типам подключён существующий `varbase_editorial_workflow` со состояниями
Draft, In review, Published, Archived/Unpublished. Новый workflow не создавался.
Форма требует новую revision и хранит revision log; content lock защищает от
одновременного редактирования.

RU/EN readiness подтверждена созданием RU translation. Перевод необязателен:
английский материал существовал до RU translation, а публикация управляется по
revision/moderation. Перед production нужно провести field-by-field audit:
часть временных полей была создана non-translatable (например, Service reference
в активной конфигурации), поэтому нельзя считать все поля правильно
локализованными. Общие идентификаторы, Media и числовые значения часто разумно
оставлять shared; title/body/SEO — переводить.

## SEO

Varbase автоматически добавил к новым типам SEO title, description, image и
Yoast analysis; также доступны Pathauto, Redirect, Metatag/Schema,
hreflang-модуль и Simple Sitemap per-entity settings. Это сильный штатный
baseline, но не готовая SEO-стратегия. До production требуются patterns,
canonical/hreflang правила, Schema.org mapping каждого типа, sitemap inclusion
и политика индексации draft/translation.

Классификация: **Varbase штатно / Входит contrib**.

## CKEditor 5 для Technical Article

В активном Rich editor практически обнаружены toolbar/actions:

| Требование | Результат | Классификация |
| --- | --- | --- |
| Длинный структурированный текст | Formatted long text + CKEditor 5 | Drupal core |
| Заголовки | Heading dropdown | Drupal core |
| Таблицы | Insert table | Drupal core |
| Изображения и embedded Media | Media Library/embed и image tooling | Drupal core + входит contrib |
| Ссылки/anchor | Link и Anchor | Drupal core + входит contrib |
| Цитаты | Block quote | Drupal core |
| Программный код | inline Code и Insert code block | Drupal core/входит contrib |
| Syntax highlighting | подсветчик Prism/Highlight не найден; code block не равен highlighting | Требуется дополнительный contrib либо лучше скорректировать требование |
| Автоматическое оглавление | штатного TOC в toolbar/enabled packages не найдено | Требуется дополнительный contrib либо лучше скорректировать требование |

Также присутствуют source editing, show blocks, fullscreen, find/replace,
lists, alignment, bidi, special characters, emoji и WProofreader. Дополнительные
модули для syntax highlighting/TOC не устанавливались. Сначала следует решить,
действительно ли нужны автоматическое TOC и цветная подсветка, или достаточно
ручного списка anchors и семантического code block.

## Search, JSON:API и AI readiness

Структурированные taxonomy/reference/decimal/text поля доступны Views и могут
быть добавлены в Search API index — **Drupal core + Входит contrib**. Модули
Search API и Database включены, но активного `search_api.index.*` не найдено.
То есть модель пригодна к поиску, а сам индекс ещё не настроен. Это правильно
оставить отдельному исследованию поиска.

Core JSON:API включён, вместе с JSON:API Extras/Defaults/OpenAPI. Но активная
настройка Varbase:

```yaml
path_prefix: api
default_disabled: true
```

Конфигураций `jsonapi_resource_config.*` для новых типов нет; `/jsonapi` и
попытки коллекций вернули 404. Следовательно, данные технически хорошо
моделированы для JSON:API, но **не опубликованы в API по умолчанию**. Это
security-first baseline JSON:API Extras и полезное ограничение поверхности API.
Разрешать ресурсы, поля, read-only access и authentication следует отдельным
архитектурным решением; в этом исследовании настройки не менялись.

Для будущего AI-агента предпочтительны именно типизированные entities:
стабильные IDs, language/revision/moderation metadata, taxonomy и explicit
relationships лучше свободной Canvas-разметки. Но «AI-ready» не означает
«автоматически доступно AI»: нужны API permissions, schema/versioning,
publication filtering, provenance и защита персональных/неопубликованных данных.

## Когда применять механизмы Varbase

| Механизм | Естественное применение | Не следует использовать как |
| --- | --- | --- |
| Canvas Page | уникальная landing/home/campaign page с композиционной свободой | каталог повторяющихся объектов или хранилище структурированных характеристик |
| Content Type + Canvas Content Template | много однотипных сущностей, общий display, поиск, Views, workflow, API | способ вручную собирать каждую запись как независимую страницу |
| Taxonomy | управляемая общая классификация, иерархия, фильтры, переводы терминов | место для уникальных чисел, адресов и длинного текста |
| Media | переиспользуемый файл/изображение/video с metadata и lifecycle | повторная загрузка одного файла в каждое поле |
| Views | списки, каталоги, сортировка, exposed/contextual filters, related content | полноценная поисковая система relevance или уникальный page layout |

### Проверка исходных сущностей

- Technical Article — однозначно Content Type: повторяемость, revisions,
  moderation, taxonomy, search и API важнее свободной композиции.
- Asset — однозначно Content Type: площадь/статус/адрес должны быть полями для
  каталога и фильтрации. Canvas Page здесь был бы архитектурной ошибкой.
- Service — Content Type оправдан, если услуг несколько, они связаны с
  материалами и участвуют в списках/поиске. Если подтвердится одна-две полностью
  уникальные маркетинговые страницы без каталога и жизненного цикла, можно
  скорректировать требование и сделать Canvas Pages. Пока окончательное решение
  не принято.

## Матрица требований

| Требование | Классификация | Вывод |
| --- | --- | --- |
| Типы, поля, revisions, entity references | Drupal core | Подтверждено |
| Общие Varbase content/SEO fields | Varbase штатно | Переиспользовать вместо дубликатов |
| Media entities/Library/embed | Drupal core | Подтверждено |
| Focal point/bulk/extended media UX | Входит contrib | Подтверждено частично |
| Taxonomy vocabularies/terms/references | Drupal core | Подтверждено |
| Asset list/filter/sort | Drupal core | Подтверждено |
| Улучшенный exposed-filter UX | Входит contrib | Better Exposed Filters установлен; расширенный UX ещё не проверен |
| Service -> Article и reverse listing | Drupal core + входит contrib | Reference сохранён, обратный View работает; Tagify требует выбор dropdown |
| Canvas Content Template | Входит contrib / Varbase штатно | Один шаблон применился к двум nodes |
| Individual Canvas Override | Входит contrib / Varbase штатно | Доступен, нужен governance |
| Editorial workflow | Drupal core + Varbase штатно | Существующий workflow применён |
| RU/EN без обязательного перевода | Drupal core | Подтверждено; нужен field audit |
| SEO baseline | Varbase штатно / входит contrib | Поля и инструменты есть, strategy не настроена |
| Structured search | Входит contrib | Search API установлен, индекс отсутствует |
| JSON:API | Drupal core + входит contrib | Модули есть, ресурсы intentional disabled |
| Syntax highlighting | Требуется дополнительный contrib / лучше скорректировать требование | Не устанавливать до подтверждения |
| Automatic TOC | Требуется дополнительный contrib / лучше скорректировать требование | Не устанавливать до подтверждения |
| Address/geospatial catalog | Требуется дополнительный contrib либо лучше скорректировать требование | Не исследовано, требований пока недостаточно |
| Custom code | Возможный custom | Оснований в этом проходе не найдено |

## Как скорректировать требования E+E Master

1. Считать Content Type источником данных, а Canvas Content Template — общим
   display layer; не хранить значимые характеристики только внутри Canvas props.
2. Запретить свободные per-node overrides по умолчанию и дать их ограниченной
   роли для редких исключений.
3. Переиспользовать Varbase `field_description`, `field_content`, Media и SEO
   fields; удалить дублирующий exploratory `field_article_content` при
   утверждении модели.
4. Использовать Taxonomy только для действительно общих управляемых справочников.
5. Отложить карты/geocoding, TOC и syntax highlighting до подтверждения
   пользовательской ценности.
6. Определить Service как Content Type только после подтверждения повторяемости;
   уникальную маркетинговую услугу допустимо представить Canvas Page.
7. Не включать новые JSON:API resources «на всякий случай»; спроектировать
   минимальный read-only контракт для сайта/AI отдельно.

## Ограничения и последующие вопросы

- Автоматически добавленные/созданные поля требуют form/display и translation
  audit; Service category оказался hidden в active form display.
- Asset images на форме показали «one media item remaining» вопреки плану
  unlimited — проверить storage cardinality и widget configuration перед
  моделью.
- Asset, созданный до подключения workflow, ещё не имеет moderated revision;
  нужен lifecycle regression test после утверждения workflow.
- Canvas override multilingual semantics требуют отдельного теста: документация
  Canvas описывает symmetric translations для общего component tree.
- Production search relevance, facets, multilingual indexing и Solr не входят
  в этот проход.
- API exposure, permissions и OAuth — отдельное архитектурное решение.

Ни одно из этих ограничений пока не доказывает необходимость custom-кода.

## Первичные источники

- [Drupal CMS: content modeling](https://project.pages.drupalcode.org/drupal_cms/manage/content/drupal-content/)
- [Drupal User Guide: parts of a View](https://www.drupal.org/docs/user_guide/en/views-parts.html)
- [Drupal core Media Library overview](https://www.drupal.org/docs/core-modules-and-themes/core-modules/media-library-module/overview)
- [Embedding Media with CKEditor 5](https://www.drupal.org/docs/core-modules-and-themes/core-modules/media-library-module/embedding-media-with-ckeditor-5)
- [Drupal core JSON:API module](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module)
- [JSON:API security considerations](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/security-considerations)
- [Drupal Canvas content templates](https://project.pages.drupalcode.org/canvas/code-components/workbench/content-templates/)
- [Drupal Canvas translations](https://project.pages.drupalcode.org/canvas/guides/translations/)

