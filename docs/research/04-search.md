# Исследование 04: поиск в Varbase 11

- Дата: 2026-08-21
- Baseline: Varbase 11.0.0-rc1, Drupal 11.4.5, PHP 8.4.22, full installation
- Search API: 1.41.0
- Better Exposed Filters: 7.1.3
- Canvas: 1.10.1
- Статус: четвёртый практический проход завершён
- Scope: Search API Database, Views, Canvas, RU/EN, relevance, access,
  фильтры, UX, performance и путь к Solr

## Ограничения исследования

Использовалась только текущая установка. Не создавались custom-модули,
custom-тема, плагины или иной custom-код; новые contrib-модули и внешние
поисковые сервисы не устанавливались. Два добавленных материала являются
временным исследовательским корпусом, а не контентом E+E Master.

Production-конфигурация индекса и View намеренно не менялась. Это позволило
проверить реальный Varbase baseline, но означает, что потенциальные возможности
Search API нельзя считать уже настроенными.

## Краткий вывод

Текущий Varbase baseline уже даёт работающий site search без разработки:

- один Search API index для node и Canvas Page;
- Database backend;
- полнотекстовый поиск по rendered entity и title;
- title boost `3`;
- snippets и highlighting;
- Views page `/search` и header block;
- pagination, relevance sort и node access checks;
- прямую индексацию после сохранения и пакетную cron-индексацию.

Этого достаточно для прототипа и малого сайта с простым EN-поиском. Для
E+E Master baseline пока недостаточен как production-поиск: русский stemmer
фактически не работает, RU/EN выдача не разделена, структурированные поля не
добавлены в индекс, facets/autocomplete отсутствуют, а короткие технические
термины с пунктуацией дают ложную широкую выдачу.

Главный архитектурный вывод: не проектировать custom search. Сначала привести
один Search API index и одну Views-выдачу к согласованной языковой и field
policy. Затем провести приёмку на реальном корпусе. Если RU morphology,
facets, spellcheck, autocomplete или рост объёма останутся обязательными,
естественный следующий backend — Search API Solr, а не custom-код.

## 1. Фактический baseline

### Установленные и включённые механизмы

| Компонент | Состояние | Классификация |
| --- | --- | --- |
| Search API 1.41.0 | Установлен и включён | Входит contrib |
| Search API Database | Включён, server `database` | Входит contrib |
| Search API Exclude | Включён | Входит contrib |
| Views | Использует Search API index | Drupal core + входит contrib |
| Better Exposed Filters 7.1.3 | Включён, но search View использует обычную exposed form | Входит contrib |
| Canvas 1.10.1 | Datasource и processor component tree inputs активны | Varbase штатно / входит contrib |
| Facets | Не установлен | Требуется дополнительный contrib |
| Search API Autocomplete | Не установлен | Требуется дополнительный contrib |
| Search API Solr | Не установлен | Требуется дополнительный contrib + сервис Solr |

### Index `content`

Index включён и связан с server `database`. Datasources:

- `entity:node`: все bundles, все языки;
- `entity:canvas_page`: все bundles, все языки.

Индексируемые поля:

| Поле | Тип | Настройка |
| --- | --- | --- |
| Node title | Full text | boost `3` |
| Canvas Page title | Full text | boost `3` |
| Rendered item | Full text | node view mode `search_index`, Canvas default, anonymous role |

Структурированные поля bundle, language, created date, taxonomy, entity
references, Media metadata и Asset status/area сейчас не добавлены как
отдельные searchable/filterable fields. Часть их видимого текста может попасть
в `rendered_item`, но это не заменяет структурированную индексацию.

### Processors

Практически и через active config подтверждены:

- rendered item;
- entity status;
- node exclude;
- HTML filter с весами `h1=5`, `h2=3`, `h3=2`, `strong=2`;
- lower case;
- tokenizer: minimum word length `3`, partial matching;
- stemmer;
- language with fallback;
- highlighting/excerpt length `256` с `<strong>`;
- Canvas component tree inputs; props `class`, `cssClasses`, `extraClasses`
  и `id` исключены;
- reverse entity references, URL, entity type и aggregated/custom fields.

Database server использует:

- minimum word length `3`;
- partial matching;
- phrase matching через bigrams.

### Search View

View `search` предоставляет:

- page `/search`;
- header block;
- exposed full-text field `keywords`;
- сортировку только по relevance descending;
- full pager, 10 результатов на страницу;
- result count и empty text;
- title/link и Search API excerpt;
- permission `access content`;
- `bypass_access = false`, `skip_access = false`.

