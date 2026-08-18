# Baseline installation log

- Дата: 2026-08-18
- Режим: Varbase `full`, исследовательский стенд
- Результат: успешно

## Основные команды

Официальная последовательность:

```bash
ddev config --project-type=drupal11 --docroot=web --php-version=8.4
ddev start
ddev composer create-project "drupal/varbase_project:~11"
ddev install-varbase full
ddev drush language:add ru -y
ddev drush cache:rebuild
```

Поскольку репозиторий уже содержал документацию, Composer template сначала был
создан во временном каталоге и затем объединён с корнем без изменения
upstream-кода. Из-за низкой скорости `/mnt/c` фактическая сборка Composer была
выполнена в нативном временном каталоге контейнера, после чего baseline перенесён
в репозиторий.

Для рабочего запуска создана WSL-native копия:

```bash
mkdir -p /home/dukar/projects/epe-master
tar -C /mnt/c/Users/Dukar/Documents/ChatGPT/Drupal -cf - . \
  | tar -C /home/dukar/projects/epe-master -xf -
cd /home/dukar/projects/epe-master
ddev start
ddev install-varbase full
ddev drush language:add ru -y
ddev exec yarn install
ddev restart
```

## Проверенные версии

| Компонент | Версия |
| --- | --- |
| Varbase project template | 11.0.6 |
| Varbase | 11.0.0-rc1 |
| Drupal | 11.4.5 |
| PHP | 8.4.22 |
| Composer | 2.10.2 |
| Drush | 13.7.6 |
| DDEV | 1.25.3 |
| MariaDB | 11.8.8 |
| Node.js | 20.20.2 |
| Yarn | 4.18.0 |

## Проверки

- Drupal bootstrap: `Successful`.
- Database status: `Connected`.
- HTTP с Windows: `200`.
- Front-end theme: `vartheme_bs5`.
- Admin theme: `gin`.
- Default language: English (`en`).
- Added language: Russian (`ru`).
- Storybook daemon после `yarn install`: запускается без DDEV spawn error.

## Проблемы и наблюдения

1. Composer patch download один раз получил HTTP 429 от
   `raw.githubusercontent.com`; повтор использовал cache и завершился.
2. Upstream post-create script потребовал Corepack/Yarn 4; это предусмотрено
   штатной DDEV-конфигурацией Varbase.
3. `/mnt/c` оказался крайне медленным для Composer и Mutagen initial sync.
   WSL-native copy сократила операции установки с десятков минут до минут.
4. Штатный full AI recipe без OpenAI key автоматически получил credentials
   анонимного amazee.ai trial account. Не использовать с реальными данными.
5. Для части contrib-модулей русские переводы отсутствуют.
6. DDEV сообщает о проблеме доверия локальному mkcert CA; HTTP работает, а HTTPS
   trust следует исправить отдельно через `ddev utility tls-diagnose`.
7. Composer сообщает, что `oomphinc/composer-installers-extender` abandoned;
   пакет приходит из upstream lock и должен отслеживаться в supply-chain audit.
8. Drupal status report после чистой full-установки содержит три ошибки baseline,
   которые намеренно не исправлялись без исследования:
   - Automatic Updates не принимает плагины `vardot/varbase-patches` и
     `vardot/drupal-core-patches` как поддерживаемые;
   - Canvas предупреждает о влиянии `justinrainbow/json-schema` на performance
     при одновременно включённых JSON:API и PHP assertions;
   - Persistent Login требует session cookie lifetime `0`, тогда как baseline
     сообщает `2000000`.
