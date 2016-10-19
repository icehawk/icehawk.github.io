# IceHawk configuration

This documentation shows you which configuration abilities IceHawk offers.

<hr class="blockspace">

## Defaults

IceHawk makes use of a ["traitful config"](https://phpind.de/posts/traitful-configs?locale=en_US) and offers an interface for its configuration, 
which can be implemented by simply using several traits covering all interface methods.

The configuration interface is: `IceHawk\IceHawk\Interfaces\ConfiguresIceHawk`

Here is how this looks like:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	use Defaults\Traits\DefaultRequestProviding;
	use Defaults\Traits\DefaultReadRouting;
	use Defaults\Traits\DefaultWriteRouting;
	use Defaults\Traits\DefaultEventSubscribing;
	use Defaults\Traits\DefaultFinalReadResponding;
	use Defaults\Traits\DefaultFinalWriteResponding;
}
```

By the way, this is (pretty much) the [_IceHawkConfig.php_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Defaults/IceHawkConfig.php) that the IceHawk component already ships.

Unpacking the traits, this is what the config class would look like:


```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RoutesToReadHandler;
use IceHawk\IceHawk\Interfaces\RoutesToWriteHandler;
use IceHawk\IceHawk\Interfaces\ProvidesRequestInfo;
use IceHawk\IceHawk\Interfaces\RespondsFinallyToReadRequest;
use IceHawk\IceHawk\Interfaces\RespondsFinallyToWriteRequest;
use IceHawk\IceHawk\Defaults\RequestInfo;
use IceHawk\IceHawk\Defaults\FinalReadResponder;
use IceHawk\IceHawk\Defaults\FinalWriteResponder;

class IceHawkConfig implements ConfiguresIceHawk
{
	/**
	 * @return array|\Traversable|RoutesToReadHandler[]
	 */
	public function getReadRoutes()
	{
		return [];
	}
	
	/**
	 * @return array|\Traversable|RoutesToWriteHandler[]
	 */
	public function getWriteRoutes()
	{
		return [];
	}
	
	/**
	 * @return array|SubscribesToEvents[]
	 */
	public function getEventSubscribers() : array
	{
		return [];
	}
	
	public function getRequestInfo() : ProvidesRequestInfo
	{
		return RequestInfo::fromEnv();	
	}
	
	public function getFinalReadResponder() : RespondsFinallyToReadRequest
	{
		return new FinalReadResponder();
	}
	
	public function getFinalWriteResponder() : RespondsFinallyToWriteRequest
	{
		return new FinalWriteResponder();
	}
}
```

As you can see the config is very empty. Now let's have a look at all these methods.
   
You can decide whether you implement the configuration interface or extend the shipped default configuration. 
   
<hr class="blockspace">

## Configure read routes

This example shows 3 different read routes:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RoutesToReadHandler;
use IceHawk\IceHawk\Routing\ReadRoute;
use IceHawk\IceHawk\Routing\Patterns\Literal;
use IceHawk\IceHawk\Routing\Patterns\RegExp;
use IceHawk\IceHawk\Routing\Patterns\NamedRegExp;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	use Defaults\Traits\DefaultRequestProviding;
	use Defaults\Traits\DefaultWriteRouting;
	use Defaults\Traits\DefaultEventSubscribing;
	use Defaults\Traits\DefaultFinalReadResponding;
	use Defaults\Traits\DefaultFinalWriteResponding;
	
	/**
	 * @return array|\Traversable|RoutesToReadHandler[]
	 */
	public function getReadRoutes()
	{
		return [
			
			# Simple route with a fixed URI for the home page
			new ReadRoute( new Literal( '/' ), new ShowHomeRequestHandler() ),
			
			# Route to a product page using a regular expression and a map for its matches
			# In words: The first match of the regular expression shall be named "productId"
			new ReadRoute( 
				new RegExp( "#^/product/([0-9]+)/$#", ['productId'] ), 
				new ShowProductRequestHandler() 
			),
			
			# Route to a gallery page using a named regular expression
			# A more comfortable way to express the previous example
			# because the match name is already in the regular expression
			new ReadRoute( 
				new NamedRegExp( '^/gallery/(?<galleryId>[^/]+)/$' ), 
				new ShowGalleryRequestHandler() 
			),
		];
	}
}
```

As you may noticed, we omitted the return type for this method, because we wanted to allow building a `\Generator` and `yield`ing the routes.  

