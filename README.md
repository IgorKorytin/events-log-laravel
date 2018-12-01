<p align="center">
  <img src="https://laravel.com/assets/img/components/logo-laravel.svg" alt="Laravel" width="240" />
</p>

# Логгирование событий в Laravel-приложениях

[![Version][badge_packagist_version]][link_packagist]
[![Version][badge_php_version]][link_packagist]
[![Build Status][badge_build_status]][link_build_status]
[![Coverage][badge_coverage]][link_coverage]
[![Code quality][badge_code_quality]][link_code_quality]
[![Downloads count][badge_downloads_count]][link_packagist]
[![License][badge_license]][link_license]

Данный пакет позволяет удобно использовать в вашем Laravel приложении функционал логгирования событий, которые реализуют определенный интерфейс.

## Install

Require this package with composer using the following command (`laravel/framework` version 5.6 and above is required):

```shell
$ composer require avto-dev/events-log-laravel "^1.1"
```

> Installed `composer` is required ([how to install composer][getcomposer]).

> You need to fix the major version of package.

Сервис-провайдер будет зарегистрирован автоматически. В противном случае выполните:

```shell
$ php artisan package:discover
```

## Настройка

После установки пакета вам необходимо произвести его настройку. Минимальной конфигурацией является добавление в ваш файл `./config/logging.php` значения:

```php
<?php

return [
    // ...
    
    'events_channel' => env('EVENTS_LOG_CHANNEL', 'stack'),
    
    // ...
];
```

Где `stack` - это имя одного из каналов, перечисленного в секции `channels` этого же файла. Без указания данной опции логгирование будет производиться с использованием канала по умолчанию.

> Переопределить данную опцию вы сможете добавив в `.env` файл вашего приложения строку `EVENTS_LOG_CHANNEL=%channel_name%`.

Например, если вам необходимо производить логгирование событий в отдельный файл в формате `Monolog` и дополнительно вести запись в **другой файл** в формате `Logstash`, то конфигурация может иметь следующий вид:

```php
<?php

return [

    'events_channel' => env('EVENTS_LOG_CHANNEL', 'events-stack'),

    // ...    

    'channels' => [
    
        // ...

        'events-stack' => [
            'driver'   => 'stack',
            'channels' => ['events-monolog', 'events-logstash'],
        ],

        'events-monolog' => [
            'driver' => 'single',
            'path'   => storage_path('logs/laravel-events.log'),
            'level'  => 'debug',
        ],

        'events-logstash' => [
            'driver' => 'custom',
            'via'    => AvtoDev\EventsLogLaravel\Logging\EventsLogstashLogger::class,
            'path'   => storage_path('logs/logstash/laravel-events.log'),
            'level'  => 'debug',
        ],
    ],
];
```

Более подробно о настройке логгирования вы можете прочитать по [этой ссылке][laravel_logging].


## Использование

Данный пакет работает следующий образом:

- Сервис-провайдер данного пакета регистрирует свой "слушатель" на все события, что происходят в приложении;
- При получении события он проверяет реализацию класса события интерфейса `ShouldBeLoggedContract`;
- Если предыдущее условие выполняется - то используя указанный в файле `logging.php` канал логгирования производится запись данных которые возвращают методы, описанные в интерфейсе `ShouldBeLoggedContract`.

Пример класса логгируемого события:

```php
<?php

class SomeApplicationEvent implements \AvtoDev\EventsLogLaravel\Contracts\ShouldBeLoggedContract
{
    /**
     * {@inheritdoc}
     */
    public function logLevel(): string
    {
        return 'info';
    }

    /**
     * {@inheritdoc}
     */
    public function logMessage(): string
    {
        return 'My log message';
    }

    /**
     * {@inheritdoc}
     */
    public function logEventExtraData(): array
    {
        return ['key' => 'any value'];
    }

    /**
     * {@inheritdoc}
     */
    public function eventType(): string
    {
        return 'default_event';
    }

    /**
     * {@inheritdoc}
     */
    public function eventSource(): string
    {
        return 'service_name';
    }
}
```

