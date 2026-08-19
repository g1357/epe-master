# Исследование 03: мультиязычность RU/EN в Varbase 11

- Дата: 2026-08-19
- Baseline: Varbase 11.0.0-rc1, Drupal 11.4.5, PHP 8.4.22, full installation
- Статус: третий практический проход завершён
- Scope: languages, translations, fields, taxonomy, Media, Canvas, Views,
  navigation, URLs, SEO, search и API readiness

## Ограничения исследования

Использовались только текущая установка и уже включённые модули. Не создавались
custom-модули, custom-тема, custom-компоненты или иной custom-код; новые contrib
пакеты не устанавливались. Все добавленные переводы и термины являются
временными исследовательскими прототипами. Production-языковая политика и
архитектурно значимые URL-настройки не применялись.

## Краткий вывод

Правило E+E Master «материал может быть опубликован только на одном языке»
естественно поддерживается Drupal core и существующим Varbase workflow. Node
имеет общий identity, но отдельные language rows, revision metadata и moderation
state. Практически получены две важные комбинации:

- Service 16: EN Published + RU Draft;
- Asset 17: EN Published + RU Draft;
- Technical Article 18: EN Draft + RU Draft.

Views с фильтром `status = Published` не вывел RU draft Asset и не создал дубль.
Metatag с `metatag_hreflang` вывел только реально существующие переводы. Simple
Sitemap включил только опубликованные EN URLs. Следовательно, обязательная пара
переводов платформе не нужна.

Главная проблема baseline — не ограничение Drupal, а незавершённая настройка
language negotiation. В `language.negotiation` записан prefix `ru`, но текущие
RU aliases и hreflang генерируются без `/ru`, а прямые `/ru/...` возвращают
404. Browser negotiation не включён, language switcher отсутствует, Canvas Page
translations не включены. Эти настройки нельзя считать production-ready.

Рекомендуемая естественная архитектура для E+E Master:

- русский — default language;
- русский живёт на `/`, английский на `/en`;
- browser-language redirect не использовать как основной механизм;
- перевод необязателен, прямой доступ к материалу на исходном языке разрешён;
- локализованные Views показывают только опубликованные записи текущего языка;
- текст/SEO переводятся, факты и identity/reference по умолчанию shared;
- Canvas использует общий component tree и переведённые component inputs;
- меню имеет общую структуру и переведённые labels;
- API остаётся закрытым по умолчанию до отдельного решения.

Требование обязательных `/ru/...` и `/en/...` лучше скорректировать: для
русскоязычного сайта проще и естественнее Drupal-модель `/` + `/en`. Она не
требует отдельного root redirect, создаёт один канонический default URL и
уменьшает число negotiation edge cases. Если бизнес всё же требует оба
префикса, это реализуемо конфигурацией core, но должно быть отдельным решением.

## Baseline и установленные механизмы

### Языки и negotiation

- default language: English;
- дополнительные языки: Russian;
- content translation включён для Research Technical Article, Service, Asset
  и четырёх исследовательских taxonomy vocabularies;
- `language_interface`: URL negotiation;
- `language_content`: наследует interface language;
- `language_url`: URL + fallback;
- browser language negotiation отсутствует;
- session parameter настроен как `language`, но query `?language=ru` не изменил
  фактический `<html lang="en">`;
- конфигурация prefix: EN `''`, RU `ru`, однако prefix не сработал практически.

Классификация language setup: **Drupal core**. Исправление текущей конфигурации
не требует custom-кода, но менять default language и URL policy следует только
после отдельного архитектурного решения и до ввода production-контента.

### Модули в текущей установке

- Drupal core: Language, Interface Translation, Configuration Translation,
  Content Translation, Views, Media, Taxonomy, Workflows, Content Moderation;
- contrib/Varbase baseline: Pathauto, Redirect, Metatag + hreflang/Open Graph,
  Schema Metatag modules, Simple XML Sitemap, Search API Database, Canvas,
  Canvas Override, JSON:API Extras/Defaults.

