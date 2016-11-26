# IceHawk delegation

One of IceHawks outstanding features is to provide a defined place to set up the environment. 
This is the concern of the delegate object that IceHawk requires at construction.

This documentation shows you how to use this delegate object.

<hr class="blockspace">

## Defaults

The IceHawk component ships with a default delegate class ([_IceHawkDelegate.php_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Defaults/IceHawkDelegate.php)) you can use if you don't need any set up.

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

The call order of the delegate methods is as shown above in the default class. 

<hr class="blockspace">

## Set up global vars

The `setUpGlobalVars` method is meant to be used when you need initially change some values in PHP's super-global variables.
A typical use-case is to map the client IP address when your server acts as a backend of a load balancer.

**Please note:** This method is called before the request info object is pulled from the config, so you are able to manipulate the `$_SERVER` variable 
here that is used by the default request info object.

This is an example how to map the client IP address populated by a load balancer as `HTTP_X_FORWARDED_FOR`:
 
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\SetsUpEnvironment;

class IceHawkDelegate implements SetsUpEnvironment
{
	public function setUpGlobalVars()
	{
		$clientIpAddress = $_SERVER['HTTP_X_FORWARDED_FOR'] ?? $_SERVER['REMOTE_ADDR'];
		
		$_SERVER['REMOTE_ADDR'] = $clientIpAddress;
	}
	
	# ...
}
```

Please see our [Request information](/docs/icehawk/request-information.html) section for the server array keys used by the `RequestInfo` class.

<hr class="blockspace">

## Set up error handling

The `setUpErrorHandling` method is your place to configure your application's error handling, e.g. by 
[registering an own error handler](http://php.net/manual/en/function.set-error-handler.php) or 
simply change the error reporting level.

**Please note:** This method is called after pulling the request info object from the config, so we can serve it here.

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\SetsUpEnvironment;

class IceHawkDelegate implements SetsUpEnvironment
{
	# ...
	
	public function setUpErrorHandling( ProvidesRequestInfo $requestInfo )
	{
		# simply turn on PHP error reporting and display errors
		error_reporting( E_ALL );
		ini_set( 'display_errors', 'On' );
		
		# or register an own error handler
		set_error_handler([YourErrorHandler::class, 'handleError']);
		
		# or conditionally enable/disable error handling based on current request information
		if ( $requestInfo->getQueryString() == 'reportErrors=yes' )
		{
			error_reporting( E_ALL );
            ini_set( 'display_errors', 'On' );
		}
	}
	
	# ...
}
```

<hr class="blockspace">

## Set up session handling

The `setUpSessionHandling` method is your place to configure your application's session handling, e.g. by 
[registering an own session handler](http://php.net/manual/en/function.session-set-save-handler.php) or 
simply set up ini and cookie values.

**Please note:** This method is called after pulling the request info object from the config, so we can serve it here.

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\SetsUpEnvironment;

class IceHawkDelegate implements SetsUpEnvironment
{
	# ...
	
	# ...
	
	public function setUpSessionHandling( ProvidesRequestInfo $requestInfo )
	{
		# simply setup ini and cookie settings
		ini_set( 'session.name', 'yourSID' );
		ini_set( 'session.save_handler', 'redis' );
		ini_set( 'session.save_path', 'tcp://127.0.0.1:6379?weight=1&database=0' );
		ini_set( 'session.gc_maxlifetime', (60 * 60 * 24) );
        
		session_set_cookie_params( (60 * 60 * 24), '/', '.your-domain.com', true, true );
		
		# or register an own session handler
		session_set_save_handler(new YourSessionhandler());
		
		# or conditionally set up session handling based on current request information
		if ( strpos( $requestInfo->getUri(), '/long-life/') === 0 )
		{
			ini_set( 'session.name', 'yourLongSID' );
			ini_set( 'session.gc_maxlifetime', (60 * 60 * 24 * 365) );
            
            session_set_cookie_params( (60 * 60 * 24 * 365), '/', '.your-domain.com', true, true );
		}
	}
}
```
