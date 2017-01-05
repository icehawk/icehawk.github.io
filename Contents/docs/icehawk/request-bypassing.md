# Request bypassing
 
One of IceHawk's main goals is to help you keep your application divided into a read and a write side. But when interacting with external systems, like e.g. 
external payment providers, you probably be confronted with a situation where you receive a GET request that should trigger a write operation in your
application. Since a GET request is bound to the read side this would go against the previous mentioned goal.
 
In order to process such requests properly on the write side of your application the IceHawk component offers "request bypassing" to internally 
re-route such a request (without HTTP redirection) to the write side of your application. 

To make that behaviour a conscious decision a separate configuration is needed.
   
This section shows how to set up request bypassing.

---

## Configure a request bypass

As you can see in the [configuration section](@baseUrl@/docs/icehawk/configuration.html) there is a method called `getRequestBypasses()`, which returns an empty array by default:

```php
/**
 * @return array|\Traversable|BypassesRequest[]
 */
public function getRequestBypasses()
{
	return [];
}
```

Assuming the following use-case:

* Your application receives a 
    ```http
    HTTP/1.1 GET: /payment/success?token=12345
    ```
* Your application shall process this request like 
    ```http
    HTTP/1.1 POST: /payment/mark-succeeded
    token=12345
    ```

The configuration could look like this:
 
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RoutesToWriteHandler;
use IceHawk\IceHawk\Interfaces\BypassesRequest;
use IceHawk\IceHawk\Routing\WriteRoute;
use IceHawk\IceHawk\Routing\RequestBypass;
use IceHawk\IceHawk\Routing\Patterns\Literal;
use IceHawk\IceHawk\Defaults;

class IceHawkConfig implements ConfiguresIceHawk
{
    use Defaults\Traits\DefaultRequestProviding;
    use Defaults\Traits\DefaultReadRouting;
    use Defaults\Traits\DefaultEventSubscribing;
    use Defaults\Traits\DefaultCookieProviding;
    use Defaults\Traits\DefaultFinalReadResponding;
    use Defaults\Traits\DefaultFinalWriteResponding;

    /**
     * @return array|\Traversable|BypassesRequest[]
     */
    public function getRequestBypasses()
    {
    	return [
    	    new RequestBypass(
    	    	new Literal('/payment/success'), 
    	    	'/payment/mark-succeeded', 
    	    	HttpMethod::POST
    	    ),	
		];
    }

    /**
     * @return array|\Traversable|RoutesToWriteHandler[]
     */
    public function getWriteRoutes()
    {
        return [
            new WriteRoute( 
            	new Literal('/payment/mark-succeeded'), 
            	new MarkPaymentSucceededRequestHandler() 
            ),
        ];
    }
}
```

**Please note:** The `MarkPaymentSucceededRequestHandler` has to implement the `IceHawk\IceHawk\Interfaces\HandlesPostRequest` interface.

Of course you can use any of the [URI pattern classes](@baseUrl@/docs/icehawk/routing.html#uri-pattern-classes) for matching in the request bypass.