## Практические прототипы

### Content translations и moderation

| Entity | EN | RU | Результат |
| --- | --- | --- | --- |
| Technical Article 18 | Draft | Draft | Общий node identity, две translation rows |
| Service 16 | Published | Draft | Независимая публикация подтверждена |
| Asset 17 | Published | Draft | Независимая публикация подтверждена |
| Taxonomy term 28 | Published | Published | `Building systems` / `Инженерные системы` |

На translation forms Service и Asset показано штатное предупреждение «Fields
that apply to all languages are hidden». Общие поля действительно отсутствовали
на форме перевода. Каждый save создал новую обязательную revision и собственную
строку moderation state для `ru`; опубликованная EN default revision осталась
неизменной.

Varbase использует существующий workflow `varbase_editorial_workflow`:
Draft, In review, Published, Archived/Unpublished. Он применён ко всем трём
типам. Это **Drupal core + Varbase штатно**.

Практическая тонкость: после сохранения непубликованного перевода переход на его
canonical/latest route показал access denied, хотя EN-оригинал оставался
доступен. Это ожидаемое следствие независимой публикации, а не повреждение EN
материала. Редакторам нужны понятные ссылки на Translations/Revisions, а не
навигация через публичный canonical draft.

### Материал без перевода и перевод позже

Technical Article 15, Service 16 и Asset 17 существовали и публиковались только
на EN до создания RU translation. Добавление перевода позже не изменило UUID,
numeric ID, references или EN revision. Правило необязательного перевода
подтверждено — **Drupal core**.

## Рекомендуемая field translation policy

Временная конфигурация не считается окончательной. В ней некоторые base fields
(`status`, author, created/changed, moderation state) отмечены translatable,
поскольку так была выполнена исследовательская настройка. Для production это
надо нормализовать.

| Поле/группа | Рекомендация | Обоснование |
| --- | --- | --- |
| Title/name | Translatable | Пользовательский текст |
| Description/body/summary | Translatable | Языковой контент |
| SEO title/description | Translatable | Отдельные SERP snippets |
| Open Graph textual values | Translatable через Metatag tokens | Следуют SEO/title/description |
| Taxonomy references | Shared | Один концепт, переводится сам term |
| Entity references | Shared | Связь — факт, не перевод; исключение только при разных редакционных подборках |
| Media reference | Shared | Один asset для обоих языков по умолчанию |
| Image alt/title/caption | Translatable на Media при необходимости | Текстовая доступность и подписи |
| Numerical values, площадь | Shared | Один физический факт |
| Address components | Shared по умолчанию | Один адрес; локализовать только display label/страна/регион при подтверждённой потребности |
| Dates | Shared | Факт времени; формат локализует Drupal |
| Author | Shared | Один автор entity/identity |
| Business status | Shared taxonomy reference | Один статус объекта/услуги |
| Published/moderation state | Translation-aware | Нужна независимая публикация RU/EN |
| URL alias | Translatable | Отдельные понятные slug |
| Menu title | Translatable | UI label |
| Canvas component tree | Shared/synchronized | Symmetric translation model Canvas |
| Canvas component input values | Translatable | Текст/props различаются по языкам |
| Canvas Override tree | Shared/synchronized | Разный layout по языкам не считать нормой |

Классификация: преимущественно **Drupal core**; Canvas synchronization —
**Входит contrib / Varbase штатно**.

### Отклонения текущих прототипов

- Asset address, area, status и images сейчас non-translatable — это совпадает
  с рекомендуемой shared-политикой;
- Service related articles и category non-translatable — рекомендуемый shared
  baseline;
- Technical Article topics, technologies и featured image non-translatable —
  рекомендуемый baseline;
