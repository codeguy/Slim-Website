---
title: Middleware
---

## What is middleware?

A middleware is a _callable_ that accepts three arguments: 

1. `\Psr\Http\Message\RequestInterface` - The request object
2. `\Psr\Http\Message\ResponseInterface` - The response object
3. `callable` - The next middleware callable
 
It can do whatever is appropriate with these objects. The only hard requirement is that a middleware **MUST** return an instance of `\Psr\Http\Message\ResponseInterface`. Each middleware **MAY** invoke the next middleware and pass it Request and Response objects as arguments.

## How does middleware work?

Different frameworks use middleware differently. Slim adds middleware as concentric layers surrounding your core application. Each new middleware layer surrounds any existing middleware layers. The concentric structure expands outwardly as additional middleware layers are added.

When you run the Slim application, the Request and Response objects traverse the middleware structure from the outside in. They first enter the outer-most middleware, then the next outer-most middleware, (and so on), until they ultimately arrive at the Slim application itself. After the Slim application dispatches the appropriate route, the resultant Response object exits the Slim application and traverses the middleware structure from the inside out. Ultimately, a final Response object exits the outer-most middleware, is serialized into a raw HTTP response, and is returned to the HTTP client. Here's a diagram that hopefully illustrates the middleware process flow:

<div style="padding: 2em 0; text-align: center">
    <img src="/docs/images/middleware.png" alt="Middleware architecture" style="max-width: 80%;"/>
</div>

## How do I write middleware?

Middleware is a callable that accepts three arguments: a Request object, a Response object, and the next middleware. Each middleware **MUST** return an instance of `\Psr\Http\Message\ResponseInterface`. 

### Closure middleware example.

This example middleware is a Closure.

{% highlight php %}
<?php
function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};
{% endhighlight %}

### Invokable class middleware example

This example middleware is an invokable class that implements the magic `__invoke()` method.

{% highlight php %}
<?php
class ExampleMiddleware
{
    public function __invoke($request, $response, $next)
    {
        $response->getBody()->write('BEFORE');
        $response = $next($request, $response);
        $response->getBody()->write('AFTER');

        return $response;
    }
}
{% endhighlight %}

To use this class as a middleware, please use `->add( new ExampleMiddleware() );` function chain after the `$app`, `Route`,  or `group()`, which in the code below, any one of these, could represent $subject.

{% highlight php %}
$subject->add( new ExampleMiddleware() );
{% endhighlight %}

## How do I add middleware?

You may add middleware to a Slim application or to an individual Slim application route. Both scenarios accept the same middleware and implement the same middleware interface.

### Application middleware

Application middleware is invoked for every *incoming* HTTP request. Add application middleware with the Slim application instance's `add()` method. This example adds the Closure middleware example above:

{% highlight php %}
<?php
$app = new \Slim\App();

$app->add(function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER'); 

    return $response;
});

$app->get('/', function ($req, $res, $args) {
    echo ' Hello ';
});

$app->run();
{% endhighlight %}

This would output this HTTP response body:

    BEFORE Hello AFTER

### Route middleware

Route middleware is invoked _only if_ its route matches the current HTTP request method and URI. Route middleware is specified immediately after you invoke any of the Slim application's routing methods (e.g., `get()` or `post()`). Each routing method returns an instance of `\Slim\Route`, and this class provides the same middleware interface as the Slim application instance. Add middleware to a Route with the Route instance's `add()` method. This example adds the Closure middleware example above:

{% highlight php %}
<?php
$app = new \Slim\App();

$mw = function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
});

$app->get('/', function ($req, $res, $args) {
    echo ' Hello ';
})->add($mw);

$app->run();
{% endhighlight %}

This would output this HTTP response body:

    BEFORE Hello AFTER

### Group Middleware

In addition to the overall application, and standard routes being able to accept middleware, the `group()` multi-route definition functionality, also allows individual routes internally. Route group middleware is invoked _only if_ its route matches one of the defined HTTP request methods and URIs from the group. To add middleware within the callback, and entire-group middleware to be set by chaining `add()` after the `group()` method.

Sample Application, making use of callback middleware on a group of url-handlers 
{% highlight php %}
<?php

require_once __DIR__.'/vendor/autoload.php';

$app = new \Slim\App();

$app->get('/', function ($request, $response) {
    return $response->write('Hello World');
});

$app->group('/utils', function () use ($app) {
    $app->get('/date', function ($request, $response) {
        return $response->write(date('Y-m-d H:i:s'));
    });
    $app->get('/time', function ($request, $response) {
        return $response->write(time());
    });
})->add(function ($request, $response, $next) {
    $response->write('It is now ');
    $response = $next($request, $response);
    $response->write('. Enjoy!');

    return $response;
});
{% endhighlight %}

When calling the `/utils/date` method, this would output a string similar to the below

    It is now 2015-07-06 03:11:01. Enjoy!

visiting `/utils/time` would output a string similar to the below

    It is now 1436148762. Enjoy!

but visiting `/` *(domain-root)*, would be expected to generate the following output as no middleware has been assigned

    Hello World

Obviously there are more useful examples of middleware and both class-based, and simple anonymous function-based middleware, are also supported at the application, route and group level. The reasons for each will be a mixture of personal preference, and utility / code quality.
