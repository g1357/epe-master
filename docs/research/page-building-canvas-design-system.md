# Исследование построения страниц: Canvas и Varbase Design System

- Статус: первый практический проход завершён
- Дата: 2026-08-18
- Baseline: Varbase 11.0.0-rc1, Drupal 11.4.5
- Область: Canvas, Vartheme BS5/Varbase Design System, Storybook, SDC,
  reusable patterns и Canvas Pages

## Границы исследования

Исследование выполнено на чистой full-установке Varbase. Не создавались custom-
модули, custom-тема, custom-компоненты и окончательный контент E+E Master.
Использованы только включённые модули, компоненты, recipes и UI платформы.

Прототипы являются временными исследовательскими материалами. Их тексты и
изображения — демонстрационный контент Varbase, а не проектирование сайта.

## Установленный стек

| Механизм | Фактическая версия/состояние | Роль |
| --- | --- | --- |
| Drupal Canvas | `drupal/canvas` 1.10.1, enabled | Canvas Pages, visual editor, component trees, patterns, global regions |
| Canvas Override | `drupal/canvas_override` 1.0.0-beta3, enabled | Дополнительные возможности изменения Canvas-вывода |
| Varbase Components | `drupal/varbase_components` 4.0.0-rc1, enabled | Интеграция Varbase-компонентов |
| Vartheme BS5 | enabled, default theme | Bootstrap 5 design system и SDC library |
| Storybook | `drupal/storybook` 1.0.4; daemon running | Изолированная документация и визуальная проверка SDC |
| SDC development | `drupal/sdc_devel` 1.0.3 | Инструменты разработки/диагностики SDC |
| Layout Builder | Drupal core, enabled | Отдельный штатный механизм layout; не смешивался с прототипами Canvas |

Canvas 1.10.1 является стабильным пакетом, но сам Varbase baseline остаётся
release candidate. Production-ready вывод требует отдельной проверки обновлений,
permissions, accessibility, performance и migration behavior.

## Модель механизмов

### SDC-компоненты

Vartheme BS5 содержит 125 файлов `*.component.yml`. Фактические группы:

| Группа | Количество |
| --- | ---: |
| atoms | 41 |
| molecules | 25 |
| organisms | 41 |
| pages | 7 |
| foundation | 7 |
| getting-started | 2 |
| base | 1 |
| templates | 1 |

Для page building особенно значимы:

- layout: Section, Container, Row, Column, Group, Grid, Divider, Spacer;
- typography: Heading, Text, Rich Text, Plain Text, Blockquote, List, Anchor;
- hero: Hero Slider, Hero Slide, Hero side by side, Hero Card, Hero CTA;
- cards: обычные, icon, featured, overlay, pricing, testimonial, text;
- media: Image, Video, Figure, Media banner;
- interaction: Accordion, Tabs, CTA, Carousel, Share;
- Drupal data: Views displays, navigation, breadcrumbs, language switcher,
  taxonomy, webform и blog listings.

Canvas отображает не все найденные компоненты плоским списком. Library формирует
редакторские категории и скрывает административные/неподходящие компоненты.
На проверенном сайте доступны категории Base, Card, Hero, Layout, Lists (Views),
Menus, Navigation, Other, System, Views и Webform.

### Patterns

Recipe `varbase_starter` поставляет 14 готовых Canvas patterns:

1. Hero Slider;
2. Page Intro Banner;
3. Text with Cards;
4. Feature Cards;
5. Counters;
6. Feature Highlight;
7. Latest Blog Posts;
8. FAQ Accordion;
9. Image Cards;
10. Contact Form with Info;
11. Call to Action Cards;
12. Call to Action Banner;
13. Site Header;
14. Site Footer.

Patterns — повторно вставляемые снимки component tree. После вставки копия
редактируется независимо. Header и Footer также используются как global regions,
то есть общий каркас отделён от page content.

### Storybook

В работающем Storybook обнаружено 409 entries (`story` и `docs`):

| Раздел | Entries |
| --- | ---: |
| Atoms | 87 |
| Base | 62 |
| Cards | 63 |
| Foundation | 7 |
| Getting Started | 2 |
| Hero | 29 |
| Layout | 50 |
| Molecules | 76 |
| Organisms | 25 |
| Others | 8 |

