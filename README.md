# Atlassian Connect Core

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Software License][ico-license]](LICENSE.md)
[![Build Status][ico-travis]][link-travis]
[![Quality Score][ico-code-quality]][link-code-quality]
[![Total Downloads][ico-downloads]][link-downloads]

Atlassian Connect Framework to build add-ons for the JIRA and Confluence

## Requirements

* Laravel 5.5
* PHP ~7.0

## Getting Started

### Installing

Install dependency via Composer

``` bash
$ composer require brezzhnev/atlassian-connect-core
```

Register **route middleware** `jwt` by adding to the `app\Http\Kernel.php` following line

``` php
'jwt' => \AtlassianConnectCore\Http\Middleware\JWTAuth::class
```

Set authentication driver to `jwt` in `config/auth.php`

``` php
'guards' => [
    'web' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
...
```

Set model class in `config/auth.php` **providers** section

``` php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => \AtlassianConnectCore\Models\Tenant::class,
    ],
...
```

Register plugin events by adding to the `app/Providers/EventServiceProvider.php` to the `listen` property value following

``` php
\AtlassianConnectCore\Events\Installed::class => [
    \AtlassianConnectCore\Listeners\CreateOrUpdateTenant::class
],
\AtlassianConnectCore\Events\Uninstalled::class => [
    \AtlassianConnectCore\Listeners\DeleteTenant::class
]
```

> You can use any add-on events to register your own listeners

Register subscriber in the same file:

```
/**
 * The subscriber classes to register.
 *
 * @var array
 */
protected $subscribe = [
    \AtlassianConnectCore\Listeners\PluginEventSubscriber::class
];
```

Configure database and run

```
php artisan migrate
php artisan plugin:install
```

Command `php artisan plugin:install` will publish config, views and resources that you can change whatever you want.

Also, it will create "dummy" tenant needed for local testing and development 
without the need of installing the add-on in real JIRA or Confluence instance.

### Get it working

If your application returns the add-on descriptor on the request 
to URL `http://localhost:8000/atlassian-connect.json` it means you are close to getting it working and you can
install the add-on.

Firstly, you need to enable development options. 
Go to the "Manage add-ons" page. You'll see the link "Settings" at bottom of page. 
After clicking you'll see two checkboxes that must be selected. Apply your changes.

For installing the add-on in the instance it should see your server.
If you working locally the easiest way is use [ngrok](https://ngrok.com/).

After you are visible for the JIRA or Atlassian instance you should put your actual website URL to environment variable `PLUGIN_URL`. Also, you need to configure your add-on by editing the file `config/plugin.php`. Most values may be overwritten using env vars. 

Then you need to upload the add-on. Click "Upload add-on" and paste your public URL with descriptor path, 
eg. `https://d1ea31ce.ngrok.io/atlassian-connect.json` or `https://yourplugindomain.com/atlassian-connect.json`

> Note: HTTPS is required

After successfully installing you can see "Your add-on" top menu item (in case of JIRA). 
You also can go to the add-on general page by direct link `:product_base_url/plugins/servlet/ac/sample-plugin/hello-page`

Instead `:product_base_url` you should put your JIRA or Cofluence instance URL (eg. `https://google-dev.atlassian.net`).

If you see page working, the application configured and add-on installed correctly.

### Publish resources

Instead of using `plugin:install` you can perform actions manually.

To copy all publishes you should use following commands:

```
artisan vendor:publish --provider="AtlassianConnectCore\ServiceProvider"
```

To copy only specific publish you must call this command with option `--tag`. 
Value can be `public` (to copy assets), `views` and `config`.

## Workflow

### Add-On Configuration

After copying publishes you can see the file `config/plugin.php` in your application. Please, use this configuration file to change add-on properties.

### Default routes

The following routes registered by default

* `GET /atlassian-connect.json` descriptor contents
* `POST /installed` add-on installed callback
* `POST /uninstalled` add-on uninstalled callback
* `POST /enabled` add-on enabled callback
* `POST /disabled` add-on disabled callback
* `GET /hello` sample page to persuade all working correctly

You can disable it by setting to `false` config value `plugin.loadRoutes`.

### Descriptor

You can use `Descriptor` facade to customize or create from scratch your own descriptor contents.

For example, you can customize it by adding to the `app\Providers\AppServiceProvider` in `boot` section the following:

``` php
Descriptor::base() // base descriptor contents
    ->setScopes(['admin' , 'act_as_user'])
    ->withModules([
        'webhooks' => [[
            'event' => 'jira:issue_created',
            'url' => route('webhookHandlerRouteName')
        ]]
    ])
    ->set('version', $this->getLatestPluginVersion());
```

> Warning: if you are using `route` helper in the `AppServiceProvider` you should have `RouteServiceProvider` defined above `AppServiceProvider` in your `app.php` config.

### Performing requests

In most of cases in development add-on for Atlassian Product you need to perform requests to the instance. 

For this case you should use `JWTClient`. It uses [GuzzleHttp](https://github.com/guzzle/guzzle) as HTTP client and 
if you want to have custom handling (middlewares etc.) you can pass client instance to the constructor.

#### Pagination

If you want to send a request to an endpoint with pagination you should use `JWTClient::paginate` method. In most cases
you don't need to pass paginator instance to the `JWTClient` constructor because it will instantiate automatically by resolving 
your Tenant product type (JIRA or Confluence), but you always can use specific paginator.

There are two paginators:
* `JiraPaginator`
* `ConfluencePaginator`

Of course you can extend `Paginator` class and create your own.

### Console commands

* `plugin:install` is a helper command that creates "dummy" tenant with fake data and publishes package resources (config, views, assets)
* `plugin:dummy` provides interactive way to set tenant as "dummy" without manually editing database

## Tests

Run the following in the package folder:

```
vendor/bin/phpunit
```

## TODO

* Implement descriptor builder and validator
* Implement webhooks manager
* Take out pagination and make more abstract

## Security

If you discover any security related issues, please email **brezzhnev@gmail.com** instead of using the issue tracker.

## Credits

- [brezzhnev][link-author]
- [All Contributors][link-contributors]

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

[ico-version]: https://img.shields.io/packagist/v/brezzhnev/atlassian-connect-core.svg?style=flat-square
[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/brezzhnev/atlassian-connect-core/master.svg?style=flat-square
[ico-code-quality]: https://img.shields.io/scrutinizer/g/brezzhnev/atlassian-connect-core.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/brezzhnev/atlassian-connect-core.svg?style=flat-square

[link-packagist]: https://packagist.org/packages/brezzhnev/atlassian-connect-core
[link-travis]: https://travis-ci.org/brezzhnev/atlassian-connect-core
[link-scrutinizer]: https://scrutinizer-ci.com/g/brezzhnev/atlassian-connect-core/code-structure
[link-code-quality]: https://scrutinizer-ci.com/g/brezzhnev/atlassian-connect-core
[link-downloads]: https://packagist.org/packages/brezzhnev/atlassian-connect-core
[link-author]: https://github.com/brezzhnev
[link-contributors]: ../../contributors
