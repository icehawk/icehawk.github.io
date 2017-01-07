# IceHawk exceptions

This section describes exceptions throws by the IceHawk component.
 
Because the whole IceHawk framework is written for PHP 7 with strict types we got rid of a lot of defensive validation code 
throwing exceptions. Instead PHP 7 will throw appropriate `\Error`s if the component API is misused.

But there are still cases where we throw exceptions you need to deal with.
 
<hr class="blockspace">
 
## Base component exception

To easily identify whether an exception was thrown by the IceHawk component or not, all exceptions thrown by IceHawk inherit from a base component 
exception class named `IceHawkException`:

```php
<?php declare(strict_types=1);

namespace IceHawk\IceHawk\Exceptions;

class IceHawkException extends \Exception {}
```

So a `catch( IceHawkException $e )` block would catch all possible exceptions thrown by the component.

<hr class="blockspace">

## UnresolvedRequest

If you perform a request and the IceHawk component did not find a matching route for the requested URI, it throws an `UnresolvedRequest` exception.
To help you identify what request caused this exception it carries information about the requested URI and request method:

```php
<?php declare(strict_types=1);

use IceHawk\IceHawk\Exceptions\UnresolvedRequest;

try 
{
	# ...
}
catch ( UnresolvedRequest $e )
{
	printf( "%s request on %s could not be resolved.", $e->getRequestMethod(), $e->getUri() );
}
```

We _recommend_ that you catch this exception in your [final responders](@baseUrl@/docs/icehawk/final-responding.html) and provide a valid `404 Not Found` response.
We intentionally decided not to auto-respond to unresolved requests, because:

* You shall be able to create your own responses, e.g. a custom 404 page or API response.
* We wanted to give you access to the information what request could not be resolved, e.g. for logging.

<hr class="blockspace">
  
## InvalidEventSubscriberCollection

This exception is thrown when you [configured one or more event subscribers](@baseUrl@/docs/icehawk/configuration.html) which do not implement the 
`IceHawk\IceHawk\PubSub\Interfaces\SubscribesToEvents` interface and will be emitted when your IceHawk config is checked - when `IceHawk::init()` is executed.

This exception provides information about the keys/indices that carry the invalid event subscribers:
  
```php
<?php declare(strict_types=1);

use IceHawk\IceHawk\Exceptions\InvalidEventSubscriberCollection;

try 
{
	# ...
}
catch ( InvalidEventSubscriberCollection $e )
{
	printf( "Invalid event subscribers found at: %s", join( ', ', $e->getInvalidKeys() ) );
}
```

<hr class="blockspace">

## EventSubscriberMethodNotCallable

This exception is thrown when you [configured an event subscriber](@baseUrl@/docs/icehawk/configuration.html) to accept a specific event, 
but forgot to implement the appropriate event handler method or used `private` as the visibility level of that method.
 
This exceptions provides information about the expected method name, that could not be found:

**Please note:** This exception is only thrown if you inherit from our abstract class `AbstractEventSubscriber`. 

[Read more about events and subscribers.](@baseUrl@/docs/icehawk/events-and-subscribers.html)

```php
<?php declare(strict_types=1);

use IceHawk\IceHawk\PubSub\Exceptions\EventSubscriberMethodNotCallable;

try 
{
	# ...
}
catch ( EventSubscriberMethodNotCallable $e )
{
	printf( "Could not find evend handler method: %s", $e->getMethodName() );
}
```
 
<hr class="blockspace">

## RoutesAreNotTraversable

This exception is throws when you [configured a list of routes](@baseUrl@/docs/icehawk/routing.html) that is not 
traversable, meaning: which is not an `array` or does not implement the `\Traversable` interface.

**Please note:** This exception [becomes obsolete](https://github.com/icehawk/icehawk/issues/19) in the next major version `v3.0.0` that targets 
PHP 7.1 introducing the [iterable](https://github.com/php/php-src/pull/1941) type.