Please see our [routing section](/docs/icehawk/routing.html) for more details and examples. 

<hr class="blockspace">

## Configure write routes

This example shows 3 different write routes:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RoutesToWriteHandler;
use IceHawk\IceHawk\Routing\WriteRoute;
use IceHawk\IceHawk\Routing\Patterns\Literal;
use IceHawk\IceHawk\Routing\Patterns\RegExp;
use IceHawk\IceHawk\Routing\Patterns\NamedRegExp;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	use Defaults\Traits\DefaultRequestProviding;
	use Defaults\Traits\DefaultReadRouting;
	use Defaults\Traits\DefaultEventSubscribing;
	use Defaults\Traits\DefaultFinalReadResponding;
	use Defaults\Traits\DefaultFinalWriteResponding;
	
	/**
	 * @return array|\Traversable|RoutesToWriteHandler[]
	 */
	public function getWriteRoutes()
	{
		return [
			
			# Simple route with a fixed URI for a new user to register
			new WriteRoute( new Literal( '/register' ), new RegisterUserRequestHandler() ),
			
			# Route to a user profile using a regular expression and a map for its matches
			# In words: The first match of the regular expression shall be named "userId"
			new WriteRoute( 
				new RegExp( "#^/users/([0-9]+)/$#", ['userId'] ), 
				new UpdateProfileRequestHandler() 
			),
			
			# Route to a user profile password using a named regular expression
			# A more comfortable way to express the previous example
			# because the match name is already in the regular expression
			new WriteRoute( 
				new NamedRegExp( '^/users/(?<userId>[0-9]+)/password/$' ), 
				new ChangePasswordRequestHandler() 
			),
		];
	}
}
```

As you may noticed, we omitted the return type for this method, because we wanted to allow building a `\Generator` and `yield`ing the routes.  

Please see our [routing section](/docs/icehawk/routing.html) for more details and examples. 

<hr class="blockspace">

## Configure event subscribers

IceHawk publishes some events you can subscribe to. The following example shows how to register such event subscribers:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\SubscribesToEvents;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	use Defaults\Traits\DefaultRequestProviding;
	use Defaults\Traits\DefaultReadRouting;
	use Defaults\Traits\DefaultWriteRouting;
	use Defaults\Traits\DefaultFinalReadResponding;
	use Defaults\Traits\DefaultFinalWriteResponding;
	
	/**
	 * @return array|SubscribesToEvents[]
	 */
	public function getEventSubscribers() : array
	{
		return [
			# Subscriber for initialization events
			new InitializationEventSubscriber(),
			
			# Subscriber for read request handling events
			new ReadRequestHandlingEventSubscriber(),
			
			# Subscriber for write request handling events
			new WriteRequestHandlingEventSubscriber(), 			
		];
	}
}
```

So you simply need to return an array of subscriber instances.

To create an event subscriber, you can either implement the subscriber interface `IceHawk\IceHawk\Interfaces\SubscribesToEvents`
or simply extend the abstract event subscriber that the IceHawks component ships: `IceHawk\IceHawk\PubSub\AbstractEventSubscriber`.
 
Read more about how to implement an event subscriber in out [Events and subscribers section](/docs/icehawk/events-and-subscribers.html).

<hr class="blockspace">

## Configure the request information object

The request information object is technically a wrapper for the super-global variable `$_SERVER` and provides important
information to the IceHawk component like the request method, that is used to decide between read or write request handling.

In most cases it should be absolutely fine to simply use the shipped default [RequestInfo](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Defaults/RequestInfo.php) object.
But in case you need to have own logic or your `$_SERVER` variable holds non-standard keys you also can provide a mapped array to the default request info object 
or implement its interface `IceHawk\IceHawk\Interfaces\ProvidesRequestInfo` and provide an own object.

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\SubscribesToEvents;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	use Defaults\Traits\DefaultReadRouting;
	use Defaults\Traits\DefaultWriteRouting;
	use Defaults\Traits\DefaultEventSubscribing;
	use Defaults\Traits\DefaultFinalReadResponding;
	use Defaults\Traits\DefaultFinalWriteResponding;
	
	public function getRequestInfo() : ProvidesRequestInfo
	{
		# Return the default request info object constructed from the current environment (using `$_SERVER`)
		return Defaults\RequestInfo::fromEnv();
		
		# Return the default request info object with own data array
		return new Defaults\RequestInfo( $yourOwnData );
		
		# Return an own request info object
		return new YourOwnRequestInfo( $_SERVER );
	}
}
```

When using the shipped default request info object with construction from enviromnent (using `$_SERVER`), you also can simply use the default trait, instead of implementing the method.
 

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	# ...
	# Use default request info object contructed from environment (using $_SERVER)
	use Defaults\Traits\DefaultRequestInfoProviding;
	# ...
}
```

