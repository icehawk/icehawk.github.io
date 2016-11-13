# IceHawk events and subscribers 

This section describes how to subscribe to and handle events the IceHawk component emits.
 
<hr class="blockspace">

## Events

The IceHawk component emits the following events in the mentioned order.
The name of each event class implies the moment the event is emitted, because we always publish an event before and after an action takes resp. took place.

All event classes implement the [CarriesEventData](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/PubSub/Interfaces/CarriesEventData.php)
interface that guaranties a getter named `getRequestInfo()` to retrieve the current [request info object](@baseUrl@/docs/icehawk/request-information.html).

### 1. InitializingIceHawkEvent
  
This event is published whithin the `IceHawk::init()` method, after the [ConfiguresIceHawk::setUpGlobalVars()](@baseUrl@/docs/icehawk/configuration.html) is called, but before
[ConfiguresIceHawk::setUpErrorHandling()](@baseUrl@/docs/icehawk/configuration.html) and [ConfiguresIceHawk::setUpSessionHandling()](@baseUrl@/docs/icehawk/configuration.html) are called.  

The event carries the current [request info object](@baseUrl@/docs/icehawk/request-information.html).

### 2. IceHawkWasInitializedEvent

This event is published whithin the `IceHawk::init()` method, after
[ConfiguresIceHawk::setUpErrorHandling()](@baseUrl@/docs/icehawk/configuration.html) and [ConfiguresIceHawk::setUpSessionHandling()](@baseUrl@/docs/icehawk/configuration.html) were executed.  

The event carries the current [request info object](@baseUrl@/docs/icehawk/request-information.html).

### 3. HandlingReadRequestEvent / HandlingWriteRequestEvent

This event is published within the `IceHawk::handleRequest()` method after the route was resolved, but before the actual request handler's `handle()` 
method is called.

If the current request is a read request (HTTP methods GET or HEAD), the `HandlingReadRequestEvent` will be emitted. If the current request is a write 
request (HTTP methods POST, PUT, PATCH, DELETE), the `HandlingWriteRequestEvent` will be emitted.

The event carries the current request object including [request info](@baseUrl@/docs/icehawk/request-information.html) and 
[request input data](@baseUrl@/docs/icehawk/request-input-data.html) objects.

### 4. ReadRequestWasHandled / WriteRequestWasHandled

This event is published within the `IceHawk::handleRequest()` method after the actual request handler's `handle()` method was executed.

If the current request is a read request (HTTP methods GET or HEAD), the `ReadRequestWasHandledEvent` will be emitted. If the current request is a write 
request (HTTP methods POST, PUT, PATCH, DELETE), the `WriteRequestWasHandledEvent` will be emitted.

The event carries the current request object including [request info](@baseUrl@/docs/icehawk/request-information.html) and 
[request input data](@baseUrl@/docs/icehawk/request-input-data.html) objects.

<hr class="blockspace">

## Event subscribers

An event subscriber needs to implement the [SubscribesToEvents](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/PubSub/Interfaces/SubscribesToEvents.php)
interface. This interface has two methods:

```php
public function acceptsEvent( CarriesEventData $event ) : bool
```

This method shall return a boolean whether the passed event is accepted by this subscriber or not.

```php
public function notify( CarriesEventData $event )
```

If the subscriber accepts the current event, this method is called and gets the current event passed. 

The intention of the `notify()` method is to call your business logic for the passed event.

<hr class="blockspace">

## Implementing event subscribers

One way to implement an event subscriber is of course to write a class that implements the `SubscribesToEvents` interface and do the handling
all on your own.

To help you out with the handling the IceHawk ships with an [abstract event subscriber](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/PubSub/AbstractEventSubscriber.php) 
class, that requires you to return the accepted event class names an to implement a protected/public event handler method. 

An example implementation for an event subscriber for the `IceHawkWasInitializedEvent` would look like this:
  
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\PubSub\AbstractEventSubscriber;
use IceHawk\IceHawk\Events\IceHawkWasInitializedEvent;

final class YourEventSubscriber extends AbstractEventSubscriber
{
	public function getAcceptedEvents() : array
	{
		return [
			IceHawkWasInitializedEvent::class,
		];
	}
	
	protected function whenIceHawkWasInitialized( IceHawkWasInitializedEvent $event )
	{
		echo "IceHawk was initialized with request URI: " . $event->getRequestInfo()->getUri();
	}
}
```

As you can see the abstract event subscriber drills down the `acceptsEvent()` method to a simple question for event class names (`getAcceptedEvents()`).
Furthermore it tries to call a method named after the event class on the event subscriber - the event handler method.

The name of the event handler method has the prefix "when" to describe that it is called _when_ the subscriber is notified about the event.
In addition the suffix "Event" is omitted in the method name. Furthermore the concrete event class name is used as the parameter type declaration, 
to make clear that this handler method is only responsible for this particular event.

<hr class="blockspace">

## Registering event subscribers

You can register your event subscribers in the [IceHawk config's `getEventSubscribers()` method](@baseUrl@/docs/icehawk/configuration.html) by simply 
returning an array of event subscriber instances.

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;

final class IceHawkConfig implements ConfiguresIceHawk
{
	# ...
	
	public function getEventSubscribers() : array
	{
		return [
			new YourEventSubscriber(),	
		];
	}
}
```

**Please note:** The IceHawk component makes sure that the event subscribers were instantiated only once.
