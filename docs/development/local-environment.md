# Локальная среда

## Рекомендуемое расположение

DDEV рекомендует хранить проект в нативной файловой системе WSL2, например:

```text
/home/dukar/projects/epe-master
```

Работа из `/mnt/c/...` поддерживается, но Composer, Drupal и Node выполняют
много мелких файловых операций и работают значительно медленнее. Временный
локальный `config.performance.yaml` может включать Mutagen, однако этот файл не
коммитится и не заменяет WSL-native clone.

## Конфигурация

- project: `ee-master`;
- type: `drupal11`;
- docroot: `web`;
- PHP: 8.4;
- web server: Apache FPM (штатный Varbase template);
- database: MariaDB 11.8;
- Composer: 2;
- Node.js: 20;
- дополнительный Storybook endpoint: `storybook.ee-master.ddev.site`.

## Установка с нуля

```bash
git clone https://github.com/g1357/epe-master.git
cd epe-master
ddev start
ddev composer install
ddev install-varbase full
```

`full` выбран для исследовательского стенда. Он не является заранее
утверждённым production-составом.

## Повседневная работа

```bash
ddev start
ddev describe
ddev drush status
ddev stop
```

## Проверка версий

```bash
ddev version
ddev php --version
ddev composer --version
ddev drush --version
ddev exec node --version
ddev exec yarn --version
ddev composer show vardot/varbase --locked
ddev composer show drupal/core --locked
ddev composer show drush/drush --locked
```

Не коммитить database dumps, `settings.php`, пользовательские файлы, секреты,
`vendor`, `node_modules` и локальные DDEV overrides.
