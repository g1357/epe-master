# Исследование №8: production readiness и Beget

- Статус: практический проход завершён
- Дата: 2026-08-21
- Baseline: Varbase 11.0.0-rc1 / Drupal 11.4.5 / PHP 8.4.22
- Среда: локальный DDEV, full-вариант Varbase
- Ограничения: без deploy, DNS, secrets, новых contrib-модулей и custom-кода

## 1. Решение в одном абзаце

Для первого запуска E+E Master рекомендуется **вариант C: начать на Beget
shared hosting, но только после короткой квалификационной проверки конкретного
аккаунта**, сохранив переносимость на VPS. Требуются PHP 8.4 CLI и web,
MySQL 8.0 (MySQL 5.7 для Drupal 11 не подходит), Composer 2, SSH, cron, writable
public/private directories, возможность направить DocumentRoot на `web/` или
создать symlink, и достаточный лимит памяти/процесса. Redis, CDN, Solr и
long-running workers для первой версии не нужны. Если хотя бы один обязательный
пункт shared не подтверждается, стартовать на небольшом Beget VPS, а не
адаптировать Drupal под ограничения hosting.

До production есть пять blocker-ов:

1. authoritative config export не хранится в Git;
2. `composer audit` сообщает об уязвимом `paragonie/sodium_compat`;
3. production settings/secrets ещё не спроектированы и не проверены;
4. Varbase 11 имеет статус RC, а automatic update readiness уже не проходит;
5. реальный config import не проходит validation из-за orphaned Tour config.

## 2. Метод и первичные источники

Проверены Composer/Git, DDEV services, Drupal status report, config export,
cache headers, anonymous/authenticated requests, cron/queues, БД, logs,
ImageMagick и реальное восстановление DB backup. Production benchmark и deploy
не выполнялись.

Основные источники:

