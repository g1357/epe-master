# Исследование №6: SEO, формы и обратная связь

- Статус: практический проход завершён
- Дата: 2026-08-21
- Baseline: Varbase 11.0.0-rc1 / Drupal 11.4.5 / PHP 8.4.22
- Среда: локальный DDEV, full-вариант Varbase
- Ограничения: без custom-кода, новой темы и новых contrib-модулей

## 1. Цель и границы

Исследование проверяет штатный путь для технического SEO, публичных форм и
закрытой редакционной обратной связи E+E Master. Созданные формы, submissions,
aliases и материалы являются временными исследовательскими прототипами и не
задают окончательную бизнес-модель.

Не выполнялись архитектурные изменения: не включались дополнительные модули,
не создавался custom-код, не настраивался production mail transport и не
утверждалась юридическая политика хранения персональных данных.

Официальные источники: [Metatag](https://www.drupal.org/project/metatag),
[Schema.org Metatag](https://www.drupal.org/project/schema_metatag),
[Simple XML Sitemap](https://www.drupal.org/project/simple_sitemap),
[Redirect](https://www.drupal.org/project/redirect),
[Webform](https://www.drupal.org/project/webform) и
[Schema.org](https://schema.org/).

## 2. Методика и временные прототипы

Проверены активная конфигурация через Drush, административный UI и фактически
отрендеренный HTML для Technical Article, Service, Asset и Canvas Page. Для
форм выполнены анонимный HTTP-доступ, обязательность полей, CAPTCHA/Antibot,
реальная отправка синтетического submission, хранение в БД и два письма в
DDEV Mailpit.

Созданы две временные формы:

1. `research_06_contact_prototype`: имя, email, организация, тема, сообщение,
   обязательное согласие и CAPTCHA; уведомление администратору и подтверждение
   отправителю.
2. `research_06_article_feedback_pro`: тип обращения, необязательный email,
   сообщение, источник, начальный статус и согласие; уведомление редакции.

Для redirect regression alias временной статьи изменён с
`/project-management-search-benchmark` на `/research-seo-alias-v2`.

## 3. Фактический baseline

| Область | Включено | Версия / состояние |
| --- | --- | --- |
| Metadata | Metatag и submodules Open Graph, Twitter Cards, hreflang | 2.2.0 |
| Aliases | Pathauto | 1.15 |
| Redirects | Redirect, Redirect 404, Redirect Domain | 1.13 |
| Sitemap | Simple XML Sitemap | 4.2.3 |
| Structured data | Schema.org Metatag | 3.0.4 |
| SEO UX | Real-time SEO / Yoast SEO | 2.2 |
| Forms | Webform, Webform UI/Templates, Webform Views | 6.3.0 / 5.6 |
| Anti-spam | Antibot, Honeypot, CAPTCHA, FriendlyCaptcha, Flood Control | включены |
| Mail | Mail System, Symfony Mailer Lite, Easy Email, Reroute Email | включены; default остаётся PHP mail |
| Public comments | Drupal core Comment | выключен |

`reCAPTCHA` также включён, но ключи не настроены. Это не повод использовать
несколько CAPTCHA одновременно.

## 4. Metadata и indexability

### 4.1 Глобальные defaults

Node default использует:

- title: SEO title с fallback на node title и site name;
- description: SEO description с fallback на description;
- canonical: абсолютный URL текущей страницы;
- Open Graph и Twitter image: SEO image с fallback на featured image;
- `og:type=article` для всех nodes.

Фактический HTML:

| Страница | Title | Description | Canonical | OG/Twitter image | JSON-LD |
| --- | --- | --- | --- | --- | --- |
| Technical Article | да | да | да | да | нет |
| Service | да | да | да | да | нет |
| Asset | да | нет, поле пусто | да | нет, поле пусто | нет |
| Canvas home | да | нет | конфликт `/home` и `/` | частично | нет |

Для Service и Asset общий `og:type=article` семантически неточен. Это решается
bundle-level Metatag defaults, а не custom-кодом.

### 4.2 Draft, Archive и 404

Draft/Archived node недоступен anonymous и возвращает 404; опубликованный node
возвращает 200. Следовательно, неопубликованный материал не индексируется по
обычному публичному URL. Администратор всё ещё видит metadata на защищённой
странице, что нормально.

Baseline Metatag default для 404 задаёт canonical на корень сайта и не задаёт
`noindex`. HTTP 404 сам по себе достаточен для удаления URL из индекса, но
canonical на homepage вводит поисковую систему в заблуждение. Перед production
следует убрать canonical с 404 и при желании явно добавить `noindex`; это
штатная конфигурация Metatag.

### 4.3 Homepage

Front-page default правильно использует `[site:url]`, но фактический Canvas
homepage отрендерил canonical `/home`, тогда как hreflang указывал `/`.
Нужно устранить конфликт source node alias/front route до production. Не
следует исправлять его custom-кодом.

## 5. Schema.org

Включены только schema plugins Article, ItemList, WebPage и WebSite. UI
подтвердил доступные типы:

- Technical Article: `TechArticle` через Article — естественное соответствие;
- Asset: `RealEstateListing` через WebPage — приемлемое соответствие для
  страницы конкретного предложения;
- breadcrumbs: `BreadcrumbList` через ItemList;
- Organization доступна как вложенный author/publisher object, но отдельный
  Organization plugin выключен;
- Service plugin присутствует в установленном пакете, но выключен.

Ни одного Schema mapping в baseline не настроено, поэтому JSON-LD на
проверенных страницах отсутствует. Попытка создать временный mapping через UI
не была сохранена; активная конфигурация не изменилась.

Рекомендация: после утверждения bundles создать bundle-level mappings для
Technical Article и Asset; отдельно согласовать включение уже установленного
`schema_service`. Не имитировать Service через Article и не собирать полный
Organization graph из неподходящих полей. Проверять результат валидатором по
фактически отрендеренному JSON-LD.

Официальные типы: [TechArticle](https://schema.org/TechArticle),
[Service](https://schema.org/Service),
[RealEstateListing](https://schema.org/RealEstateListing),
[Organization](https://schema.org/Organization) и
[BreadcrumbList](https://schema.org/BreadcrumbList).

## 6. Aliases и redirects

Baseline Pathauto содержит patterns только для Blog (`/blog/[node:title]`) и
Page (`/[node:title]`). Для исследовательских bundles отдельные patterns
отсутствуют.

Практический regression test подтвердил:

- после смены alias автоматически создан 301;
- старый URL делает один hop на новый;
- новый URL возвращает 200;
- canonical и Open Graph URL указывают новый alias;
- redirect хранится как отдельная сущность и виден в Redirect UI.

Рекомендуемая будущая политика:

- короткие стабильные разделы `/articles/...`, `/services/...`, `/assets/...`;
- не включать изменчивые taxonomy terms в canonical path;
- после принятого multilingual решения: RU без prefix, EN под `/en/...`;
- поддерживать один прямой redirect от каждого старого alias к текущему URL,
  не создавать цепочки;
- удалённый URL перенаправлять только на семантически эквивалентного преемника;
  не отправлять всё на homepage.

Это configuration-level решение Pathauto/Redirect.

## 7. Sitemap и robots

`/sitemap.xml` работает как один `urlset`, а не sitemap index. Включены
hreflang и `skip_untranslated`; максимальный chunk — 2000 URL, генерация идёт
по cron. Фактически в sitemap включены только опубликованные Blog и Page с
priority `0.9` и changefreq `daily`. Research Technical Article, Service,
Asset, taxonomy и Media туда не попали.

Рекомендуемый production scope:

- включать опубликованные canonical landing pages Technical Article, Service,
  Asset и утверждённые Canvas Pages;
- исключать Webform standalone routes и submissions;
- исключать Media entity pages и taxonomy terms, пока они не стали реальными
  публичными landing pages;
- не включать draft/archive;
- полагаться прежде всего на корректный `lastmod`, а не задавать всем URL
  искусственные `daily`/`0.9`;
- сохранить отсутствие несуществующего перевода благодаря
  `skip_untranslated`.

`robots.txt` — стандартный Drupal: закрывает служебные routes, но не содержит
Sitemap directive. Добавление production URL sitemap следует выполнить
deployment/scaffold-конфигурацией после утверждения домена. Robots не является
механизмом защиты приватных данных.

## 8. Contact form

### 8.1 Проверенный сценарий

В форме настроены обязательные name/email/topic/message/consent, типизированный
email, select topic и explicit CAPTCHA. Browser required validation остановила
пустую отправку. Синтетическая заполненная заявка успешно создала submission.

Отправлены два письма и перехвачены Mailpit:

- уведомление на site mail;
- подтверждение на `research06@example.test`.

В обоих письмах From — адрес сайта, а email посетителя используется как
Reply-To. Это правильная базовая модель для SPF/DMARC.

Anonymous HTTP получил FriendlyCaptcha и Antibot. Администратор CAPTCHA не
видел из-за permission bypass, поэтому anti-spam нельзя проверять только под
UID 1.

### 8.2 Canvas embedding

В Canvas уже зарегистрирован Webform Block component и pattern
`contact_split`. Существующая Canvas Page `/contact-us` действительно выводит
Business Contact через этот block. Следовательно, форму можно переиспользовать
на Canvas Page и в template без custom component.

Одна Webform должна оставаться самостоятельной reusable сущностью; Canvas
управляет размещением, а не копирует поля и handlers.

## 9. Обратная связь к статье

Прототип принимает Question, Comment, Correction или Suggestion, сообщение и
необязательный email. Anonymous access был временно отключён и дал 403, затем
восстановлен и снова дал 200. Оба режима являются штатной настройкой.

Рекомендуется anonymous feedback: регистрация пользователей baseline закрыта,
а обязательный аккаунт заметно снижает количество полезных исправлений.

### 9.1 Контекст материала

На standalone route токен source entity остался literal, а URL/title указали
страницу самой формы. Это важное ограничение: контекст статьи появляется только
при embedding формы в node/Canvas template с корректной source entity.
Существующая Canvas contact page подтверждает передачу source entity, но точный
feedback token необходимо повторно проверить после создания окончательного
Technical Article template.

Не нужен custom component: сначала использовать Webform Block в Canvas Content
Template.

### 9.2 Редакционная обработка

Webform Results предоставляет таблицу, сортировку, keyword/state filter, star,
lock и notes. Export поддерживает CSV/TSV/HTML/JSON/YAML. `webform_views`
включён, поэтому отдельную административную очередь можно собрать Views.

Однако Webform не является ticket workflow: скрытое публичное поле `status`
может быть подменено, а built-in state не моделирует New/In progress/Resolved.
Для маленькой команды достаточно star/notes и административного View. Если
появится обязательный SLA/workflow, сначала оценить Webform Views/ECA и только
потом custom entity.

Submissions не следует публиковать, индексировать Search API или открывать в
JSON:API. Будущий AI-agent должен получать отдельное минимизированное
административное представление только нерешённых обращений, без лишнего PII и
с отдельной ролью.

## 10. Comments или Webform

Для E+E Master v1 естественнее Webform feedback:

- обращение закрыто от публики;
- тип структурирован;
- есть email notification, consent, export и anti-spam;
- регистрация не нужна;
- отсутствует риск публичной модерации дискуссии.

Core Comment сейчас выключен. Его стоит рассматривать только при подтверждённом
требовании публичной дискуссии, threading и публичной истории ответов. Это
увеличит moderation, spam, privacy и notification scope. Миграция Webform
submissions в comments возможна только как отдельное mapping/import-решение и
не является автоматической.

Классификация требования «публичные комментарии сейчас» — **Лучше
скорректировать требование**.

## 11. Anti-spam и abuse control

Baseline содержит Antibot, Honeypot, CAPTCHA/FriendlyCaptcha, reCAPTCHA и Flood
Control, но наличие модулей не означает, что их надо складывать вместе.

Фактическое состояние:

- Antibot действует на Webform routes;
- новый contact prototype получил explicit FriendlyCaptcha;
- новый feedback prototype работает только с Antibot;
- глобальный Honeypot не защищает все Webforms;
- per-form rate limits не настроены;
- reCAPTCHA keys отсутствуют.

Рекомендуемая лестница:

1. Antibot + Honeypot для низкого трения;
2. Webform total/user/IP limits по фактическому риску;
3. FriendlyCaptcha только при измеренном abuse;
4. не запускать параллельно несколько CAPTCHA.

CAPTCHA — **Входит contrib**, per-form validation/limits — **Входит contrib**;
решение о внешнем CAPTCHA provider требует отдельной privacy/security оценки.

## 12. Email и Beget readiness

Сейчас `system.mail` использует PHP mail, DSN Symfony Mailer Lite — sendmail,
а Reroute Email выключен. DDEV всё равно безопасно перехватывает письма в
Mailpit. Отдельного SMTP module в baseline не обнаружено.

Перед production на Beget нужно:

- заменить `webmaster@vardot.com` адресом домена E+E Master;
- выбрать и проверить поддерживаемый Beget transport (локальный sendmail или
  authenticated SMTP — отдельное инфраструктурное решение);
- использовать доменный From и submitter только в Reply-To;
- настроить SPF, DKIM и DMARC;
- проверить RU/EN subjects и bodies, envelope sender, bounce и logs;
- выполнить delivery test вне DDEV.

Webform handlers имеют conditions и language settings, а configuration
translation доступна. Русский перевод labels/options contact prototype сохранён
через UI. Полный public `/ru` regression зависит от нерешённой language
negotiation из исследования №3 и здесь не менялся.

## 13. Privacy и хранение

Submission хранит values, UID, langcode, timestamps, URI и `remote_addr`.
Схема таблицы не содержит user-agent. В прототипе purge выключен, то есть
данные сохраняются бессрочно. Email, имя, организация, сообщение и IP являются
данными, которые требуют обоснования и контроля доступа.

Рекомендуемый минимальный baseline:

- обязательный понятный consent с ссылкой на privacy policy;
- feedback email оставить необязательным;
- не разрешать attachments в v1 без подтверждённой необходимости;
- отключить сохранение IP в Webform, если оно не требуется для расследования
  abuse/юридической цели;
- ограничить Results/Export ролями Content Admin/Site Admin;
- принять политику удаления или анонимизации, ориентир для обсуждения — 180
  дней, а не считать его юридически утверждённым сроком;
- удалять контролируемые exports после использования;
- не пересылать PII в AI/search/API по умолчанию.

Выбор точного срока и правового основания требует отдельного решения владельца
данных, а не Drupal custom-кода.

## 14. Роли

Фактический baseline:

- Content editor и SEO admin видят Webform overview, но не являются владельцами
  глобальной form/submission configuration;
- Content admin создаёт/редактирует Webforms и имеет расширенный overview;
- Site admin имеет `administer webform` и `administer webform submission`;
- SEO admin уже может редактировать content, aliases, redirects и Yoast; это не
  узкая read-only SEO-роль;
- bundle metadata редактируется вместе с node и наследует content permissions;
- глобальные Metatag/Schema defaults должны оставаться у trusted site
  administrators.

Рекомендация: редактор меняет поля SEO своего контента; SEO admin управляет
metadata/aliases/redirects в доверенной редакции; Content Admin обрабатывает
forms/submissions; Site Admin управляет Webform и global SEO configuration.
После утверждения bundles permissions проверяются явно.

## 15. Классификация требований

| Требование | Классификация | Вывод |
| --- | --- | --- |
| Titles, descriptions, canonical, OG/Twitter | Входит contrib | Metatag baseline достаточен |
| Stable aliases и automatic 301 | Входит contrib | Pathauto + Redirect проверены |
| XML sitemap и hreflang | Входит contrib | Нужно включить окончательные bundles |
| TechArticle/RealEstateListing JSON-LD | Входит contrib | Нужен bundle mapping |
| Service JSON-LD | Входит contrib | Plugin уже установлен, но выключен; нужно согласие на включение |
| Web contact form | Varbase штатно | Webform baseline и Mailpit проверены |
| Reusable form in Canvas | Varbase штатно | Webform Block и Contact Split pattern |
| Private article feedback | Входит contrib | Webform естественнее Comment |
| Public comments сейчас | Лучше скорректировать требование | Не включать без подтверждённого community use case |
| Basic anti-spam | Входит contrib | Antibot/Honeypot, CAPTCHA по риску |
| Production email | Входит contrib | Текущий mail stack может работать с sendmail; если Beget потребует SMTP, отдельно оценить transport/module |
| Ticket-grade feedback workflow | Возможный custom | Только после проверки Views/ECA и реального SLA |
| Privacy retention | Лучше скорректировать требование | Сначала политика и минимизация, потом настройка purge |

## 16. Рекомендуемая естественная архитектура E+E Master

1. Nodes/Canvas Pages хранят SEO fields; bundle defaults обеспечивают единый
   metadata/Schema contract.
2. Pathauto создаёт короткие стабильные aliases, Redirect сохраняет старые URL.
3. Simple XML Sitemap индексирует только реальные опубликованные landing pages.
4. Webform остаётся reusable entity, а Canvas/Webform Block только размещает её.
5. Contact и private feedback остаются отдельными forms с разной data policy.
6. Comments не включаются в v1.
7. Anti-spam начинается с невидимых методов и усиливается по метрикам.
8. Submissions закрыты от search/API/AI; доступ и retention минимальны.
9. Production mail и юридические сроки утверждаются отдельно.

Эта схема соответствует принципу platform first, custom last. Реального
ограничения, требующего custom-кода на текущем этапе, не обнаружено.

## 17. Нерешённые вопросы перед production

- согласовать включение `schema_service` и окончательные Schema mappings;
- устранить canonical `/home` versus `/`;
- утвердить RU default/no-prefix и EN `/en` из исследования №3;
- определить реальные sitemap bundles и landing taxonomy;
- выбрать Beget mail transport и выполнить external delivery test;
- утвердить privacy text, purpose, retention и IP policy;
- проверить source entity feedback после финального Canvas Content Template;
- проверить permissions после создания production bundles/forms.

Исследование №6 завершено. Следующий этап автоматически не начинается.
