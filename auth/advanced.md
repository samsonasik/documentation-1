Advanced Authentication and Authorization
=========================================

## Modules & Components

Authentication is provided via the `Laminas\Authentication` component, authorization via the
`Laminas\Permissions` components, and MVC bindings are provided via the
[api-tools-mvc-auth](https://github.com/laminas-api-tools/api-tools-mvc-auth) module. This module's purpose is
to deliver a generalized solution that adds events, services, and models into the Zend Framework 2 MVC
lifecycle that can be utilized to simplify both authentication and authorization.

## Events

In order to achieve integration to the ZF2 MVC lifecycle for authentication, `api-tools-mvc-auth` wires in
4 listeners that then propagate their own events.  Each of these listeners are registered within the
event `MvcEvent::EVENT_ROUTE` at different priorities.  This table describes the new event
names:

| `api-tools-mvc-auth` event | MVC event in which triggered | MVC event priority |
| :------------------ | :--------------------------: | :----------------: |
| `EVENT_AUTHENTICATION` | `EVENT_ROUTE` | 500 |
| `EVENT_AUTHENTICATION_POST` | `EVENT_ROUTE` | 499 |

(The first column are event constants from `Laminas\ApiTools\MvcAuth\MvcAuthEvent`; the second are event constants
from `Laminas\Mvc\MvcEvent`.)

As you can tell from their priorities, authentication happens *before* routing.  There are
effectively two listeners that deal with authentication related workflows:

- `Laminas\ApiTools\MvcAuth\Authentication\DefaultAuthenticationListener` is registered to listen to the event
  `Laminas\ApiTools\MvcAuth\MvcAuthEvent::EVENT_AUTHENTICATION`.  This listener is generally responsible for (a)
  determining the kind of authentication that needs to take place, whether it is HTTP Basic, HTTP
  Digest, or OAuth2; and (b) authenticating the provided identity. It then assigns the discovered
  identity (either a `Laminas\ApiTools\MvcAuth\AuthenticatedIdentity` on success, or `GuestIdentity` if no
  credentials were provided) to the `MvcAuthEvent`.

- `Laminas\ApiTools\MvcAuth\Authentication\DefaultPostAuthenticationListener` is registered to listen to the event
  `Laminas\ApiTools\MvcAuth\MvcAuthEvent::EVENT_AUTHENTICATION_POST`.  This listener is responsible for 
  determining if some identity was presented by the client, and if it was successfully
  authenticated. If it was not, this listener will inject a `401 Unauthorized` status code to the
  current HTTP response object and return it, ending the request.

| `api-tools-mvc-auth` event | MVC event in which triggered | MVC event priority |
| :------------------ | :--------------------------: | :----------------: |
| `EVENT_AUTHORIZATION` | `EVENT_ROUTE` | -600 |
| `EVENT_AUTHORIZATION_POST` | `EVENT_ROUTE` | -601 |

(The first column are event constants from `Laminas\ApiTools\MvcAuth\MvcAuthEvent`; the second are event constants
from `Laminas\Mvc\MvcEvent`.)

As you can tell from their `EVENT_ROUTE` priorities, authorization happens *after* routing.  There 
are effectively three listeners that deal with authorization related workflows:

- `Laminas\ApiTools\MvcAuth\Authorization\DefaultResourceResolverListener` is registered with the event
  `Laminas\ApiTools\MvcAuth\MvcAuthEvent::EVENT_AUTHORIZATION` and given a priority of `1000` (executes early).
  This listener is responsible for determining the matched controller service name from the matched
  route, which will later be used for checking against the access control lists.

- `Laminas\ApiTools\MvcAuth\Authorization\DefaultAuthorizationListener` is registered with the event
  `Laminas\ApiTools\MvcAuth\MvcAuthEvent::EVENT_AUTHORIZATION` with the default priority.  This listener is
  responsible for taking information from the `Laminas\ApiTools\MvcAuth\MvcAuthEvent` and determining if the
  identity discovered during authentication is authorized to perform the current request against the
  discovered controller service.

- `Laminas\ApiTools\MvcAuth\Authorization\DefaultAuthorizationPostListener` is registered with the
  `Laminas\ApiTools\MvcAuth\MvcAuthEvent::EVENT_AUTHORIZATION_POST` event, at default priority.  This listener is
  responsible for checking if the current request is unauthorized, and if so, it will assign a 
  `403 Unauthorized` status to the HTTP response object, and return it.

## Services & Models

The following table describes services and models that are accessible through the Service Manager:

| Service | Class/Interface returned by service |
| :------ | :---------------------------------- |
| api-identity | `Laminas\ApiTools\MvcAuth\Identity\IdentityInterface` (either a `GuestIdentity` or an `AuthenticatedIdentity`) |
| authentication | `Laminas\Authentication\AuthenticationService` |
| authorization | `Laminas\ApiTools\MvcAuth\Authorization\AuthorizationInterface` (likely a `Laminas\ApiTools\MvcAuth\Authorization\AclAuthorization` instance, a variant of `Laminas\Permissions\Acl\Acl`) |