- [Varbase project и 11.0.0-rc1](https://www.drupal.org/project/varbase);
- [Drupal PHP requirements](https://www.drupal.org/docs/getting-started/system-requirements/php-requirements);
- [Drupal database requirements](https://www.drupal.org/docs/getting-started/system-requirements/database-server-requirements);
- [Drupal web-server requirements](https://www.drupal.org/docs/getting-started/system-requirements/web-server-requirements);
- [Drupal dependency management](https://www.drupal.org/docs/develop/using-composer/manage-dependencies);
- [Beget Composer](https://beget.com/ru/kb/how-to/web-apps/instrukcziya-po-ustanovke-composer);
- [Beget MySQL](https://beget.com/ru/kb/faq/hosting/bazy-dannyh-mysql);
- [Beget Cron](https://beget.com/ru/kb/manual/crontab);
- [Beget shared backup](https://beget.com/ru/kb/manual/backup);
- [Beget VPS backup](https://beget.com/ru/kb/manual/rezervnoe-kopirovanie-vps);
- [Beget hosting limits](https://beget.com/ru/kb/faq/hosting/ogranicheniya-hostinga).

Данные Beget зависят от сервера и тарифа. Перед оплатой требуется ticket с
перечнем проверок из раздела 5; маркетинговое наличие функции не заменяет
проверку конкретного аккаунта.

## 3. Точный локальный baseline

| Компонент | Фактическое состояние | Production-вывод |
| --- | --- | --- |
| Varbase | `vardot/varbase 11.0.0-rc1`, 2026-08-17 | pre-stable; release notes и test restore обязательны перед update |
| Drupal | `drupal/core-recommended 11.4.5` | PHP 8.3/8.4 поддержаны; lock фиксирует dependency set |
| PHP | 8.4.22 CLI, OPcache | выбрать PHP 8.4 и одинаковую версию для web/CLI |
| Composer | 2.10.2 | production install только из lock |
| Drush | 13.7.6 | требует PHP >=8.3 |
| DB | MariaDB 11.8, InnoDB/PDO MySQL | Drupal 11 требует MariaDB 10.6+ или MySQL 8.0+ |
| Web | DDEV Apache-FPM; router Traefik | Apache 2.4.7+ с rewrite/htaccess или корректный Nginx |
| Node | 20 | нужен только для Storybook/theme build, не для runtime сайта |
| DDEV | 1.25.3 | local only |
| Upload limits | 100M upload/post; CLI memory unlimited | для production начать с 20–32 МБ upload, PHP memory 256 МБ минимум |

Включены PHP extensions: `bcmath`, `curl`, `dom`, `fileinfo`, `gd`, `json`,
`mbstring`, `mysqli`, `openssl`, `PDO`, `pdo_mysql`, `SimpleXML`, `xml`,
`xmlreader`, `xmlwriter`, `zip`, `zlib`, OPcache. Image toolkit — ImageMagick 6;
JPEG, PNG, GIF и WebP включены, AVIF выключен. Drupal core требует PDO,
подходящий DB driver, DOM/XML, GD или иной image library; multilingual требует
`mbstring`, HTTPS/update workflows — OpenSSL/cURL.

Файловая политика:

- code, `composer.json`, `composer.lock`, scaffold metadata и patches lock —
  read-only для web user после deploy;
- `web/sites/default/files` — writable web user, без `0777`;
- private files — отдельный абсолютный путь вне web root, writable web user;
- temp — системный или отдельный путь вне web root;
- `settings.php` — readable runtime, не writable web user;
- deploy user может атомарно собирать release и менять symlink, web user — нет.

## 4. Что хранится в Git и что строит Composer

Репозиторий содержит `composer.json`, `composer.lock`, `patches.lock.json`,
DDEV config, docs и project scaffolding. Не версионируются:

- `vendor/`, `bin/`;
- `web/core/`;
- `web/modules/contrib/`, `web/themes/contrib/`, contrib profiles/recipes;
- runtime `web/sites/*/files`, private files, DB и secrets;
- `settings.php`, `.env`, `auth.json`.

Это корректная Composer architecture: `vendor`, Drupal core, contrib modules и
themes должны воспроизводиться командой install из lock, а не коммититься.
Trusted Composer plugins явно перечислены; Varbase использует
`vardot/varbase-patches`, `vardot/drupal-core-patches`, Composer Scaffold и
Recipe Unpack. Patching включён с fail-on-error; `patches.lock.json` обязателен
для review.

Production build policy:

```bash
composer validate --no-check-publish
composer audit --locked
composer install --no-dev --prefer-dist --no-interaction \
  --optimize-autoloader --classmap-authoritative
composer check-platform-reqs --no-dev
```

Не применять `composer update` на production. Update выполняется в отдельной
ветке, обновлённый lock проходит review/test и затем deploy. `composer validate
--strict` сейчас возвращает warning из-за project `version`; это не runtime
ошибка, но CI следует использовать `--no-check-publish` либо исправить upstream
template отдельным решением.

`composer audit --locked` на 2026-08-21 нашёл:

- `paragonie/sodium_compat` affected `<1.24.1` или `>=2,<2.5.1`, advisory
  `PKSA-32g2-byr9-drtw`;
- abandoned `oomphinc/composer-installers-extender` без указанной replacement.

Автоматическое обновление в этом исследовании не выполнялось. Advisory — blocker
до launch: обновление следует сделать через Composer в отдельной ветке после
проверки Varbase compatibility.

## 5. Beget shared и VPS/cloud

| Возможность | Shared hosting | VPS/cloud | Оценка E+E Master |
| --- | --- | --- | --- |
| PHP | Beget CLI list включает 8.4; web version выбирается в панели | полный контроль | shared пригоден после проверки одинаковых web/CLI 8.4 |
| DB | MySQL 5.7 **или** 8 в зависимости от сервера | выбрать MySQL 8/MariaDB 10.6+ | аккаунт с 5.7 — blocker |
| SSH/terminal | доступен | root SSH | shared достаточно |
| Composer | системный v1, локально ставится v2 | любой v2 | shared возможен, но build лучше делать CI/off-host |
| Cron | panel/manual, вплоть до минуты | system cron/timers | shared достаточно для 5–15 минут |
| CLI PHP | версии вызываются явно | полный контроль | cron обязан использовать PHP 8.4 binary |
| Git deployment | SSH позволяет Git, но atomic releases не гарантированы | полноценный Git/symlink workflow | shared: upload prepared artifact либо checkout+Composer после теста |
| Symlink/web root | symlink применяется в инструкциях Beget | полный контроль | подтвердить `public_html -> web` на аккаунте |
| Private/writable paths | возможны в home вне `public_html` | полный контроль | shared достаточно после проверки quota/permissions |
| Mail | native mail/sendmail доступен; SMTP возможен приложением | любой MTA/SMTP | для production предпочтителен внешний transactional SMTP |
| SSL/TLS | панель hosting | полный контроль | оба варианта подходят |
| Memcached | сервис доступен на некоторых тарифах | Redis/Memcached устанавливаются | не нужен для launch |
| Redis | не считать гарантированным | устанавливается | trigger для VPS, не baseline |
| Solr | не предполагать наличие | можно развернуть/managed | явный trigger для VPS |
| workers | process/time limits | supervised workers | long-running queue/AI — trigger для VPS |
| backup | бесплатные automatic/on-demand restore | файловые backup; DB надо дампить отдельно | в обоих случаях нужна offsite проверяемая копия |
| CPU/RAM | общие ресурсы и process limits | выбранные ресурсы | Canvas/image spikes надо проверить trial/ticket |

Shared предпочтительнее, если проходит квалификация:

1. web и CLI PHP 8.4 с нужными extensions;
2. MySQL 8.0, InnoDB, `utf8mb4`, `READ COMMITTED` возможно;
3. memory limit не ниже 256 МБ для CLI/web и Composer build не обязан работать
   на сервере;
4. SSH, Cron, symlink/DocumentRoot, private directory и SSL подтверждены;
5. smoke deploy и обработка трёх реальных изображений укладываются в limits;
6. provider backup можно выгрузить, а собственный DB dump восстановить.

Конкретные triggers перехода на VPS:

- Search API Solr;
- обязательный Redis backend или persistent cache tuning;
- long-running queue workers, scheduler consumers или AI runtime;
- cron/Composer/updb завершаются hosting limits;
- PHP memory/CPU/process limit регулярно достигается после оптимизации;
- ImageMagick derivative generation стабильно прерывается;
- shared не даёт MySQL 8, PHP 8.4 CLI/web, symlink или private path;
- sustained traffic и cache-miss latency не укладываются в согласованный SLO.

## 6. Окружения, Git и release flow

Минимальный branch model:

1. `main` — reviewed, deployable;
2. короткая `codex/...`/feature/research branch;
3. pull request, review и зелёный minimal CI;
4. merge в `main`;
5. annotated release tag (`vX.Y.Z`);
6. deploy exact tag;
7. rollback на предыдущий release **только если schema/config совместимы**.

Отдельный `develop` для текущего масштаба не нужен: он создаёт второй источник
истины без отдельной release team.

Сейчас есть операционный риск: Git/docs рабочая копия находится в Windows
`C:\Users\Dukar\Documents\ChatGPT\Drupal`, а работающий DDEV — отдельный clone
`/home/dukar/projects/epe-master`, оставшийся на старом `main` `4b64faa`.
Production work следует вести из одной WSL-копии, а Windows path оставить только
если он является тем же checkout. Автоматически объединять копии в исследовании
не стали.

### Нужен ли staging

Для немедленного информационного launch достаточно DDEV + production только
после стабилизации config и security baseline. Однако **до первого значимого
Varbase/Drupal update после launch нужен staging**, потому что Varbase 11 RC,
Canvas config, multilingual, Search, forms и будущие API затрагивают DB/config.
Практичная модель: не держать staging постоянно до launch, но предусмотреть
воспроизводимый temporary staging из production backup перед каждым update.
Постоянный staging становится оправдан после API/AI integrations, регулярной
редакторской работы или команды из нескольких разработчиков.

## 7. Configuration management

Фактический `drush status`:

```text
config-sync: sites/default/files/sync
```

Эта директория находится в runtime public files и исключена `.gitignore`.
Первый `drush config:status` показал весь active config как `Only in DB`.
`drush config:export -y` успешно создал export. Второй export сообщил, что
active и sync идентичны, но последующий `config:status` всё равно показывал
восемь Canvas objects как `Different` (`canvas.pattern.*` и один content
template). Это указывает на normalization/serialization drift, который нужно
локализовать до production.

Реальный `drush config:import -y` не изменил active config и завершился на
validation error: `tour.tour.honeypot` зависит от extension `tour`, которого не
будет после import. После эксперимента pre-import DB восстановлена, cache
rebuilt, homepage — HTTP 200. Это подтверждает, что текущий export
**не импортируем** и не может считаться release artifact. Исправление должно
быть штатным config cleanup/recipe correction после отдельного review, а не
игнорированием ошибки.

Рекомендуемая модель:

- перенести `config_sync_directory` вне public files в versioned `config/sync`;
- экспортировать и review-ить YAML в Git;
- secrets и environment overrides держать в environment-specific
  `settings.php`, не в config export;
- production не является местом ручного изменения authoritative config;
- `drush config:status` перед release должен быть чистым и повторный export
  должен сходиться;
- import сначала проверяется на clone production DB.

Recipes остаются **bootstrap/composition mechanism**, а exported config —
**authoritative lifecycle state**. Recipe не является generic update/rollback
системой и не заменяет `config:export/import`. Site UUID должен совпадать между
DB и export; новый production site не устанавливать отдельно с другим UUID, а
создавать из утверждённой config/DB lifecycle.

Config Split не нужен для двух окружений, пока простых settings overrides
достаточно. Пакет `config_ignore` есть в Composer, но модуль не был включён;
использовать его для маскировки config drift не следует.

## 8. Deploy и update sequence

Без custom deployment tooling рекомендуемый порядок:

```bash
# traffic handling/maintenance определяется hosting
drush state:set system.maintenance_mode 1 --input-format=integer
drush cache:rebuild

# подтверждённый DB + public/private files backup и release metadata
git checkout <release-tag>
composer install --no-dev --prefer-dist --no-interaction \
  --optimize-autoloader --classmap-authoritative
composer check-platform-reqs --no-dev

drush updatedb --yes
drush config:import --yes
drush cache:rebuild
drush cron

# smoke tests, затем
drush state:set system.maintenance_mode 0 --input-format=integer
drush cache:rebuild
```

`updb` идёт до `cim`: новый code/schema должен существовать до импорта config,
который может ссылаться на новые definitions. Перед `updb`/`cim` обязательны
backup и preview на staging clone. В Drush 13 у `config:import` нет проверенной
опции `--preview`; использовать `config:status`, Git diff export и dry run на
clone, а не полагаться на несуществующий флаг.

Rollback code после необратимого DB update небезопасен. Если update изменил
schema/data или imported config, rollback означает согласованный набор:
предыдущий code release + DB restore + files restore при необходимости + cache
rebuild. Нельзя просто переключить symlink назад.

Update policy:

- еженедельно проверять Drupal security advisories и Composer audit;
- обычные dependency updates — ежемесячным batch;
- Varbase/core minor update — отдельная PR, release notes, full backup,
  staging restore, `updb`, `cim`, smoke tests;
- проверить patches apply, deprecated APIs и Canvas config convergence;
- Automatic Updates/Package Manager не использовать как production authority.

Enabled inventory: core Update Manager, Package Manager, contributed Automatic
Updates и Project Browser включены. Status report: Automatic Updates не проходит
readiness из-за unsupported Composer plugins `vardot/varbase-patches` и
`vardot/drupal-core-patches`. Следовательно Composer/Git deployment остаётся
единственным production механизмом; automatic production updates выключены до
отдельного решения.

## 9. Cache и functional measurements

Включены Page Cache, Dynamic Page Cache, render cache, Views cache mechanisms,
BigPipe; CSS/JS aggregation и compression включены. Homepage cache metadata
включает language, route, theme, permissions/roles и menu active trail
contexts, permanent max-age и granular cache tags.

Измерения после cache rebuild в DDEV:

| Сценарий | Первый запрос | Тёплые запросы | Headers |
| --- | ---: | ---: | --- |
| anonymous homepage | 1.057 s | 0.0106 / 0.0104 s | `Cache-Control: public,max-age=900`, page cache HIT |
| authenticated UID 1 | 0.814 s | 0.0626 / 0.0599 s | page uncacheable, Dynamic Page Cache HIT |

Это functional evidence правильного поведения, не production capacity test.
В DDEV authenticated response существенно больше (≈215 КБ против 97 КБ), что
нормально для admin toolbar/permissions.

Redis/Memcached сейчас не нужны: DB 71 МБ, queues пусты, warm core caches быстры.
CDN/Cloudflare/reverse proxy также не baseline. Начать с HTTPS origin, Drupal
cache и static file headers. CDN вводить при измеренном external latency/traffic
или большом media delivery. При reverse proxy обязательно задать точные trusted
proxy addresses/headers и не доверять произвольным forwarded headers.

## 10. Изображения и прежние 196 секунд

Фактический toolkit — ImageMagick 6, quality 75, WebP включён, AVIF выключен.
Responsive Image, Drimage Improved, Focal Point и Image Optimize включены.
Крупнейшие оригиналы:

- JPEG 4096×2731, 10.4 МБ;
- JPEG 4096×2730, 7.0 МБ;
- WebP 4096×2731, 3.0 МБ.

Повторный isolated test `auto-orient + resize <=1920 + WebP quality 82`:

| Исходник | Время | Результат |
| --- | ---: | ---: |
| 10.4 МБ JPEG | 0.512 s | 137 КБ WebP |
| 7.0 МБ JPEG | 0.429 s | 101 КБ WebP |
| 3.0 МБ WebP | 0.614 s | 194 КБ WebP |

196 секунд не воспроизведены и не объясняются одиночной ImageMagick operation.
Вероятные составляющие прежнего результата: cold DDEV/filesystem I/O на
Windows mount, Media metadata/EXIF/focal point, синхронное создание нескольких
responsive/Drimage derivatives, Canvas rebuild или UI automation wait. Для
точного attribution нужен отдельный profiler trace конкретного upload; менять
image stack без trace не следует.

Production media policy:

- редакторские изображения: обычно <=2560 px long edge и <=5 МБ;
- hero: заранее оптимизированный JPEG/WebP, не загружать 10+ МБ без причины;
- lowercase ASCII filenames, смысловые слова, без secrets/PII;
- сохранять оригинал, derivatives генерировать штатными styles;
- responsive styles и native lazy loading для below-the-fold;
- WebP использовать, AVIF не обещать до hosting/browser pipeline test;
- production upload limit начать с 20–32 МБ, а не DDEV 100 МБ;
- warm critical derivatives после deploy/large media batch, если реальные
  first-hit measurements покажут проблему.

## 11. Cron, queues, data growth и logs

Ultimate Cron включён. Зафиксированы jobs для Search API, sitemap, Scheduler,
file/temp cleanup, dblog, update checks, locale, Webform, ECA, Canvas, OAuth,
password policy, locks и Trash. Однократный `drush cron` успешно завершился и
освободил stale content lock.

Database queue workers сейчас:

```text
ai_automator_field_modifier          0
eca_task                             0
entity_usage_recreate_tracking_data  0
locale_translation                   0
media_entity_thumbnail               0
package_manager_cleanup              0
trash_entity_purge                   0
```

Рекомендация: запуск cron каждые 5 минут после launch (15 минут допустимы до
появления scheduled content), single-run lock и alert при отсутствии успешного
run >30 минут. RabbitMQ/Redis queue не нужен. AI/media bulk/Search Solr или
очередь, регулярно не успевающая за cron, являются trigger для supervised
workers/VPS.

Текущая БД 71.22 МБ, public files 109 МБ / 1050 files. Крупнейшие таблицы:
`router` 9.09 МБ, `cache_discovery` 7.55, `key_value_expire` 6.03,
`cache_config` 5.78, `cache_default` 5.55, `config` 4.36,
`search_api_db_content_text` 3.03 МБ. Caches восстанавливаемы и не относятся к
ценным backup data. Быстрее всего будут расти public media, revisions,
Search API tables, watchdog/audit и Webform submissions.

Logging baseline:

- PHP/web error log + Drupal dblog для launch;
- retain application logs 30 дней, security/audit 90 дней с учётом privacy;
- Webform submission retention задавать по бизнес/legal purpose;
- alert: repeated PHP errors, failed cron/search, mail failures, disk >80%,
  backup/restore failure;
- внешний observability stack пока не нужен.

В dblog есть ожидаемые access-denied API probes, предыдущие исследовательские
PHP parse errors и Webform mail events. Перед launch тестовые logs/submissions
следует очистить вместе с исследовательским content, но не в этой фазе.

## 12. Security и secrets

Status report выявил:

- Automatic Updates readiness error из-за Varbase patch plugins;
- Canvas warning: `justinrainbow/json-schema` + JSON:API + PHP assertions;
- Persistent Login требует session cookie lifetime `0`, сейчас `2000000`.

Production settings обязательны:

- verbose errors off, assertions off, no development services/Twig debug;
- exact `trusted_host_patterns` для `epe-m.ru` и выбранного canonical host;
- random non-empty hash salt из environment/secret file;
- private/temp paths вне web root;
- exact reverse-proxy settings только если proxy действительно есть;
- public files writable, code/settings non-writable web user;
- HTTPS redirect/HSTS только после полного TLS test;
- SecKit headers проверить без поломки Canvas/editor integrations;
- extension allowlists, MIME checks и upload limits сохранить строгими.

Secrets, которые не должны попадать в Git/config export: DB credentials, SMTP,
OAuth consumer secrets, Simple OAuth private keys, AI provider keys, CAPTCHA,
hash salt и encryption key material. На shared — environment variables или
permission-restricted file выше `public_html`; на VPS — systemd/environment
file или root-managed secret file. Backup secrets хранится отдельно и
зашифрованно, с документированным recovery access.

UID 1 использовать только break-glass. Ежедневная работа — именованные роли из
исследования №5; Super Admin минимизировать и audit-ить. Password Policy,
flood control, username enumeration prevention, SecKit и upload restrictions
уже входят в baseline, но production values требуют checklist review.

MFA для административных ролей желательно **до launch**, особенно если есть
production data/forms/API credentials. Mature варианты нужно отдельно сверить
на Drupal 11 (например TFA и WebAuthn ecosystem) и выбрать без установки в
этом исследовании. Если launch полностью brochure-only и admin ограничен одним
IP/VPN с strong unique credentials, MFA можно отложить в Phase 2, но это
принятый риск, не штатная возможность текущего baseline.

## 13. Backup/restore experiment и DR

### Что восстанавливается из Git/Composer

- project code, docs, DDEV descriptors;
- `composer.json`, `composer.lock`, patches lock;
- будущий versioned config export;
- generated core/contrib/vendor через `composer install`.

### Что обязательно backup-ить

- database;
- public files;
- private files;
- production config export/release SHA;
- secrets и OAuth keys отдельно, encrypted;
- mail/API operational settings, если они environment-only.

### Практический тест DDEV

1. `ddev export-db` создал gzip dump 4.6 МБ;
2. `tar` сохранил весь `web/sites/default/files`, включая текущий sync:
   71 МБ archive;
3. `system.site:slogan` изменён на `RESEARCH08_RESTORE_MARKER`;
4. `ddev import-db` успешно восстановил dump;
5. marker после restore отсутствовал — вернулось исходное пустое значение;
6. `drush cache:rebuild` успешен;
7. homepage вернул HTTP 200.

SHA-256 временных локальных artifacts:

```text
33545731404c6c3b9224d24a7ca2dc473535d0c11c76c4f46fbf7b590cfa03a8 database.sql.gz
72ce1d78874bb45f30c8cb129249f791c532da920b72ee195fcb0a26319c5650 files-and-config.tar.gz
```

Artifacts находятся только в WSL `/tmp/epe-master-research08-20260821` и не
попадают в Git. Restore public files отдельно не понадобился, поскольку
различимое изменение было в DB; archive был создан и checksum verified. Перед
production нужен второй test со специально изменённым runtime file.

Beget backup полезен как быстрый слой, но не единственный: VPS docs прямо
предупреждают, что filesystem backup не гарантирует корректный DB restore.
Рекомендуемая схема малого бизнеса:

- nightly DB dump + daily public/private files incremental;
- provider backup;
- encrypted offsite copy минимум ежедневно, 7 daily + 4 weekly + 3 monthly;
- quarterly restore drill и перед каждым release/update;
- config/Git/release SHA входит в recovery record.

Рекомендуемый launch target: **RPO 24 часа, RTO 4–8 часов**. RPO 1 час/RTO 1 час
требует чаще дампить DB, автоматизировать offsite restore, standby procedures и
оператора; для малого brochure/corporate сайта это пока несоразмерно. Forms с
ценными leads могут стать trigger для RPO 1 час.

## 14. Smoke tests и минимальный CI

Production smoke checklist:

- homepage и assets 200 over HTTPS;
- RU/EN URL, language switch и single-language material;
- Technical Article и Canvas Page/template;
- Search query и отсутствие draft/other-language leakage;
- contact/feedback submit, spam protection и mail delivery;
- editor login, admin login, workflow Draft→Review→Published;
- sitemap/canonical/hreflang;
- cron successful, queues not growing;
- JSON:API/OpenAPI/resources остаются closed/read-only по решению №7;
- status report без errors, recent logs без new PHP/mail/search failures.

Текущий репозиторий не содержит `.github/workflows`; есть Dependabot config,
issue/PR templates. Минимальный CI до launch:

1. `composer validate --no-check-publish`;
2. `composer audit --locked`;
3. clean `composer install --no-dev` и `check-platform-reqs`;
4. boot test на disposable DB + config import/status;
5. несколько HTTP smoke tests.

PHP code standards/static analysis не нужны, пока custom-code отсутствует.
Тяжёлый browser suite отложить; Varbase/upstream tests не заменяют project
config smoke test.

## 15. Production readiness checklist

| Requirement | Current state | Required before production | Optional after launch | Blocker |
| --- | --- | --- | --- | --- |
| Varbase release maturity | 11.0.0-rc1 | review newer release and RC risk | migrate to stable | да, решение риска |
| PHP | DDEV 8.4.22 | Beget web+CLI 8.4/extensions | patch cadence | да |
| DB | MariaDB 11.8 local | Beget MySQL 8, not 5.7 | managed DB | да |
| Composer lock build | lock present | clean no-dev build/platform check | artifact build CI | да |
| Dependency security | one advisory | resolve and re-audit | monthly batch | да |
| Config authority | ignored files/sync; import blocked by orphaned Tour config; Canvas drift | versioned sync, clean validation, convergent export/import | Config Split if justified | да |
| Secrets | DDEV local only | external production secrets | managed vault | да |
| Trusted hosts/errors | not production-configured | exact hosts, verbose off | CSP tuning | да |
| Private files | unset | create/test outside web root if forms/private media need it | remote storage | depends on launch scope |
| Cron | successful manually | PHP 8.4 cron every 5–15 min + monitoring | supervised workers | да |
| Queues | seven empty DB queues | verify after cron/smoke | worker daemon | нет |
| Cache | page/dynamic/aggregation work | production headers/smoke | Redis/CDN | нет |
| Images | ImageMagick/WebP fast locally | hosting trial and upload policy | AVIF/prewarm | нет |
| Mail | Mailpit local | authenticated SMTP/delivery test | provider failover | да для forms |
| Logging | dblog/syslog enabled | retention/error monitoring | external stack | да |
| MFA | absent | decide/install mature option or accept risk | Phase 2 only by explicit risk | рекомендуется blocker admin security |
| Backup | local restore passed | provider+offsite and production-like restore | shorter RPO | да |
| Rollback | documented | tested release+DB restore procedure | automation | да |
| Staging | absent | temporary clone before update; decide pre-launch | persistent staging | нет для initial brochure launch |
| CI | no workflows | minimal validate/audit/build/smoke | browser suite | да перед repeatable deploy |
| API | closed/read-only baseline | smoke confirm closed | OAuth/AI | нет |

Классификация:

- cache, revisions, config API, queues, cron contract — **Drupal core**;
- prepared feature set, ImageMagick/Drimage/Ultimate Cron/SecKit/audit —
  **Varbase штатно / Входит contrib**;
- PHP/DB/cron/permissions/secrets/backup — **Hosting/configuration**;
- Redis, CDN, Solr и persistent workers без measured need — **Лучше
  скорректировать требование**;
- custom deployment module/theme/code — не требуется; **Возможный custom** не
  подтверждён.

## 16. Решения, для которых позже нужен ADR

ADR в этом исследовании не создавались. Кандидаты на согласование:

1. Beget shared-first с явными VPS triggers;
2. authoritative versioned config directory и environment override policy;
3. release/deploy/rollback и backup RPO/RTO;
4. временный versus постоянный staging;
5. MFA policy для administrative roles.

## 17. Итог

Varbase 11 технически может работать как обычный Composer-managed Drupal 11 на
простом hosting, и текущей нагрузке не нужны Redis, CDN, Solr или Kubernetes.
Главный риск — не производительность, а воспроизводимость: config пока живёт в
ignored runtime directory, две working copies рассинхронизированы, dependency
audit не чист, production secrets/settings отсутствуют, а Varbase 11 — RC.

Поэтому естественная архитектура E+E Master: один Git source of truth,
Composer-from-lock, versioned config, Beget shared после qualification,
проверяемые backups и простой tagged release. VPS выбирается по конкретному
trigger, а не «на будущее». Разработка сайта после этого исследования не
начинается.
