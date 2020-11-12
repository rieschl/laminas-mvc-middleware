# Migration from version 1

This documents details changes made from version 1 of this package or from its implementation in laminas-mvc.
If installed, it overrides the default behavior in laminas-mvc.

## PSR-15 Middleware

This package now supports only PSR-15 interfaces. Support of `http-interop/http-middleware` and `callable` middlerware
has been dropped.

Please also refer to [Laminas-stratigility v3 migration guide](https://docs.laminas.dev/laminas-stratigility/v3/migration/)
if you used any of its features.

## Removed Route match params from request
Previously, for the matched route both `Laminas\Router\RouteMatch` object as well as individual matched params were added
to PSR-7 request as attributes. This is no longer the case and only the `RouteMatch` object can be accessed
as PSR-7 request attribute: `$request->getAttribute(\Laminas\Router\RouteMatch::class);`