Baseline View не предоставляет:

- language filter;
- content type, date, taxonomy, difficulty или Asset status filters;
- facets;
- autocomplete/suggestions;
- пользовательский sort selector;
- promoted/featured editorial boost.

Классификация базового поиска: **Varbase штатно + входит contrib**.

## 2. Исследовательский корпус

К 28 существующим tracker items через штатную UI-форму добавлены:

| ID | Язык | Состояние | Назначение |
| --- | --- | --- | --- |
| Node 19, `Project Management Search Benchmark` | EN | Published | title/body relevance, English stemming, API/CI-CD/.NET/C# |
| Node 20, `Управление проектами — поисковый эталон` | RU | Published | русские словоформы, `ё/е`, технические токены |

Существующие prototype nodes и Canvas Pages использовались для проверки
taxonomy, Media/rendered metadata, code text, Canvas и access. Финальные
материалы E+E Master не создавались.

После сохранения Search API сообщил `30/30`, `100%`. Физические DB index tables
содержали только 25 доступных published entity-language items: tracker также
учитывает исключённые processors сущности, поэтому его `30/30` нельзя
интерпретировать как «30 документов доступны в выдаче».

## 3. Что индексируется

### Technical Article

Поиск `Technical` вернул шесть результатов. На первых позициях:

1. `Research: Structured Technical Article`;
2. `Research: Technical Consulting Service`.

Excerpt первой статьи включил title, автора/дату, description, основной текст
и строку из code content. Поиск `rankingalpha` вернул ровно node 19 и показал
маркер в snippet. Следовательно, rendered `Content` и Article content попадают
в индекс и searchable.

Текст taxonomy/Media попадает только если он реально выводится в view mode
`search_index`. Отдельных taxonomy/Media fields в index нет, поэтому:

- exact filtering и facet counts невозможны;
- скрытие поля из render mode может неожиданно удалить его из full text;
- Media metadata нельзя независимо boost/filter;
- связи нельзя надёжно использовать для AI/search API ответа.

Рекомендация: для production индексировать важные structured fields отдельно,
а rendered item оставить как общий catch-all. Это штатная конфигурация Search
API, не custom.

### Canvas

Физический индекс содержит published Canvas Pages 1–7. Поиск `469` вернул
Canvas `Home` и snippet из component tree: `469+ Sites using Varbase ...`.
Это подтверждает индексацию текстовых component inputs, а не только Canvas title.

В то же время presentation props намеренно исключены. Это правильно:
CSS classes и component IDs не должны загрязнять пользовательский поиск.

Ограничение: Search API видит то, что Canvas processor/rendered entity может
извлечь как индексируемое значение. Динамические значения, скрытые props,
client-only output или недоступный render context нельзя считать searchable
без отдельного теста компонента.

Классификация Canvas indexing: **Varbase штатно / входит contrib**.

## 4. RU/EN и морфология

### Language metadata

Один index хранит language-specific item IDs, например:

- `entity:node/19:en`;
- `entity:node/20:ru`.

DB field `search_api_language` содержит `en`/`ru`. Отдельные indexes по языкам
для самого факта разделения данных не нужны.

Но search View не фильтрует текущий язык. Запрос `API` одновременно вернул EN
и RU статьи. Для E+E Master production View должен фильтровать
`search_api_language = current content/interface language`, иначе локализованная
выдача смешивает языки.

Текущая проблема language negotiation из исследования 03 сохраняется:
`/ru/search?keywords=API` возвращает 404. Search View нельзя признать
multilingual-ready до решения URL/default-language policy.

### English stemming

English processor работает как Porter-like stemming:

- `project`, `projects` индексируются как `project`;
- `management` — как `manag`;
- `article` — как `articl`;
- запрос `projects` нашёл документы с разными формами и node 19 был первым.

Для EN baseline приемлем, хотя качество relevance нужно валидировать на
реальном доменном корпусе.

### Russian stemming

Практически русский stemming отсутствует:

- DB сохранила раздельные tokens `проект`, `проекты`, `проектами`,
  `проектный`, `проектом`;
- запрос `проектами` нашёл RU node 20;
- отсутствующая в тексте форма `проектов` дала zero results;
- `ёлка` и `елка` сохранены как разные tokens.

