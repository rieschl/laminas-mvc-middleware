# Introduction

This library provides the ability to dispatch middleware pipelines in place of
controllers within laminas-mvc.

## Dispatching PSR-7 Middleware

[PSR-7](http://www.php-fig.org/psr/psr-7/) defines interfaces for HTTP messages,
and is now being adopted by many frameworks; Laminas itself offers a
parallel microframework targeting PSR-7 with [Mezzio](https://docs.mezzio.dev/mezzio).
What if you want to dispatch PSR-7 middleware from laminas-mvc?

laminas-mvc currently uses [laminas-http](https://github.com/laminas/laminas-http)
for its HTTP transport layer, and the objects it defines are not compatible with
PSR-7, meaning the basic MVC layer does not and cannot make use of PSR-7
currently.

However, starting with version 2.7.0, laminas-mvc offers
`Laminas\Mvc\MiddlewareListener`. This `Laminas\Mvc\MvcEvent::EVENT_DISPATCH`
listener listens prior to the default `DispatchListener`, and executes if the
route matches contain a "middleware" parameter, and the service that resolves to
is callable. When those conditions are met, it uses the [PSR-7 bridge](https://github.com/laminas/laminas-psr7bridge)
to convert the laminas-http request and response objects into PSR-7 instances, and
then invokes the middleware.

Starting with laminas-mvc version 3.2.0, `Laminas\Mvc\MiddlewareListener` is deprecated and replaced
by `Laminas\Mvc\Middleware\MiddlewareListener` provided by this package.  
After package installation, `Laminas\Mvc\Middleware` module must be registered in your
laminas-mvc based application.
If the [`laminas-component-installer`](https://docs.laminas.dev/laminas-component-installer/)
is installed, it will handle the module registration automatically.

## Mapping routes to Middleware

The first step is to map a route to PSR-7 middleware. This looks like any other
[routing](https://docs.laminas.dev/laminas-mvc/routing/) configuration,
with one small change: instead of providing a `controller` in the routing
defaults, you provide `middleware`:

```php
// Via configuration:
use Application\Middleware\SomeMiddleware;
use Laminas\Router\Http\Literal;

return [
    'router' => [
        'routes' => [
            'home' => [
                'type' => Literal::class,
                'options' => [
                    'route' => '/',
                    'defaults' => [
                        'middleware' => SomeMiddleware::class,
                    ],
                ],
            ],
        ],
    ],
];
```

Middleware may be provided as instance of a PSR-15 `\Psr\Http\Server\MiddlewareInterface` or `\Psr\Http\Server\RequestHandlerInterface`
or as service name strings resolving to such instances.  
You may also specify an `array` of above middleware types. These will then be piped
into a `Laminas\Stratigility\MiddlewarePipe` instance in the order in which they
are present in the array.

> ### No action required
>
> Unlike action controllers, middleware typically is single purpose, and, as
> such, does not require a default `action` parameter.

## Middleware services

Middleware are pulled from the application service manager, unlike controllers in a normal laminas-mvc dispatch cycle,
which are pulled from a dedicated `ControllerManager`.

Middleware retrieved **must** be a PSR-15 `MiddlewareInterface` or `RequestHandlerInterface` instance.
Otherwise, `MiddlewareListener` will create an error response.

## Writing Middleware

### `MiddlewareInterface` vs. `RequestHandlerInterface`

_Middleware_ is code sitting between a request and a response; it typically analyzes the request to aggregate incoming data, delegates it to another layer to process, and then creates and returns a response.

A _RequestHandler_ is a class that receives a request and returns a response, without delegating to other layers of the application. This is generally the inner-most layer of your application.

For more in-depth documentation visit the documentation for [Mezzio](https://docs.mezzio.dev/mezzio/v3/getting-started/features/)
and [Stratigility](https://docs.laminas.dev/laminas-stratigility/v3/intro/) or the [PSR specification](https://www.php-fig.org/psr/psr-15/).

### Request Handlers

```php
namespace Application\Handler;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;

class SomeHandler implements RequestHandlerInterface
{
    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        // do some work
    }
}
```

RequestHandlers resemble a MVC Controller action, so mostly a `ReqeustHandler` will be used to dispatch a request.

### Middleware

```php
namespace Application\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class SomeMiddleware implements MiddlewareInterface
{
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // do some work
        // ...
        return $handler->handle($request);
    }
}
```

Middleware can return a direct response, in effect short-circuiting the middleware pipe, or pass request further
while having a chance to act on passed request or returned response.
Middleware in laminas-mvc is similar to [routed middleware](https://docs.mezzio.dev/mezzio/v3/features/router/piping/#routing)
in Mezzio. Laminas-mvc does not have a global middleware pipes, so middleware can not be piped in front of MVC controllers.

## Middleware return values

As middleware returns a PSR-7 response (`\Psr\Http\Message\ResponseInterface`) it is converted back to a
laminas-http response and returned by the `MiddlewareListener`, causing the application to
short-circuit and return the response immediately.
