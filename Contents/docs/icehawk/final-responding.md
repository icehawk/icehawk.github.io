# IceHawk final responding 

One of IceHawk's outstanding features is the delegation of thrown exceptions to so called final responders. 
Final responders come in handy when you want a centralized and unified handling of uncaught exceptions. They also give you a last chance to provide a 
valid response to the user / client. Furthermore you don't need to handle re-occurring exceptions in multiple request handlers by 
duplicating your code.  
 
`\Exception`s and `\Error`s (`\Throwable`s) are delegated to the appropriate final responder when they were thrown during the routing or request handling process.
 
<hr class="blockspace">

## Set up final responders

Because the IceHawk component separates between read and write requests, we also offer to set up final responders for each side.
We ship two default final responders that do nothing, but throwing the passed `\Throwable` instance.
 
In the [IceHawk config](@baseUrl@/docs/icehawk/configuration.html) you are able to define your own final responders like this:

```php
<php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RespondsFinallyToReadRequest;
use IceHawk\IceHawk\Interfaces\RespondsFinallyToWriteRequest;

final class IceHawkConfig implements ConfiguresIceHawk
{
	# ...
	
	public function getFinalReadResponder() : RespondsFinallyToReadRequest
	{
		return new YourFinalReadResponder();
	}
	
	public function getFinalWriteResponder() : RespondsFinallyToWriteRequest
	{
		return new YourFinalWriteResponder();
	}
}
```

<hr class="blockspace">

## Implementing final responders

As you can see in the previous paragraph a final responder needs to implement the 
[RespondsFinallyToReadRequest](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Interfaces/RespondsFinallyToReadRequest.php) resp. 
[RespondsFinallyToWriteRequest](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Interfaces/RespondsFinallyToWriteRequest.php) interface.

These interfaces are basically the same and require a method named `handleUncaughtException()`. The only difference is the parameter type declaration 
for the `$request` object that provides access to the current [request information](@baseUrl@/docs/icehawk/request-information.html) and 
[request input data](@baseUrl@/docs/icehawk/request-input-data.html).

```php
# Read side method
public function handleUncaughtException( \Throwable $throwable, ProvidesReadRequestData $request )

# Write side method
public function handleUncaughtException( \Throwable $throwable, ProvidesWriteRequestData $request )
```

The following example shows how your application could finally respond to an [`UnresolvedRequest` exception](@baseUrl@/docs/icehawk/exceptions.html).

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\RespondsFinallyToReadRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;
use IceHawk\IceHawk\Exceptions\UnresolvedRequest;
use IceHawk\IceHawk\Constants\HttpCode;

final class YourFinalReadResponder implements RespondsFinallyToReadRequest
{
	public function handleUncaughtException( \Throwable $throwable, ProvidesReadRequestData $request )
	{
		try 
		{
			throw $throwable;
		}
		catch ( UnresolvedRequest $e )
		{
			# Handle unresolved requests and respond with HTTP/1.1 404 Not Found
			header( 'Content-Type: text/plain; charset=utf-8', true, HttpCode::NOT_FOUND );
			echo "404 - Not Found";
			flush();
		}
		catch ( \Throwable $e )
		{
			# Handle any other exception / error
			
			# Log the error
			$logMessage = sprintf(
				'Uncaught %s thrown in %s on line %s with message: %s',
				get_class($e),
				$e->getFile(),
				$e->getLine(),
				$e->getMessage()
			);
			
			error_log( $logMessage, 3, '/path/to/error.log' );
			
			# Respond with HTTP/1.1 500 Internal Server Error
			header( 'Content-Type: text/plain; charset=utf-8', true, HttpCode::INTERNAL_SERVER_ERROR );
			echo "500 - Internal Server Error";
			flush();
		}
	}
}
```

As you can see you can decide what level of exceptions / errors you want to handle and how.

<hr class="blockquote">

## Why two final responders?

You may wonder why we also separated the final responders for read and write requests. One reason is, that we wanted to handle especially write 
requests in a different way than read requests. On a read request we can simply respond with an appropriate HTTP status code and error page like shown above.

If a write request throws an uncaught exception we don't want the user / client to re-submit that request over and over again, e.g. by hitting <kbd>F5</kbd> 
in the browser, as this may causes more harm to our application or floods our error log. Instead we want to inform the user that something went wrong and respond with a redirect 
to an appropriate error page - which is a read request responding directly with the correct headers and output. If the user / client refreshes now only 
the read request is performed again, no data is submitted again to the server, no exception / error is produced again.
    
The following example shows how such a final write request responder could look like:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\RespondsFinallyToWriteRequest;
use IceHawk\IceHawk\Interfaces\ProvidesWriteRequestData;
use IceHawk\IceHawk\Constants\HttpCode;

final class YourFinalWriteResponder implements RespondsFinallyToWriteRequest
{
	public function handleUncaughtException( \Throwable $throwable, ProvidesWriteRequestData $request )
	{
		try 
		{
			throw $throwable;
		}
		catch ( \Throwable $e )
		{
			# Handle any other exception / error
			
			# Log the error (once)
			$logMessage = sprintf(
				'Uncaught %s thrown in %s on line %s with message: %s',
				get_class($e),
				$e->getFile(),
				$e->getLine(),
				$e->getMessage()
			);
			
			error_log( $logMessage, 3, '/path/to/error.log' );
			
			# Respond with redirect to a 500 error page
			header( 'Location: /error/500', true, HttpCode::TEMPORARY_REDIRECT );
			flush();
		}
	}
}
```