Storybook подтверждает состояния и варианты компонентов: размеры, цвета,
spacing, responsive images, ссылки, icons, alignment и Bootstrap utilities.
Это каталог дизайн-системы и инструмент визуальной проверки, но не источник
контента и не page builder.

Локальное ограничение: UI доступен по `http://storybook.ee-master.ddev.site`,
но `index.json` доступен через DDEV Storybook port
`http://ee-master.ddev.site:6007/index.json`. Proxy-домен возвращает Drupal 404
для `index.json` и `iframe.html`. Для разработки это обходится прямым DDEV URL;
до CI/remote Storybook потребуется отдельная проверка конфигурации proxy.

## Практические эксперименты

### Эксперимент 1: корпоративная главная

- Canvas Page: `Research Prototype — Corporate Home`;
- alias: `/research/prototype-corporate-home`;
- статус: Published;
- вставлены штатные Hero Slider и Feature Cards;
- header/footer получены из global regions;
- публичная страница возвращает HTTP 200 и содержит hero, CTA и три feature card.

Подтверждено:

- крупный hero со слайдером;
- секции преимуществ/направлений;
- CTA, cards, counters, blog listing, form и footer присутствуют в готовой
  библиотеке, даже если не все были включены в этот минимальный прототип;
- responsive preview и настройки Section доступны в editor UI;
- SEO/metatag fields доступны на уровне Canvas Page.

Вывод: корпоративная главная собирается штатно. Классификация — **Varbase
штатно**.

### Эксперимент 2: страница услуги

- Canvas Page: `Research Prototype — Service Page`;
- alias: `/research/prototype-service`;
- статус: Published;
- вставлен штатный Feature Highlight;
- публичная страница возвращает HTTP 200.

Композиция Page Intro Banner + Feature Highlight + Text with Cards + FAQ + CTA
покрывается готовыми patterns. Rich Text, Accordion, Cards и Webform доступны как
компоненты.

Вывод: маркетинговое представление услуги собирается штатно. Если услуге нужны
структурированные атрибуты, связи, фильтры или единый шаблон для многих услуг,
нужно исследовать content type + Canvas Content Template. Это не подтверждённая
потребность в custom-коде. Текущая классификация визуального сценария —
**Varbase штатно**.

### Эксперимент 3: объект недвижимости

- Canvas Page id 8;
- исследовательское название и alias введены в editor;
- статус: Draft;
- во время повторного открытия Library вкладка Canvas SPA завершилась;
- опубликованная revision осталась `Untitled page`, а pending editor change
  восстановился при повторном открытии editor.

В библиотеке присутствуют Media Banner, Image Cards, cards, taxonomy, Views,
Rich Text, CTA и Webform. Этого достаточно для визуальной карточки объекта.

Но Canvas Page из коробки предоставляет в Page data главным образом title,
alias, SEO/social metadata и image. Цена, площадь, адрес, координаты, статус,
галерея как данные, связи и фильтрация должны быть структурированными полями,
если проект действительно требует их поиска, импорта, API или повторного вывода.

Вывод:

- презентационный landing объекта — **Varbase штатно**;
- каталог структурированных объектов — предположительно **Drupal core**
  (content type, fields, media, taxonomy, Views) + Canvas Content Template;
- окончательная модель откладывается до исследования Content types, Media,
  Views и Canvas templates;
- custom-код не обоснован.

### Эксперимент 4: документационная/техническая статья

- Canvas Page id 9;
- исследовательское название и alias введены в editor;
- статус: Draft;
- editor показывает pending changes, но базовая revision остаётся
  `Untitled page` до publish workflow.

Для визуальной статьи доступны Page Intro Banner, Heading, Rich Text, Image,
Video, List, Blockquote, Anchor, Accordion и Tabs. Этого достаточно для короткой
и средней технической публикации.

Не подтверждены штатными прототипами:

- автоматически строящееся оглавление;
- versioning документа как предметная модель;
- related documents и structured product references;
- code syntax highlighting как управляемый компонент;
- единый строго ограниченный шаблон для большой базы документации.

Вывод: отдельная техническая статья визуально собирается штатно. Для серии
однородных документов предпочтительно сначала проверить обычный content type,
CKEditor, revisions и Canvas Content Template. Классификация текущего
визуального сценария — **Varbase штатно**; требования к полноценной knowledge
base пока **лучше уточнить/скорректировать** до выбора contrib или custom.

