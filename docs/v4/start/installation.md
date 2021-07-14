---
title: Installation
---

## System Requirements

* Web server with URL rewriting
* PHP 7.2 or newer

## Step 1: Install Composer

Don't have Composer? It's easy to install by following the instructions on their [download](https://getcomposer.org/download/) page.

## Step 2: Install Slim

We recommend you install Slim with [Composer](https://getcomposer.org/).
Navigate into your project's root directory and execute the bash command
shown below. This command downloads the Slim Framework and its third-party
dependencies into your project's `vendor/` directory.

```bash
composer require slim/slim:"4.*"
```

## Step 3: Install a PSR-7 Implementation and ServerRequest Creator

Before you can get up and running with Slim you will need to choose a PSR-7 implementation that best fits your application.
In order for auto-detection to work and enable you to use `AppFactory::create()` and `App::run()` without having to manually create a `ServerRequest` you need to install one of the following implementations:

### [Slim PSR-7](https://github.com/slimphp/Slim-Psr7)
```bash
composer require slim/psr7
```

### [Nyholm PSR-7](https://github.com/Nyholm/psr7) and [Nyholm PSR-7 Server](https://github.com/Nyholm/psr7-server)
```bash
composer require nyholm/psr7 nyholm/psr7-server
```

### [Guzzle PSR-7](https://github.com/guzzle/psr7) and [Guzzle HTTP Factory](https://github.com/http-interop/http-factory-guzzle)
```bash
composer require guzzlehttp/psr7 1.4.2 http-interop/http-factory-guzzle
```

### [Laminas Diactoros](https://github.com/laminas/laminas-diactoros)
```bash
composer require laminas/laminas-diactoros
```

## Step 4: Hello World

File: `public/index.php`

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function (Request $request, Response $response, $args) {
    $response->getBody()->write("Hello world!");
    return $response;
});

$app->run();
```