Наличие generic Stemmer processor не означает поддержки языка. Для E+E Master
нельзя обещать поиск по русской морфологии на Database backend.

Классификация качественной RU morphology: **Требуется дополнительный contrib**
вместе с language-aware backend/configuration; наиболее естественный кандидат —
Search API Solr. Custom stemmer не рассматривается.

### Материал только на одном языке

RU-only node 20 и EN-only node 19 индексируются независимо. Перевод не
обязателен. Это совместимо с принятой policy проекта.

Для локализованной выдачи рекомендуется:

- индексировать каждый published translation как отдельный item;
- filter по текущему языку;
- не показывать original fallback в списке автоматически;
- direct URL к original разрешать по policy исследования 03;
- language metadata хранить в будущих API/AI responses.

Классификация: **Входит contrib + Drupal core**.

## 5. Relevance и query behavior

### Подтверждённые сигналы

- Node/Canvas title boost: `3`;
- HTML heading weights в rendered item;
- term frequency и bigrams;
- relevance descending — единственный sort;
- exact marker `rankingalpha` дал один ожидаемый результат;
- `project management` поставил exact-title node 19 первым из 14.

То есть title и phrase proximity влияют ожидаемо. Однако Search API DB не даёт
удобного editor-facing relevance profile, explain UI или editorial boosts.

### Технические термины

| Запрос | Результат | Причина/риск |
| --- | --- | --- |
| `API` | 2 ожидаемых EN/RU результата | token длиной 3 сохранён |
| `.NET` | ожидаемые статьи + ложный `Contact Us` | punctuation отброшена, остаётся общий `net` |
| `C#` | 24 почти нерелевантных результата | короткий `C` отбрасывается; запрос фактически пуст/слишком широкий |
| `CI/CD` | 24 почти нерелевантных результата | части `CI`, `CD` короче minimum 3 |

В DB index не найден самостоятельный `c#` или `ci/cd`; `net` существует.
Это критично для технического раздела E+E Master.

Естественные варианты после утверждения требований:

1. сначала определить словарь реально важных сокращений;
2. проверить protected words, word delimiter и synonym policy в Solr;
3. не снижать global minimum word length без измерения index size/noise;
4. для названий технологий использовать отдельное indexed taxonomy/reference
   field и exact filter в дополнение к full text.

Пытаться исправить все сокращения custom query code — **Лучше скорректировать
требование** в пользу структурированного поля и backend analyzer configuration.

### Синонимы, опечатки, autocomplete

Baseline не имеет пользовательской synonym dictionary, spellcheck/did-you-mean
или autocomplete. Search API Database backend технически поддерживает extension
points, но нужные UI-модули не установлены.

Классификация:

- синонимы/опечатки production-grade — **Требуется дополнительный contrib**;
- autocomplete — **Требуется дополнительный contrib**;
- custom autocomplete — **Возможный custom**, но сейчас не обоснован.

## 6. UI, snippets и highlighting

Публичная View предоставляет понятное поле `Search by keyword`, кнопку Search,
result count, titles, links, excerpts, `<strong>` highlighting, pagination и
empty state.

Плюсы:

- URL query `?keywords=...` пригоден для sharing;
- snippet показывает найденный контекст;
- title link и count доступны без разработки;
- 10 results/page — разумный baseline.

Ограничения:

- нет filters/facets/sort control;
- labels и empty text пока EN;
- no-results не предлагает исправление/похожие запросы;
- snippets включают служебный rendered text: author, submission date;
- в excerpt возможны повторы title и summary;
- View field `search_api_excerpt` не включает отдельный
  `use_highlighting`, хотя processor уже формирует `<strong>` fragments;
- admin UI добавляет `Open ... configuration options`, но это не часть
  anonymous пользовательской выдачи.

Рекомендация: сначала настроить clean `search_index` view modes и перевести
View strings; не создавать отдельный frontend.

Классификация: **Varbase штатно + входит contrib**.

## 7. Filters, facets и catalog search

Better Exposed Filters установлен, но сам по себе не создаёт indexed fields и
facets. Он улучшает уже существующие exposed filters Views.

В baseline практически отсутствуют filters по:

- content type;
- language;
- date;
- taxonomy topic/technology/category;
- Asset status;
- difficulty — поле не создавалось, так как оно не подтверждено моделью.

Search API + Views позволяют добавить первые пять как обычные indexed fields и
exposed filters без custom-кода. Это следует проверить на production-like
corpus после утверждения field policy.