- SEO text fields translatable — правильно;
- Canvas tree в текущих node fields визуально отмечен translatable, а component
  input values synchronized/disabled; это требует привести к поддерживаемому
  Canvas symmetric mapping при утверждении модели, а не вручную менять flags.

## Taxonomy

Практически создан перевод term 28:

- EN `Building systems`;
- RU `Инженерные системы`;
- term ID и hierarchy общие;
- name/description переводятся;
- revision log явно помечает исследовательский перевод.

Рекомендация: vocabularies и hierarchy общие; name/description term —
translatable; references в nodes — shared. При отсутствии term translation
показывать исходное имя только в административном UX, а в публичной
локализованной выдаче либо исключать facet option, либо принять осознанный
fallback. Не дублировать RU/EN как отдельные terms: это разрушит Views filtering
и аналитику.

Классификация: **Drupal core**.

## Media

Media translation для baseline не включена: все Media entities имеют EN row.
При этом `media.image.field_media_image` технически translatable, включая alt,
но без включения translation для Media это не создаёт отдельные RU values.
Node media references shared.

Рекомендуемая политика:

1. Один Media/file использовать для RU/EN, если визуал одинаков.
2. Включать Media translation только если нужен различный alt, caption,
   description или language-specific creative.
3. Разные изображения по языкам делать разными Media entities либо
   translatable reference только при реальной маркетинговой необходимости.
4. Embedded Media в CKEditor остаётся частью translatable body; embed указывает
   на общий Media UUID.

Это **Drupal core + Varbase штатно**. Требовать отдельную копию каждого файла
для каждого языка — **Лучше скорректировать требование**.

## Canvas

### Canvas Page

В baseline Canvas Pages существуют только на EN, а
`language.content_settings.canvas_page.canvas_page` отсутствует. Следовательно,
Canvas Page translation технически поддержана установленным Canvas, но в
текущем сайте не включена и практически не готова.

### Symmetric translation model

Исходный код установленной версии Canvas содержит явные гарантии:

- Canvas Page `components` использует symmetrical translation;
- component tree синхронизируется между переводами;
- component input values могут различаться по языкам;
- symmetric translations поддерживаются также Content Templates и Page
  Regions.

Практическое следствие: RU и EN должны иметь одинаковую структуру/layout, а
текстовые props переводятся. Различный layout по языкам противоречит
естественной модели и увеличит governance cost. Если языковые версии требуют
разной структуры, лучше использовать разные Canvas Pages/кампании или
скорректировать требование, а не обходить synchronization custom-кодом.

### Canvas Content Template, regions, patterns и override

- Content Template остаётся общим шаблоном bundle/view mode;
- его component tree общий, config translation переводит inputs/labels;
- global Page Regions должны иметь общую структуру и переведённые props;
- patterns являются reusable composition, а не независимой языковой сущностью;
- Canvas Override для node не должен становиться способом делать разные RU/EN
  layouts: symmetric tree сохраняет структурную согласованность.

Классификация: **Входит contrib / Varbase штатно**. Разный layout RU/EN —
**Лучше скорректировать требование**. Оснований для custom-кода нет.

## Views

`research_asset_catalog` и `research_services_by_article` используют
`node_field_data`, `status = 1`, bundle/reference filters и имеют cache contexts
`languages:language_content` и `languages:language_interface`.

Практический Asset catalog после создания RU Draft:

- показал только опубликованный EN Asset;
- не показал RU draft;
- не создал дубль;
- `?language=ru` не переключил язык из-за текущей negotiation-конфигурации;
- exposed labels и taxonomy options остались EN.

В View нет явного фильтра `Content language = current language`. После
публикации нескольких переводов это может дать дубли/нежелательный fallback.
Production Views должны иметь current content language + published filter и
проверку distinct/query behavior. Contextual filters по shared references
работают независимо от языка; rendered result выбирает перевод текущего языка.

Titles, empty text и exposed labels — configuration strings и переводятся
Configuration Translation. Классификация: **Drupal core**; Better Exposed
Filters — **Входит contrib**.