Теперь достаточно в произвольном месте вашего приложения вызвать:

```php
event(new SomeApplicationEvent);
```

И быть уверенным в том, что данное событие будет записано в лог-файл. О том, как работают события (events) в Laravel вы можете прочитать по [этой ссылке][laravel_events].

### Дополнительные логгеры

Вместе с данным пакетом вам доступны следующие пред-настроенные логгеры `AvtoDev\EventsLogLaravel\Logging\...`:

Класс логгера | Назначение
---------------- | ----------
`DefaultLogstashLogger` | Пишет лог-записи в формате `logstash`, не видоизменяя тело записи (поле `context` не изменяется)
`EventsLogstashLogger` | Пишет лог-записи в формате `logstash`, но данные связанные с событиями помещаются в секцию `event`

> Более подробно о них смотрите исходный код

### Testing

For package testing we use `phpunit` framework. Just write into your terminal:

```shell
$ git clone git@github.com:avto-dev/events-log-laravel.git ./events-log-laravel && cd $_
$ composer install
$ composer test
```

## Changes log

[![Release date][badge_release_date]][link_releases]
[![Commits since latest release][badge_commits_since_release]][link_commits]

Changes log can be [found here][link_changes_log].

## Support

[![Issues][badge_issues]][link_issues]
[![Issues][badge_pulls]][link_pulls]

If you will find any package errors, please, [make an issue][link_create_issue] in current repository.

## License

This is open-sourced software licensed under the [MIT License][link_license].

[badge_packagist_version]:https://img.shields.io/packagist/v/avto-dev/events-log-laravel.svg?maxAge=180
[badge_php_version]:https://img.shields.io/packagist/php-v/avto-dev/events-log-laravel.svg?longCache=true
[badge_build_status]:https://travis-ci.org/avto-dev/events-log-laravel.svg?branch=master
[badge_code_quality]:https://img.shields.io/scrutinizer/g/avto-dev/events-log-laravel.svg?maxAge=180
[badge_coverage]:https://img.shields.io/codecov/c/github/avto-dev/events-log-laravel/master.svg?maxAge=60
[badge_downloads_count]:https://img.shields.io/packagist/dt/avto-dev/events-log-laravel.svg?maxAge=180
[badge_license]:https://img.shields.io/packagist/l/avto-dev/events-log-laravel.svg?longCache=true
[badge_release_date]:https://img.shields.io/github/release-date/avto-dev/events-log-laravel.svg?style=flat-square&maxAge=180
[badge_commits_since_release]:https://img.shields.io/github/commits-since/avto-dev/events-log-laravel/latest.svg?style=flat-square&maxAge=180
[badge_issues]:https://img.shields.io/github/issues/avto-dev/events-log-laravel.svg?style=flat-square&maxAge=180
[badge_pulls]:https://img.shields.io/github/issues-pr/avto-dev/events-log-laravel.svg?style=flat-square&maxAge=180
[link_releases]:https://github.com/avto-dev/events-log-laravel/releases
[link_packagist]:https://packagist.org/packages/avto-dev/events-log-laravel
[link_build_status]:https://travis-ci.org/avto-dev/events-log-laravel
[link_coverage]:https://codecov.io/gh/avto-dev/events-log-laravel/
[link_changes_log]:https://github.com/avto-dev/events-log-laravel/blob/master/CHANGELOG.md
[link_code_quality]:https://scrutinizer-ci.com/g/avto-dev/events-log-laravel/
[link_issues]:https://github.com/avto-dev/events-log-laravel/issues
[link_create_issue]:https://github.com/avto-dev/events-log-laravel/issues/new/choose
[link_commits]:https://github.com/avto-dev/events-log-laravel/commits
[link_pulls]:https://github.com/avto-dev/events-log-laravel/pulls
[link_license]:https://github.com/avto-dev/events-log-laravel/blob/master/LICENSE
[laravel_logging]:https://laravel.com/docs/5.6/logging
[laravel_events]:https://laravel.com/docs/5.6/events
[getcomposer]:https://getcomposer.org/download/