Faceted navigation требует Facets module, которого в установке нет. Он не был
установлен автоматически. Search API DB умеет backend facet operations, но без
Facets UI это не готовая функция сайта.

Для каталога Assets:

- малый каталог: Database backend + Views filters потенциально достаточен;
- многочисленные taxonomy/date/status facets: сначала Facets + performance test;
- крупный каталог, сложные OR facets и RU morphology: рассматривать Solr.

Классификация Views filters: **Входит contrib + Drupal core**.
Facets: **Требуется дополнительный contrib**.

## 8. Access, moderation и publication

Физические index tables содержали:

- published nodes 3–17, 19, 20;
- published Canvas Pages 1–7.

Они не содержали:

- node 18 EN Draft и RU Draft;
- Canvas Pages 8–9 Draft;
- nodes 1–2, отсутствующие из published index set.

Следовательно, Entity status/Node exclude processors исключили unpublished
entity-language variants до выдачи. Search View дополнительно имеет access
checks, а rendered item строится для anonymous role.

Важно: Search API сам предупреждает, что generic access restrictions не могут
быть обеспечены автоматически для всех entity types. Node access поддерживается,
но role-restricted/custom-grant content требует отдельного regression test с
реальными roles/grants. В исследовательской модели такого content access
механизма нет, поэтому симулировать его не стали.

Рекомендация:

- оставлять `skip_access`/`bypass_access` выключенными;
- индексировать только published translations для public index;
- private/partner content не смешивать с public index без threat model;
- тестировать anonymous, authenticated и каждую ограниченную роль;
- AI/search endpoint наследует те же access boundaries.

Классификация: **Входит contrib + Drupal core**. Custom access processor сейчас
не обоснован.

## 9. Performance и эксплуатация

### Измерения локального baseline

Корпус мал: 30 tracked items, 25 физически searchable published items.

| Операция | Результат |
| --- | --- |
| Clear index | 1.62 s |
| Полная индексация 30 tracker items, batch 10 | 3.95 s |
| `project management` HTTP | 0.791 s |
| `rankingalpha` HTTP | 0.412 s |
| `проектами` HTTP | 0.422 s |
| `C#` broad HTTP | 0.928 s |

Это development timings без concurrency/load и не production benchmark.
Они доказывают только, что текущая конфигурация исправна на малом корпусе.

Index настроен:

- direct indexing после сохранения;
- cron limit `10`;
- default tracker LIFO;
- reference change tracking;
- delete on fail.

При росте контента нужно наблюдать:

- index queue/cron duration;
- DB index table size;
- p50/p95 query latency и slow SQL;
- cache hit/miss;
- reindex window;
- impact facets and partial matching;
- concurrent editors and anonymous traffic.

Database backend официальный проект описывает как решение прежде всего для
testing/smaller sites. Поэтому «перейти на Solr сейчас» преждевременно, но
«остаться на DB независимо от результатов» тоже не должно быть требованием.

## 10. Уровни будущего поиска

| Уровень | Естественная реализация | Текущая готовность |
| --- | --- | --- |
| Глобальный site search | Один Search API index + View + current-language filter | Близко; language/filter policy не настроена |
| Technical knowledge search | Structured fields, language analyzers, protected tech terms | DB baseline недостаточен для RU/tech tokens |
| Asset catalog search | Views filters; затем Facets при необходимости | Index fields/facets не настроены |
| Related content | Shared taxonomy/references + View; MLT только при доказанной пользе | Rule-based вариант возможен без custom |
| AI-agent retrieval | Access-aware language-specific search endpoint with IDs/metadata | API/endpoint закрыт; отдельное решение |
| Semantic/vector search | Отдельный следующий слой после quality corpus/evaluation | Не требуется на текущей фазе |

Не следует смешивать site search, Asset filtering и AI retrieval в один View.
Они могут использовать общий content model и backend, но требуют разных query,
access и response contracts.

## 11. Search API DB versus Solr