## Menus, navigation и breadcrumbs

Текущий main/footer/secondary menu содержит только EN links. Custom menu link
translation не включён, language switcher block отсутствует. Easy Breadcrumb
установлен, но multilingual behavior на полноценных RU routes пока проверить
нельзя из-за negotiation baseline.

Рекомендуемая политика:

- одна общая структура меню и общие destinations;
- titles/descriptions menu links переводятся;
- отдельные RU/EN деревья создавать только при доказанной разной IA;
- active trail и breadcrumbs следуют текущему translated entity/alias;
- language switcher ведёт на существующий перевод текущей страницы;
- если перевода нет, показывать язык disabled/without link либо вести на
  language home с явным UX; не создавать ложный alternate URL.

Штатный core switcher может показывать fallback текущего route. Точный UX
«disabled unavailable language» может потребовать дополнительной конфигурации
или mature contrib; custom сейчас не обоснован.

Классификация общей политики: **Drupal core**. Идеально различающиеся деревья —
**Лучше скорректировать требование**.

## URLs, aliases и redirects

Практически для node 18 существуют два language-specific aliases:

- EN `/research-second-structured-article`;
- RU `/issledovanie-vtoraya-strukturirovannaya-statya`.

Pathauto выполнил transliteration русского title. Alias rows имеют разные
langcode, поэтому одинаковый slug допустим в разных языковых пространствах;
конфликт внутри одного языка должен предотвращаться/разрешаться suffix.
Redirect module включён и URL Redirects доступен на форме, но смена alias в этом
проходе намеренно не выполнялась, чтобы не создавать долговременную redirect
цепочку у прототипа. Автоматический redirect надо отдельно regression-test после
утверждения Pathauto patterns.

Рекомендуемая URL policy:

- RU default: `/slug`;
- EN: `/en/slug`;
- aliases переводятся и могут иметь разные slug;
- canonical всегда self-referencing для фактического перевода;
- старый alias получает 301 через Redirect;
- root `/` показывает русскую home без browser-dependent redirect.

Оба обязательных префикса `/ru` и `/en` технически доступны core, но для E+E
Master это **Лучше скорректировать требование** в пользу `/` + `/en`.

## SEO

Практический output опубликованного Technical Article 15:

- `<html lang="en">`;
- self canonical;
- `hreflang="en"`;
- translated token-based title/description;
- Open Graph URL/title/description/image/alt;
- для двуязычного draft node 18 Metatag сгенерировал EN и RU alternates;
- JSON-LD Schema.org не найден, хотя Schema Metatag modules включены;
- Simple Sitemap `/sitemap.xml` вернул 200 и только опубликованные EN URLs;
- draft RU translations в sitemap не попали.

Вывод: Metatag, hreflang, Open Graph и Sitemap baseline сильный, но Schema.org
mapping не настроен. Устанавливать новый модуль не нужно: сначала настроить уже
входящие Schema Metatag plugins для Article/Service/Asset. Untranslated material
должен иметь только собственный hreflang и canonical; не генерировать alternate
на отсутствующий перевод.

Классификация: **Входит contrib / Varbase штатно**. Schema mappings требуют
конфигурации, не custom-кода.

## Fallback policy

Рекомендуется разделить direct navigation и listings:

| Сценарий | Рекомендуемое поведение |
| --- | --- |
| Пользователь RU, материал только EN, прямой URL | Показать EN original с корректным `lang=en`; не создавать RU canonical |
| Пользователь EN, материал только RU, прямой URL | Показать RU original либо явную language notice |
| Локализованный View/list | Исключить материал без опубликованного перевода текущего языка |
| Language switcher, перевода нет | Disabled state или language home; не вести на несуществующий alternate |
| Неопубликованный перевод | Не показывать публично и не индексировать; original остаётся доступен |

