# IceHawk request information 

The IceHawk component wraps the informative part of the current request (usually the superglobal `$_SERVER` variable) into an object of the `RequestInfo` class.
This documentation shows you what you can get out of it.

<hr class="blockspace">

## The request info interface 

The [RequestInfo class](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Defaults/RequestInfo.php) belongs to IceHawk's default 
classes and implements the interface `IceHawk\IceHawk\Interfaces\ProvidesRequestInfo` which looks as follows:

```php
<?php declare(strict_types = 1);

namespace IceHawk\IceHawk\Interfaces;

interface ProvidesRequestInfo
{
	public function isSecure() : bool;
	
	public function getMethod() : string;
	
	public function getUri() : string;
	
	public function getHost() : string;
	
	public function getUserAgent() : string;
	
	public function getServerAddress() : string;
	
	public function getClientAddress() : string;
	
	public function getRequestTimeFloat() : float;
	
	public function getAcceptedContentTypes() : string;
	
	public function getQueryString() : string;
	
	public function getReferer() : string;
	
	public function getAuthUser() : string;
	
	public function getAuthPassword() : string;
	
	public function getContentType() : string;
	
	public function getContentLength() : string;
}
```

<hr class="blockspace">

## How to create an instance?

If you want to create an instance based on the superglobal `$_SERVER` variable, do it like this:
 
```php
<?php declare(strict_types=1);

# ...

# This is IceHawk's default:

$requestInfo = RequestInfo::fromEnv();

# This is the same as:

$requestInfo = new RequestInfo( $_SERVER );
```

As you can see you can inject any array into the constructor of the class. This has 2 main advantages:
 
1. You can easily test the object using unit tests
2. You can easily inject user-defined data instead of using the `$_SERVER` array

<hr class="blockspace">

## Where to intantiate the object?

As shown in the [configuration section](@baseUrl@/docs/icehawk/configuration.html) you need to provide an instance of the `RequestInfo` class 
(or an implementation of the `ProvidesRequestInfo` interface) once in the `getRequestInfo()` method. IceHawk will then inject it to all subsequent objects. 

<hr class="blockspace">

## Where can I use the RequestInfo object?

The `RequestInfo` object is injected to all relevant objects where you can place your code. In Detail:
  
### IceHawk delegate

As you can see in the [delegation section](@baseUrl@/docs/icehawk/delegation.html) the IceHawk component provides places to set up your error and/or session handling.

The request info object is directly passed to these delegate methods:

```php
class IceHawkDelegate implements SetsUpEnvironment
{
	public function setUpErrorHandling( ProvidesRequestInfo $requestInfo )
	{
		
	}
	
	public function setUpSessionHandling( ProvidesRequestInfo $requestInfo )
	{
		
	}
}
```
  
### RequestHandlers

Every IceHawk request handler, regardless of read or write side, needs to implement one method:

```php
# Read side
public function handle( ProvidesReadRequestData $request );

# Write side
public function handle( ProvidesReadRequestData $request );
```

These methods are defined by the two base interfaces for request handlers: 

* `IceHawk\IceHawk\Interfaces\HandlesReadRequest`
* `IceHawk\IceHawk\Interfaces\HandlesWriteRequest`

The provided `$request` object gives you access to a) the request info object and b) to the request input data as follows:

```php
# Read side
public function handle( ProvidesReadRequestData $request )
{
	# Get the request info object
	$requestInfo = $request->getInfo();
	
	# Get the request input data object
	$requestInput = $request->getInput();
}

# Write side
public function handle( ProvidesWriteRequestData $request )
{
	# Get the request info object
	$requestInfo = $request->getInfo();
	
	# Get the request input data object
	$requestInput = $request->getInput();
}
```

### IceHawk Events

As you can read in the [events and subscribers section](@baseUrl@/docs/icehawk/events-and-subscribers.html) the IceHawk component publishes some events you can subscribe to.
The following events will also carry the `RequestInfo` object:

* [InitializingIceHawkEvent](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Events/InitializingIceHawkEvent.php)
* [IceHawkWasInitializedEvent](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Events/IceHawkWasInitializedEvent.php)
* [HandlingReadRequestEvent](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Events/HandlingReadRequestEvent.php)
* [HandlingWriteRequestEvent](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Events/HandlingWriteRequestEvent.php)
* [ReadRequestWasHandledEvent](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Events/ReadRequestWasHandledEvent.php)
* [WriteRequestWasHandledEvent](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Events/WriteRequestWasHandledEvent.php)

The following example shows how to access the `RequestInfo` object inside an event subscriber:

```php
class YourReadHandlingEventSubscriber extends AbstractEventSubscriber
{
	protected function whenHandlingReadRequest( HandlingReadRequestEvent $event )
	{
		# Get request info object
		$requestInfo = $event->getRequestInfo();
	}
}
```

This is the same for all other subscribers / events.

### Final responders

As you can read in the [final responding section](@baseUrl@/docs/icehawk/final-responding.html) the IceHawk component lets you finally respond to requests that caused an error.

The final responder object gets passed the same `$request` object as a request handler would get. This example shows how to access the request info object in a final responder:
  