| Критерий | Database backend | Solr |
| --- | --- | --- |
| Запуск/эксплуатация | Работает в текущем Drupal DB | Нужен отдельный сервис/configset |
| Малый простой поиск | Достаточен | Избыточен без требований |
| English stemming | Baseline работает | Language analyzer настраивается точнее |
| Russian morphology | Практически не работает | Russian Snowball/Light stemmers доступны |
| Tech tokens/synonyms | Ограниченная tokenizer policy | Word delimiter/protected words/synonyms |
| Facets | Backend умеет, нужен Facets contrib | Сильная native facet support + contrib |
| Spellcheck/suggestions | Нет в текущей сборке | Native components + Drupal integration |
| Highlighting | Работает через Search API processor | Более гибкий highlighter |
| Grouping/MLT | Ограничено | Backend features доступны |
| Масштабирование | Drupal DB разделяет нагрузку с CMS | Search workload вынесен отдельно |
| Beget | Не требует отдельного daemon | Shared hosting нельзя предполагать; VPS/cloud/container нужен для контроля сервиса |

### Когда Solr действительно нужен

Переход оправдан, если после production-like evaluation подтвержден хотя бы
один обязательный критерий:

- качественная RU morphology и language-specific analysis;
- protected technical vocabulary/synonyms/spellcheck;
- facets с приемлемой latency на реальном Asset/Article объёме;
- autocomplete/suggestions;
- search load мешает primary Drupal DB;
- p95/search relevance не проходят критерии;
- нужны backend grouping/MLT или более сложный query model.

### Когда Database backend ещё достаточен

- небольшой публичный сайт;
- сотни/несколько тысяч документов, подтверждённые нагрузочным тестом;
- простая keyword выдача;
- ограниченное число Views filters;
- отсутствие обязательной RU morphology/spellcheck;
- приемлемая relevance на golden query set.

### Beget

Нельзя считать Solr доступным на обычном shared hosting без письменного
подтверждения Beget. Естественный deployment при необходимости Solr — Beget
VPS/cloud либо отдельный managed Solr. Это архитектурное и операционное решение:
ресурсы JVM, backups, upgrades, monitoring, network isolation и TLS должны быть
оценены отдельно. В этом исследовании сервис не разворачивался.

## 12. AI readiness

Текущий HTML View пригоден человеку, но не является стабильным AI contract.
Для будущего retrieval нужны:

- stable UUID/entity ID и canonical URL;
- `langcode`, original language и available published translations;
- entity type/bundle;
- title, summary, clean body/chunks;
- taxonomy/technology/Asset fields как отдельные metadata;
- published/access state;
- updated timestamp/revision ID;
- deterministic filters и ranking metadata;
- source citations для ответа агента.

Search result API сейчас не открывался. Не следует разрешать JSON:API только
ради поиска: обычный entity API не заменяет ranked retrieval endpoint. Сначала
нужен отдельный read-only access contract и evaluation set.

Semantic/vector search не нужен до решения lexical baseline: иначе embeddings
маскируют проблемы language/access/content quality. Классификация на текущем
этапе: **Лучше скорректировать требование** — сначала качественный structured
lexical search, затем отдельное исследование semantic retrieval.

## 13. Рекомендуемый baseline E+E Master

1. Сохранить один Search API index для published nodes и Canvas Pages.
2. Утвердить RU default `/`, EN `/en`, затем починить language negotiation.
3. Добавить обязательный current-language filter в public search View.
4. Индексировать отдельно: bundle, created/changed, topics, technologies,
   category, Asset status и только подтверждённые catalog fields.
5. Оставить rendered item как catch-all; очистить `search_index` view modes от
   автора/служебного текста, если он не нужен в snippets.
6. Составить golden query set RU/EN, включая доменные сокращения и no-result
   cases, с ожидаемыми top results.
7. Не добавлять difficulty field, facets или autocomplete до утверждения UX.
8. Если фильтров достаточно — использовать Views + Better Exposed Filters.
9. Если нужны counts/drill-down — отдельно согласовать зрелый Facets contrib.
10. Провести Solr proof-of-concept до production только если обязательные RU,
    relevance или scale criteria не проходят на DB.
11. Не писать custom search code до доказанного ограничения Search API/Solr.

## 14. Что скорректировать в требованиях E+E Master

1. Не считать наличие Search API доказательством качественной RU morphology.
2. Не требовать один и тот же fallback для direct URLs и search listings.
3. Не искать Asset status/category только как слова в body — индексировать их
   как structured fields.
4. Не обещать корректный поиск `C#`, `CI/CD`, `ИТ` без approved technical
   vocabulary/analyzer policy.
5. Не требовать facets «по умолчанию»: сначала проверить, нужны ли пользователю
   обычные filters.
6. Не начинать с semantic/AI search; сначала построить измеримый lexical
   baseline и access-safe metadata.
7. Не выбирать Solr только за потенциальную мощность; выбрать его по
   acceptance criteria и эксплуатационной готовности Beget.

