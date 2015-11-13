---
title: Forcing SSL links in the HTML & Forms Laravel Collective Package
date: "2015-11-12"
URL: "2015/11/forcing-ssl-links-html-forms-laravel-collective-package/"
---

If you ever find yourself in a situation where you have a Laravel application behind an SSL/TLS termination proxy, you may find that Laravel's SSL detection can work against you. In particular, I had an issue with the [HTML & Forms](http://laravelcollective.com/docs/5.1/html) package by Laravel Collective in which the application would not link to assets in a secure way.

In Laravel 4.x it was possible to override the `Request::secure()` method to make it return true when you were behind a terminator proxy. With the HTML & Forms package moved out of Laravel's core however and changes to Laravel, it seems the package calls out to Laravel's [UrlGenerator](http://laravel.com/api/5.1/Illuminate/Routing/UrlGenerator.html).

To change the way the package created links, I created a custom service provider for the package and passed it an instance of the `UrlGenerator` with `forceSchema('https')` called on it. You can see the result below.

```php
<?php namespace App\Providers;

use Collective\Html\HtmlBuilder;
use Collective\Html\FormBuilder;
use Collective\Html\HtmlServiceProvider as BaseHtmlServiceProvider;

class HtmlServiceProvider extends BaseHtmlServiceProvider
{
    protected function registerHtmlBuilder()
    {
        $this->app->singleton('html', function($app) {
            // Fetch the default UrlGenerator
            $url = $app['url'];
            if (!$this->app->environment('local'))
            {
                $url = $app->make('Illuminate\Routing\UrlGenerator');
                $url->forceSchema('https');
            }

            return new HtmlBuilder($url);
        });
    }

    protected function registerFormBuilder()
    {
        $this->app->singleton('form', function($app) {
            $url = $app['url'];
            if (!$this->app->environment('local'))
            {
                $url = $app->make('Illuminate\Routing\UrlGenerator');
                $url->forceSchema('https');
            }

            $form = new FormBuilder($app['html'], $url, $app['session.store']->getToken());

            return $form->setSessionStore($app['session.store']);
        });
    }
}
```