```php
class YourFinalReadResponder implements RespondsFinallyToReadRequest
{
	public function handleUncaughtException( \Throwable $throwable, ProvidesReadRequestData $request )
	{
		# Get the request info object
		$requestInfo = $request->getInfo();
	}
}
```

<hr class="blockspace">

## What array keys does the default RequestInfo expect?

**Please note:**

* These keys are only relevant if you use **IceHawk's default `RequestInfo` class**. In your own implementation you can of course choose whatever keys you want.
* The keys are accessed case-sensitive and therefor need to be _UPPERCASE_.

**Used keys:**

* `CONTENT_LENGTH`
* `CONTENT_TYPE`
* `HTTP_ACCEPT`
* `HTTP_HOST`
* `HTTP_REFERER`
* `HTTP_USER_AGENT`
* `HTTPS`
* `PHP_AUTH_PW`
* `PHP_AUTH_USER`
* `QUERY_STRING`
* `REMOTE_ADDR`
* `REQUEST_METHOD`
* `REQUEST_TIME_FLOAT`
* `REQUEST_URI`
* `SERVER_ADDR`

Please see the official [PHP documentation](http://php.net/manual/en/reserved.variables.server.php) for more information about the `$_SERVER` indices.

<hr class="blockspace">

## What do you get from the methods?

```php
public function isSecure() : bool
```

This methods returns whether the current request comes on a secured (SSL) connection or not. (Key: `HTTPS`)

**Please note:** The check of the value "on" is case-insensitive, so all the following variants will result to `true`: 
`['HTTPS'=> 'on']`, `['HTTPS' => 'On']`, `['HTTPS' => 'ON']`, `['HTTPS' => 'oN']`.

<hr class="blockspace">

```php
public function getMethod() : string
```

This method returns the HTTP method (or verb) of the current request. (Key: `REQUEST_METHOD`)
 
**Please note:**
 
* The returned string is always _UPPERCASE_.
* The IceHawk component ships with an abstract class containing all HTTP methods (or verbs) as class constants: [HttpMethod](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Constants/HttpMethod.php). You can use this class constants to check against the results of the `RequestInfo::getMethod()`.
* The IceHawk component decides whether to do read or write handling based on the result of this method. (So it'd better be correct!)

<hr class="blockspace">

```php
public function getUri() : string
```

This method returns the current request URI _without_ the query string. (Key: `REQUEST_URI`)

**Please note:**

* This method strips the query string part of the request URI. Use `getQueryString()` to retrieve it.
* The patterns of your routes is matched against the result of this method. See the [routing section](@baseUrl@/docs/icehawk/routing.html) for more information.
 
<hr class="blockspace">

```php
public function getHost() : string
```

This method returns the current HTTP hostname. (Key: `HTTP_HOST`)

For a request on this page, it should return: `icehawk.github.io`.
 
<hr class="blockspace">

```php
public function getUserAgent() : string
```

This method returns the client's user agent string. (Key: `HTTP_USER_AGENT`)

A request from a Google Chrome web browser for example can result in: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36`.

<hr class="blockspace">

```php
public function getServerAddress() : string
```

This method returns the ip address of the server handling the current request. (Key: `SERVER_ADDR`)

<hr class="blockspace">

```php
public function getClientAddress() : string
```

This method returns the ip address of the client that issued the current request. (Key: `REMOTE_ADDR`)

<hr class="blockspace">

```php
public function getRequestTimeFloat() : float
```

This method returns the timestamp when the current request hit the server. (Key: `REQUEST_TIME_FLOAT`)

<hr class="blockspace">

```php
public function getAcceptedContentTypes() : string
```

This method returns the contents of the _Accept_ header of the current request, if provided. (Key: `HTTP_ACCEPT`)

<hr class="blockspace">

```php
public function getQueryString() : string
```

This method returns the query string of the current request. (Key: `QUERY_STRING`)

<hr class="blockspace">

```php
public function getReferer() : string
```

This method returns the address of the page (if any) which referred the user agent to the current page. (Key: `HTTP_REFERER`)

<hr class="blockspace">

```php
public function getAuthUser() : string
```

This method returns the provided username when doing HTTP authentication. (Key: `PHP_AUTH_USER`)

<hr class="blockspace">

```php
public function getAuthPassword() : string
```

This method returns the provided password when doing HTTP authentication. (Key: `PHP_AUTH_PW`)

<hr class="blockspace">

```php
public function getContentType() : string
```

This method returns the provided content type header on a POST or PUT request. (Key: `CONTENT_TYPE`)

<hr class="blockspace">

```php
public function getContentLength() : string
```

This method returns the provided content length header on a POST or PUT request. (Key: `CONTENT_LENGTH`)

<hr class="blockspace">

## Annotation

We are aware, that not all `$_SERVER` indices were considered in the current release. 
And we also know that there is a method missing to access custom indices in the $_SERVER variable. 
We will fix this with the `v2.1.0` milestone release.

* [Issue: Support ALL $_SERVER indices in RequestInfo](https://github.com/icehawk/icehawk/issues/17)
* [Issue: Provide a getter to allow access to custom keys in RequestInfo / $_SERVER](https://github.com/icehawk/icehawk/issues/15)
