# Migration from version 1

This documents details changes made from version 1 of this package or from its implementation in `laminas-mvc`.

## PSR-15 Middleware

Only PSR-15 style middleware can be dispatched anymore, so all middleware has to implement
`\Psr\Http\Server\MiddlewareInterface` or `\Psr\Http\Server\RequestHandlerInterface` 

## Removed Route match params from request
Previously, all matched route params were added to the `ServerRequestInterface` as attributes. This is no longer the case.
If you need route params, you can extract them from the `\Laminas\Router\RouteMatch` request attribute which is still available.