## Матрица требований

| Требование | Классификация | Вывод |
| --- | --- | --- |
| Global node + Canvas search | Varbase штатно / входит contrib | Практически работает |
| Technical Article long text/code | Входит contrib | Rendered fields и code text searchable |
| Taxonomy/Media full text | Входит contrib | Только через rendered item; отдельные fields не настроены |
| Canvas component inputs | Varbase штатно / входит contrib | Published tree text подтверждён |
| EN stemming | Входит contrib | Практически работает |
| RU morphology | Требуется дополнительный contrib | DB baseline не сводит словоформы |
| `ё/е` equivalence | Требуется дополнительный contrib | Tokens различаются |
| Tech tokens `C#`, `CI/CD`, `ИТ` | Лучше скорректировать требование | Structured technology field + analyzer policy |
| Title/relevance boost | Входит contrib | Title boost 3, exact-title result first |
| Editorial promoted boost | Входит contrib | Потенциально через indexed field/sort; не настроено |
| Snippets/highlighting | Varbase штатно / входит contrib | Работает, требует cleanup render mode |
| Pagination | Varbase штатно / входит contrib | 10/page работает |
| Content type/language/date/taxonomy filters | Drupal core + входит contrib | Возможны, fields/View не настроены |
| Better Exposed Filters UX | Входит contrib | Модуль есть, search View его не использует |
| Facets | Требуется дополнительный contrib | Facets module отсутствует |
| Autocomplete | Требуется дополнительный contrib | Search API Autocomplete отсутствует |
| Spellcheck/did-you-mean | Требуется дополнительный contrib | Baseline отсутствует; Solr candidate |
| Published/draft boundary | Drupal core + входит contrib | Draft nodes/translations/Canvas исключены |
| Role-restricted search | Drupal core + входит contrib | Query access включён; нужен реальный grants regression |
| Single-language material | Drupal core + входит contrib | EN-only и RU-only items индексируются |
| Current-language results | Входит contrib | Metadata есть, View filter отсутствует |
| Database backend для малого сайта | Входит contrib | Достаточность условная, нужен real-corpus test |
| Solr | Требуется дополнительный contrib | Нужен только по подтверждённым criteria |
| Custom search | Возможный custom | Оснований нет |

## Нерешённые вопросы перед production-конфигурацией

- утвердить default language и URL policy из исследования 03;
- определить golden RU/EN query set и relevance acceptance criteria;
- утвердить searchable/filterable fields для Article, Service и Asset;
- решить, нужны ли facets либо достаточно exposed filters;
- согласовать UX autocomplete/spellcheck;
- оценить реальный объём материалов и запросов;
- уточнить вариант размещения Solr на Beget, если criteria потребуют backend;
- определить public search versus authenticated/private indexes;
- спроектировать отдельный AI retrieval contract, не открывая API сейчас.

Ни один обнаруженный пункт не требует custom-кода на текущем этапе.

## Evidence

### Локальная установка

- UI: `/search`, `/admin/config/search/search-api`, Search View;
- temporary published nodes 19 EN и 20 RU;
- Search API status/clear/index через Drush;
- active config `search_api.index.content`, `search_api.server.database`,
  `views.view.search`;
- physical DB tables `search_api_db_content*`, `search_api_item`;
- rendered Canvas and node search results;
- anonymous HTTP timings в локальном DDEV.

### Первичные источники

- [Search API project](https://www.drupal.org/project/search_api)
- [Search API backend features](https://www.drupal.org/docs/8/modules/search-api/getting-started/server-backends-and-features)
- [Search API extension modules](https://www.drupal.org/docs/contributed-modules/search-api/extension-modules)
- [Search API Solr project](https://www.drupal.org/project/search_api_solr)
- [Apache Solr language analysis](https://solr.apache.org/guide/solr/latest/indexing-guide/language-analysis.html)
- [Apache Solr filters and word delimiters](https://solr.apache.org/guide/solr/latest/indexing-guide/filters.html)
- [Apache Solr highlighting](https://solr.apache.org/guide/solr/latest/query-guide/highlighting.html)
- [Apache Solr suggester](https://solr.apache.org/guide/solr/latest/query-guide/suggester.html)
- [Apache Solr spell checking](https://solr.apache.org/guide/solr/latest/query-guide/spell-checking.html)
- [Beget VPS/cloud manual](https://beget.com/ru/kb/manual/vps)