Такой вариант поддерживает правило необязательного перевода, но не смешивает
языки в каталогах. Он проще 404/redirect matrix и не требует custom fallback.
Если редакционная стратегия требует показывать все материалы независимо от
языка, это можно изменить на fallback original в Views, но это должно быть
сознательной продуктовой политикой.

Классификация: **Drupal core + Лучше скорректировать требование**.

## Search readiness

В отличие от предварительной фиксации исследования 02, активный index уже есть:
`search_api.index.content`, server `database`. Он индексирует node и Canvas Page,
rendered HTML и title, все bundles и default set языков. Включён processor
`language_with_fallback` и Canvas component tree inputs.

Search API хранит language metadata у indexed items; отдельный index для RU и
EN технически не обязателен. Рекомендуемый baseline — один index, language
filter текущего интерфейса/content language, published access checks и
отсутствие fallback draft. Отдельные indexes нужны только при доказанных разных
analyzers/stemmers или backend constraints. Текущий generic stemmer не является
доказательством качественной русской морфологии — relevance остаётся отдельным
исследованием.

Классификация: **Входит contrib**. Новый search module не устанавливался.

## JSON:API и AI readiness

JSON:API module включён, но JSON:API Extras настроен security-first:

```yaml
path_prefix: api
default_disabled: true
```

Для Research bundles resource configs отсутствуют; `/jsonapi` и попытка
Research collection дали 404. API не открывался. Когда ресурс будет отдельно
разрешён, translation rows будут представлены как language-specific entity
representations с общим UUID/relationships и `langcode`; доступные переводы
агенту лучше определять через явно спроектированный relationship/index, а не
угадывать по title.

Для будущего AI-agent нужны:

- `langcode`, publication/moderation boundary и original language;
- стабильный UUID и shared relationships;
- список реально опубликованных переводов;
- language-aware Views/Search/API filtering;
- запрет draft/unpublished и least-privilege OAuth scope.

Классификация: **Drupal core + Входит contrib**. Открытие API — отдельное ADR;
custom-код сейчас не нужен.

## Редакционный UX

Практически редактор получает:

- Translations table с Original language, Not translated, status и Add/Edit;
- форму «Create Russian translation of ...»;
- предупреждение и скрытие shared fields;
- обязательные revisions и revision log;
- отдельный moderation transition;
- Translation panel с флагом outdated;
- content lock от одновременного редактирования.

Drupal может пометить другой перевод outdated, но автоматически определить
семантическую устарелость не способен. Процесс должен требовать: при существенном
изменении оригинала отметить переводы outdated и создать редакционную задачу.
Проблемы выбора dropdown Tagify, различия original/latest route и понимания
shared fields решаются обучением и role permissions, а не разработкой.

## Итоговые рекомендации

### Default language

Russian. Сменить до production-контента. Текущий EN default — исследовательский
baseline, не рекомендация.

### URL policy

RU `/`, EN `/en`; stable URL negotiation, browser negotiation не использовать
для обязательного redirect. Исправить текущую несогласованность prefix/aliases.

### Translation fallback

Direct original allowed; localized listings only current published language;
никаких fake translations, 404 для отсутствующего перевода не требуется.

### Field policy

Text/SEO translatable; factual scalar/reference/Media/author/date shared;
moderation and alias translation-aware.

### Taxonomy policy

Shared terms/hierarchy, translated name/description, shared node references.

### Media policy

Shared Media/file by default; translate Media metadata only when needed;
separate creative only for подтверждённый language-specific design.

### Canvas policy

Symmetric component tree, translated inputs; одинаковый layout RU/EN. Canvas
Page translation включать только после утверждения общей URL/field policy.

### Menu policy

One IA/tree, translated labels, entity links; missing translation UX описать в
редакционном руководстве.

### SEO policy

Self canonical, alternates только для опубликованных переводов, hreflang
through Metatag/Sitemap, translated SEO/OG, Schema mappings существующими
plugins, noindex/draft exclusion.

## Матрица требований