## UX и стабильность редактора

Подтверждены положительные свойства:

- visual editing в реальном frontend theme;
- component library с поиском и категориями;
- independent pattern instances;
- Layers, responsive device preview, zoom, undo/redo;
- Draft/Review/Publish workflow;
- page-level language selector;
- SEO, Open Graph, Twitter Cards и temporary unpublished access;
- настройки Bootstrap utility для Section: container, colors, spacing, border,
  alignment, background media и animation.

Зафиксированные ограничения первого прохода:

1. Canvas — тяжёлый React SPA. Две длительные автоматизированные сессии завершили
   вкладку после серии Library/Patterns операций.
2. Переключение Components → Patterns иногда не срабатывало или превышало
   timeout, хотя список из 14 patterns загружался после повторного открытия.
3. Varbase acceptance tests сами отмечают editor как SPA, который не достигает
   settled load state, и помечают pattern scenarios `slow`/`flaky`.
4. Draft title/alias видны при повторном открытии editor, но content listing и
   `canvas_page_field_revision` продолжают показывать опубликованное значение до
   завершения Review → Publish. Редакторам потребуется обучение этой модели.
5. В editor присутствует AI assistant с privacy gate. Он не использовался и не
   требуется для page building.
6. В DOM editor присутствуют дополнительные ECA debug элементы; перед UX-
   оценкой production baseline нужно проверить, относятся ли они только к full
   development recipe.

Эти наблюдения не доказывают production blocker, но требуют ручного теста
редактора, browser matrix и повторяемого smoke suite перед утверждением Canvas
как основного интерфейса редакторов.

## Reuse и governance

Штатная лестница reuse выглядит так:

1. SDC — контракт и rendering единичного компонента;
2. Storybook story — документированное состояние компонента;
3. Canvas pattern — повторно вставляемая композиция компонентов;
4. global region — общая композиция header/footer;
5. Content Template — предполагаемый единый component tree для структурированного
   content entity;
6. Canvas Page — свободно собранная standalone page.

Для E+E Master пока нельзя выбирать между свободной Canvas Page и Content
Template. Критерий выбора — нужен ли типу страницы структурированный контент,
массовая выборка, фильтрация, API, import/export и единое изменение шаблона.

Не следует создавать новые patterns или code components до инвентаризации
реальных требований и проверки возможности адаптировать их к существующим 14
patterns и 125 SDC.

## Ответ на основной вопрос

| Тип страницы | Визуально без custom | Структурно без custom | Текущий вывод |
| --- | --- | --- | --- |
| Корпоративная главная | Да | Обычно не требуется | Canvas Page + готовые patterns подходят |
| Страница услуги | Да | Вероятно да через content type/template | Landing возможен сейчас; модель серии услуг ещё исследовать |
| Объект недвижимости | Да | Вероятно да через fields/media/taxonomy/Views | Не использовать свободную Canvas Page как базу каталога до проверки templates |
| Техническая статья | Да | Вероятно да через node/CKEditor/revisions/template | Простая статья возможна; knowledge-base требования ещё уточнить |

Ни для одного из четырёх сценариев на этом этапе не подтверждена необходимость
custom PHP, custom theme или custom component.

## Следующие эксперименты

1. Canvas Content Templates для node entity и обязательных slots/props.
2. Multilingual Canvas: отдельная RU/EN revision и отсутствие перевода.
3. Поведение patterns при обновлении SDC/component versions.
4. Роли и permissions: site builder против content editor.
5. Accessibility: keyboard-only editor и итоговый markup patterns.
6. Performance: editor startup, page render/cache и image behavior.
7. Storybook proxy и возможность запуска визуальных regression tests в CI.
8. Сравнение Canvas Page, node + Canvas template, Layout Builder и CKEditor для
   каждого класса контента.

## Первичные источники и evidence

- [Drupal Canvas project](https://www.drupal.org/project/canvas)
- [Drupal Canvas releases](https://www.drupal.org/project/canvas/releases)
- [Canvas ecosystem](https://www.drupal.org/project/canvas/ecosystem)
- `composer.lock`
- `recipes/varbase_starter/config/canvas.pattern.*.yml`
- `web/themes/contrib/vartheme_bs5/components/`
- `tests/features/09-drupal-canvas/`
- локальные Canvas Pages 6–9
- локальный Storybook `index.json`

