# IceHawk delegation

One of IceHawks outstanding features is to provide a defined place to set up the environment. 
This is the concern of the delegate object that IceHawk requires at construction.

This documentation shows you how to use this delegate object.

## Defaults

The IceHawk component ships with a default delegate class ([_IceHawkDelegate.php_](https://github.com/icehawk/icehawk/blob/v2.0.1/src/Defaults/IceHawkDelegate.php)) you can use.

The required interface of the delegate object is: `IceHawk\IceHawk\Interfaces\SetsUpEnvironment`
 
This is how the default delegate looks like:

```php
<?php declare(strict_types=1);

namespace IceHawk\IceHawk\Defaults;

use IceHawk\IceHawk\Interfaces\SetsUpEnvironment;
use IceHawk\IceHawk\Interfaces\ProvidesRequestInfo;

class IceHawkDelegate implements SetsUpEnvironment
{
	public function setUpGlobalVars()
	{
	}
	
	public function setUpErrorHandling( ProvidesRequestInfo $requestInfo )
	{
	}
	
	public function setUpSessionHandling( ProvidesRequestInfo $requestInfo )
	{
	}
}
```

As you can see, the default delegate just does nothing. That means your default PHP environment is used.

You can decide to whether implement the delegate interface or extend the default delegate class.

## Set up global vars

The `setUpGlobalVars` method is meant to be used when you need initially change some values in PHP's super-global variables.
A typical use-case is to map the client IP address when your server acts as a backend of a load balancer.

**Please note:** This method is called before the request info object is pulled from the config, so you are able to manipulate the `$_SERVER` variable 
here that is used by the default request info object.

This is an example how to map the client IP address populated by a load balancer as _HTTP_X_FORWARDED_FOR_:
 
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\SetsUpEnvironment;

class IceHawkDelegate implements SetsUpEnvironment
{
	public function setUpGlobalVars()
	{
		$clientIpAddress = $_SERVER['HTTP_X_FORWARDED_FOR'] ? : $_SERVER['REMOTE_ADDR'];
		
		$_SERVER['REMOTE_ADDR'] = $clientIpAddress;
	}
	
	# ...
}
```

Please see our [Request information](/docs/icehawk/request-information.html) section for the server array keys used by the `RequestInfo` class.

<hr class="blockspace">

## Set up error handling

<hr class="blockspace">

## Set up session handling