| Требование | Классификация | Вывод |
| --- | --- | --- |
| RU/EN languages и content translation | Drupal core | Подтверждено |
| Материал только на одном языке | Drupal core | Подтверждено на nodes, workflow, Views и sitemap |
| Независимые revisions/moderation | Drupal core + Varbase штатно | EN Published + RU Draft подтверждено |
| Field translation/shared policy | Drupal core | Требуется production audit, рекомендация дана |
| Translated taxonomy | Drupal core | RU term создан, identity/hierarchy shared |
| Shared Media | Drupal core + Varbase штатно | Подтверждено; Media translation пока выключена |
| Canvas symmetric translations | Входит contrib / Varbase штатно | Tree shared, inputs translatable; Canvas Page ещё не включён |
| Language-aware Views | Drupal core | Cache contexts есть; current-language filter надо добавить при утверждении |
| Common menu tree + translated labels | Drupal core | Возможность есть, baseline ещё EN-only |
| `/` + `/en` | Drupal core | Рекомендуемая естественная policy |
| Обязательные `/ru` + `/en` | Лучше скорректировать требование | Реализуемо, но сложнее root/fallback policy |
| Pathauto aliases/transliteration | Входит contrib | Два language aliases и RU transliteration подтверждены |
| Redirect old alias | Входит contrib | Module/UI есть; regression test отложен до утверждения patterns |
| Canonical/hreflang/OG | Входит contrib / Varbase штатно | Практически подтверждено |
| Schema.org output | Входит contrib | Plugins есть, mapping/output не настроен |
| Multilingual sitemap | Входит contrib | Published EN only; drafts excluded |
| Search language readiness | Входит contrib | Один активный index, all languages, language processor |
| JSON:API language readiness | Drupal core + входит contrib | API default disabled; exposure не менялось |
| Browser negotiation | Drupal core | Не включён; не рекомендуется как основной redirect |
| Разный RU/EN Canvas layout | Лучше скорректировать требование | Противоречит symmetric model |
| Custom code | Возможный custom | Оснований не найдено |

## Что следует скорректировать в требованиях E+E Master

1. Не требовать перевод для публикации оригинала.
2. Не требовать два префикса: для RU-primary сайта принять `/` + `/en`.
3. Не переводить factual fields и references только ради формальной симметрии.
4. Не дублировать taxonomy terms и Media files по языкам.
5. Не проектировать разные Canvas layouts RU/EN; переводить component inputs.
6. Не показывать fallback original в локализованных каталогах без явного
   продуктового решения.
7. Не открывать JSON:API и draft data «для AI readiness»; сначала определить
   минимальный read-only контракт.
8. Не обещать качественный RU search только по факту наличия Search API;
   морфологию/relevance исследовать отдельно.

## Нерешённые вопросы перед применением

- утвердить Russian default и `/` + `/en` либо оба prefixes;
- после решения настроить negotiation и повторить `/ru`/`/en` regression;
- включить и практически проверить Canvas Page translation;
- нормализовать base-field translatability прототипов;
- добавить current language filter в production Views и проверить published
  RU/EN rows без дублей;
- определить UX switcher при отсутствии перевода;
- настроить Schema.org mappings;
- проверить Redirect при реальной смене approved alias;
- провести отдельное исследование RU/EN search relevance.

Ни один пункт пока не доказывает необходимость custom-кода.

## Evidence

- UI: `/admin/config/regional/content-language`;
- UI: translation forms nodes 16, 17, 18 и term 28;
- UI: `/research/assets`;
- rendered head: canonical, hreflang, Metatag и Open Graph node 15/18;
- XML: `/sitemap.xml`;
- active config: `language.types`, `language.negotiation`,
  `varbase_editorial_workflow`, Research Views, Metatag, Simple Sitemap,
  Search API, JSON:API Extras;
- installed Canvas source: symmetric component tree synchronization schema and
  translation hooks.

