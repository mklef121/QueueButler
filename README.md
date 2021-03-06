# QueueButler

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Software License][ico-license]](LICENSE.md)
[![Total Downloads][ico-downloads]][link-downloads]
[![Build Status][ico-travis]][link-travis]

Laravel Artisan commands that make it easy to run job queues using the Scheduler without the need for installing for running the Queue Daemon or installing Supervisor.

This is ideal for shared hosting or situations where you are not fully in control of the services or management of your hosting infrastructure and all you have access to is a Cron.

## Versions

__Confirmed to be working:__

* Laravel 5.3
* Laravel 5.4
* Laravel 5.5

## Install

__Via Composer__

``` bash
$ composer require web-chefs/queue-butler
```

__Add Service Provider to `config/app.php`__

```php
'providers' => [
   // Other Service Providers
   WebChefs\QueueButler\QueueButlerServiceProvider::class,
];
```

## Usage

### Artisan command

Run `queue:batch` artisan command, supports many of the same options as `queue:work`. Two additional options `time-limit` in seconds (defaults to 60 seconds) and 'job-limit' (defaults to 100) need to be set based on your Scheduling setup.

__Example:__

Run batch for 4 min 30 seconds or 1000 jobs, then stop.

`artisan queue:batch --time-limit=270 --job-limit=1000`

### Scheduler

In your `App\Console\Kernel.php` in the `schedule()` method add your `queue:batch` commands.

__Example:__

Run batch every minute for 50 seconds or 100 jobs in the background using `runInBackground()`, then stop.
Prevent overlapping batches running simultaneously with `withoutOverlapping()`.

``` php
$schedule->command('queue:batch --queue=default,something,somethingelse --time-limit=50 --job-limit=100')
         ->everyMinute()
         ->runInBackground()
         ->withoutOverlapping();
```
If your application is processing a large number of jobs for multiple queues, it is recommended setting up different batch scheduler commands per queue.

Because job queue processing is a long running process setting `runInBackground()` is highly recommended, else each `queue:batch` command will hold up all scheduled preceding items setup to run after it.

The Scheduler requires a __Cron__ to be setup. See [Laravel documentation](https://laravel.com/docs/master/scheduling) for details how the Scheduler works.

__`withoutOverlapping()` and Mutex cache expiry__

When using `withoutOverlapping()` a cache Mutex is used to keep track of running jobs. The default cache expiry is 1440 minutes (24 hours).

If your batch process is interrupted the scheduler will ignore the task for the time of the expiry and you will have no jobs processing for 24 hours. The only way to resolve this is to clear the cache or manually remove the batch processes cache entry.

To prevent long running cache expiries it is advised to match your cache cache expiry time with your task frequency.

```php
// Create Batch Job Queue Processor Task
$scheduledEvent = $schedule->command('queue:batch --queue=default,something,somethingelse --time-limit=55 --job-limit=100');

// Match cache expiry with frequency
// Set cache mutex expiry to One min (default is 1440)
$scheduledEvent->expiresAt = 1;
$scheduledEvent->everyMinute()
               ->withoutOverlapping()
               ->runInBackground();
```

## Standards

* psr-1
* psr-2
* psr-4

## Change log

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

All code submissions will only be evaluated and accepted as pull-requests. If you have any questions or find any bugs please feel free to open an issue.

## Credits

- [All Contributors][link-contributors]

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

[ico-version]: https://img.shields.io/packagist/v/web-chefs/queue-butler.svg?style=flat-square
[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/web-chefs/queue-butler.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/web-chefs/QueueButler/master.svg?style=flat-square

[link-packagist]: https://packagist.org/packages/web-chefs/queue-butler
[link-travis]: https://travis-ci.org/web-chefs/QueueButler
[link-downloads]: https://packagist.org/packages/web-chefs/queue-butler
[link-author]: https://github.com/JFossey
[link-contributors]: ../../contributors
