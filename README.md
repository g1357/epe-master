# E+E Master

Новый корпоративный и экспертный сайт компании E+E Master (`epe-m.ru`) на
Drupal 11 и Varbase 11.

Репозиторий: <https://github.com/g1357/epe-master>

## Статус

Фаза 1: исследование чистой платформы Varbase 11. Бизнес-функциональность,
собственные модули и собственная тема пока не разрабатываются.

Текущий baseline:

- Varbase project template 11.0.6;
- Varbase 11.0.0-rc1;
- Drupal 11.4.5;
- PHP 8.4;
- Drush 13.7.6;
- DDEV 1.25.3;
- MariaDB 11.8;
- Composer 2;
- Node.js 20 и Yarn 4 (штатный front-end toolchain Varbase).

Точные версии PHP, Composer, Node.js и Yarn следует сверять командами из
[руководства по локальной разработке](docs/development/local-environment.md),
поскольку часть инструментов поставляется DDEV-контейнером.

## Быстрый старт

Требования: WSL2, Docker CE/Docker Desktop и DDEV.

```bash
git clone https://github.com/g1357/epe-master.git
cd epe-master
ddev start
ddev install-varbase full
```

После уже выполненной установки достаточно `ddev start`. Локальный адрес:
<http://ee-master.ddev.site>.

## Принцип разработки

> Platform first, custom last.

Сначала используются штатные возможности Varbase 11, Drupal core и уже
входящие зрелые contrib-модули. Дополнительный contrib или custom-код допустим
только после документированного подтверждения ограничения платформы.

## Документация

- [Архитектура](docs/architecture/README.md)
- [ADR-0001: Varbase 11 и platform first](docs/adr/ADR-0001-varbase-11-platform-first.md)
- [Локальная разработка](docs/development/local-environment.md)
- [Принципы и требования](docs/requirements/project-principles.md)
- [Карта возможностей Varbase](docs/research/varbase-capability-map.md)
- [План исследования](docs/research/plan.md)

## Структура

```text
.ddev/          Штатная DDEV-конфигурация и команды Varbase
.github/        Шаблоны GitHub
docs/           Архитектура, ADR, требования и исследования
recipes/        Composer-managed Drupal/Varbase recipes
web/            Drupal document root
composer.json   Корневые PHP-зависимости
composer.lock   Зафиксированные PHP-версии
```

`vendor`, contrib-код, runtime-файлы и секреты не хранятся в Git и
восстанавливаются штатными инструментами.