Please see our [Request information](/docs/icehawk/request-information.html) section for more detailed information about the request object.

<hr class="blockspace">

## Configure final responding

The IceHawk component lets you configure a custom final responding behaviour. The final responder is called when an uncaught exception (`\Throwable` to be accurate) occurres while your request handling.
This gives you the chance to still provide a valid answer to the user or request caller. Typically you would use this to provide a valid _404 Not Found_ or _500 Internal Server Error_ page.
  
As the final responding for read requests can differ from the final respoding for write requests you need to provide a final responders for each case.
 
The component ships two default final responders that do nothing but throwing the occurred excetion (`\Throwable`), which is fine for development.
For production you should provide own final responders.

Final responders must implement the following interfaces:
 
* For read requests: `IceHawk\IceHawk\Interfaces\RespondsFinallyToReadRequest`
* For write requests: `IceHawk\IceHawk\Interfaces\RespondsFinallyToWriteRequest`

This is an example how to provide basic final responders:  
(For better readability the final responder is expressed as an [anonymous class](http://php.net/manual/en/language.oop5.anonymous.php))

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Constants\HttpCode;
use IceHawk\IceHawk\Exceptions\UnresolvedRequest;
use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RespondsFinallyToReadRequests;
use IceHawk\IceHawk\Interfaces\RespondsFinallyToWriteRequests;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
	use Defaults\Traits\DefaultRequestInfoProviding;
	use Defaults\Traits\DefaultReadRouting;
	use Defaults\Traits\DefaultWriteRouting;
	use Defaults\Traits\DefaultEventSubscribing;
	
	public function getFinalReadResponder() : RespondsFinallyToReadRequests
	{
		return new class implements RespondsFinallyToReadRequests
		{
			public function handleUncaughtException( 
				\Throwable $throwable, 
				ProvidesReadRequestData $request 
			)
			{
				try 
				{
					throw $throwable;
				}
				catch (UnresolvedRequest $e)
				{
					# IceHawk was unable to resolve the request
					# a.k.a. there is no valid read route configured
					# Respond with a 404 Not Found
					header(
						'Content-Type: text/plain; charset=utf-8', 
						true, 
						HttpCode::NOT_FOUND
					);
					echo "404 Not Found: " . $request->getInfo()->getUri();
					flush();
				}
				catch (\Throwable $e)
				{
					# In any other case respond with a 500 Internal Server Error
					header(
						'Content-Type: text/plain; charset=utf-8', 
						true, 
						HttpCode::INTERNAL_SERVER_ERROR
					);
					echo "500 Internal Server Error: " . $request->getInfo()->getUri();
					flush();
				}
			}
		};
	}
	
	public function getFinalWriteResponder() : RespondsFinallyToWriteRequests
	{
		return new class implements RespondsFinallyToWriteRequests
		{
			public function handleUncaughtException( 
				\Throwable $throwable, 
				ProvidesWriteRequestData $request 
			)
			{
				try 
				{
					throw $throwable;
				}
				catch (UnresolvedRequest $e)
				{
					# IceHawk was unable to resolve the request
					# a.k.a. there is no valid write route configured
					# Respond with a 301 redirect to a 404 page
					header( 'Location: /404.html', true, HttpCode::MOVED_PERMANENTLY );
					flush();
				}
				catch (\Throwable $e)
				{
					# In any other case respond with a 301 redirect to a 500 page
					header( 'Location: /500.html', true, HttpCode::MOVED_PERMANENTLY );
                    flush();
				}
			}
		};
	}
}
```

**Please note:** You always get the request object as the second parameter to the final responder method, so you are able to implement custom behaviour based on the actual request.

Please see our [Exceptions](/docs/icehawk/exceptions.html) section for more information about the exceptions thrown by the IceHawk component itself.
 
Please see our [Final responding](/docs/icehawk/final-responding.html) section for more information on how to implement a final responder.

## Use of objects provides by the config

Do not hesitate to provide new instances in all the configuration methods. 
The IceHawk component makes sure that all methods of the config were called only once - even if an object is internally used multiple times.

