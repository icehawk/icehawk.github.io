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
	public function newWithOverwrites( array $array ) : ProvidesRequestInfo;
    
	public function getArgv() : array;

	public function getArgc() : int;

	public function isSecure() : bool;

	public function getMethod() : string;

	public function getUri() : string;

	public function getHost() : string;

	public function getConnection() : string;

	public function getUserAgent() : string;

	public function getServerAddress() : string;

	public function getClientAddress() : string;

	public function getClientHost() : string;

	public function getClientPort() : string;

	public function getClientUser() : string;

	public function getRedirectClientUser() : string;

	public function getRequestTimeFloat() : float;

	public function getRequestTime() : string;

	public function getAcceptedContentTypes() : string;

	public function getAcceptedCharsets() : string;

	public function getAcceptedEncoding() : string;

	public function getAcceptedLanguage() : string;

	public function getQueryString() : string;

	public function getReferer() : string;

	public function getAuthType() : string;

	public function getAuthDigest() : string;

	public function getAuthUser() : string;

	public function getAuthPassword() : string;

	public function getContentType() : string;

	public function getContentLength() : string;

	public function getPhpSelf() : string;

	public function getGatewayInterface() : string;

	public function getServerName() : string;

	public function getServerSoftware() : string;

	public function getServerProtocol() : string;

	public function getServerAdmin() : string;

	public function getServerPort() : string;

	public function getServerSignature() : string;

	public function getPathTranslated() : string;

	public function getDocumentRoot() : string;

	public function getScriptName() : string;

	public function getScriptFilename() : string;

	public function getPathInfo() : string;

	public function getOriginalPathInfo() : string;

	public function getCustomValue( string $key ) : string;
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
* The keys are accessed case-sensitive and therefor need to be _UPPERCASE_, except `argc`, `argv` and optional custom keys.

**Used keys:**

* `argc`
* `argv`
* `AUTH_TYPE`
* `CONTENT_LENGTH`
* `CONTENT_TYPE`
* `DOCUMENT_ROOT`
* `GATEWAY_INTERFACE`
* `HTTP_ACCEPT`
* `HTTP_ACCEPT_CHARSET`
* `HTTP_ACCEPT_ENCODING`
* `HTTP_ACCEPT_LANGUAGE`
* `HTTP_CONNECTION`
* `HTTP_HOST`
* `HTTP_REFERER`
* `HTTP_USER_AGENT`
* `HTTPS`
* `ORIG_PATH_INFO`
* `PATH_INFO`
* `PATH_TRANSLATED`
* `PHP_AUTH_DIGEST`
* `PHP_AUTH_PW`
* `PHP_AUTH_USER`
* `PHP_SELF`
* `QUERY_STRING`
* `REDIRECT_REMOTE_USER`
* `REMOTE_ADDR`
* `REMOTE_HOST`
* `REQUEST_METHOD`
* `REMOTE_PORT`
* `REMOTE_USER`
* `REQUEST_TIME`
* `REQUEST_TIME_FLOAT`
* `REQUEST_URI`
* `SCRIPT_NAME`
* `SCRIPT_FILENAME`
* `SERVER_ADDR`
* `SERVER_ADMIN`
* `SERVER_NAME`
* `SERVER_PORT`
* `SERVER_PROTOCOL`
* `SERVER_SIGNATURE`
* `SERVER_SOFTWARE`
* Optional custom keys

Please see the official [PHP documentation](http://php.net/manual/en/reserved.variables.server.php) for more information about the `$_SERVER` indices.

<hr class="blockspace">

## What do you get from the methods?

```php
public function newWithOverwrites( array $array ) : ProvidesRequestInfo
```

This method allows to create a new instance based on an existing request info object by overriding parts/all of its values. 
Please note that the request info object is meant to be immutable, that's why you cannot change its values directly. 

<hr class="blockspace">

```php
public function getArgv() : array
```

Returns an array of command line interface arguments, if any provided. (Only when run on CLI)

<hr class="blockspace">

```php
public function getArgc() : int
```

Returns the number of command line interface arguments, if any provided. (Only when run on CLI) 

<hr class="blockspace">

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

Returns the HTTP method (or verb) of the current request. (Key: `REQUEST_METHOD`)
 
**Please note:**
 
* The returned string is always _UPPERCASE_.
* The IceHawk component ships with an abstract class containing all HTTP methods (or verbs) as class constants: [HttpMethod](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Constants/HttpMethod.php). You can use this class constants to check against the results of the `RequestInfo::getMethod()`.
* The IceHawk component decides whether to do read or write handling based on the result of this method. (So it'd better be correct!)

<hr class="blockspace">

```php
public function getUri() : string
```

Returns the current request URI _without_ the query string. (Key: `REQUEST_URI`)

**Please note:**

* This method strips the query string part of the request URI. Use `getQueryString()` to retrieve it.
* The patterns of your routes is matched against the result of this method. See the [routing section](@baseUrl@/docs/icehawk/routing.html) for more information.
 
<hr class="blockspace">

```php
public function getHost() : string
```

Returns the current HTTP hostname. (Key: `HTTP_HOST`)

For a request on this page, it should return: `icehawk.github.io`.
 
<hr class="blockspace">

```php
public function getConnection() : string
```

Returns the contents of the `Connection:` header from the current request, if there is one. Example: `Keep-Alive`. (Key: `HTTP_CONNECTION`)
 
<hr class="blockspace">

```php
public function getUserAgent() : string
```

Returns the client's user agent string. (Key: `HTTP_USER_AGENT`)

A request from a Google Chrome web browser for example can result in: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36`.

<hr class="blockspace">

```php
public function getServerAddress() : string
```

Returns the ip address of the server handling the current request. (Key: `SERVER_ADDR`)

<hr class="blockspace">

```php
public function getClientAddress() : string
```

Returns the ip address of the client that issued the current request. (Key: `REMOTE_ADDR`)

<hr class="blockspace">

```php
public function getClientHost() : string
```

Returns the hostname from which the user is viewing the current page. The reverse dns lookup is based off the REMOTE_ADDR of the user. (Key: `REMOTE_HOST`)

**Note:** Your web server must be configured to create this variable. For example in Apache you'll need `HostnameLookups On` inside `httpd.conf` for it to exist.

<hr class="blockspace">

```php
public function getClientPort() : string
```

Returns the port being used on the user's machine to communicate with the web server. (Key: `REMOTE_PORT`)

<hr class="blockspace">

```php
public function getClientUser() : string
```

Returns the authenticated user. (Key: `REMOTE_USER`)

<hr class="blockspace">

```php
public function getRedirectClientUser() : string
```

Returns the authenticated user if the request is internally redirected. (Key: `REDIRECT_REMOTE_USER`)

<hr class="blockspace">

```php
public function getRequestTime() : string
```

Returns the timestamp when the current request hit the server. (Key: `REQUEST_TIME`)

<hr class="blockspace">

```php
public function getRequestTimeFloat() : float
```

Returns the timestamp when the current request hit the server as floating point number. (Key: `REQUEST_TIME_FLOAT`)

<hr class="blockspace">

```php
public function getAcceptedContentTypes() : string
```

Returns the contents of the _Accept_ header of the current request, if provided. (Key: `HTTP_ACCEPT`)

<hr class="blockspace">

```php
public function getAcceptedCharsets() : string
```

Returns the contents of the `Accept-Charset:` header from the current request, if there is one. Example: `iso-8859-1,*,utf-8`. (Key: `HTTP_ACCEPT_CHARSET`)

<hr class="blockspace">

```php
public function getAcceptedEncoding() : string
```

Returns the contents of the `Accept-Encoding:` header from the current request, if there is one. Example: `gzip`. (Key: `HTTP_ACCEPT_ENCODING`)

<hr class="blockspace">

```php
public function getAcceptedLanguage() : string
```

Returns the contents of the `Accept-Language:` header from the current request, if there is one. Example: `en`. (Key: `HTTP_ACCEPT_LANGUAGE`)

<hr class="blockspace">

```php
public function getQueryString() : string
```

Returns the query string of the current request. (Key: `QUERY_STRING`)

<hr class="blockspace">

```php
public function getReferer() : string
```

Returns the address of the page (if any) which referred the user agent to the current page. (Key: `HTTP_REFERER`)

<hr class="blockspace">

```php
public function getAuthType() : string
```

Returns the type of HTTP authentication. (Key: `AUTH_TYPE`)

<hr class="blockspace">

```php
 public function getAuthDigest() : string
```

When doing Digest HTTP authentication this variable is set to the `Authorization:` header sent by the client 
(which you should then use to make the appropriate validation). (Key: `PHP_AUTH_DIGEST`)

<hr class="blockspace">

```php
public function getAuthUser() : string
```

Returns the provided username when doing HTTP authentication. (Key: `PHP_AUTH_USER`)

<hr class="blockspace">

```php
public function getAuthPassword() : string
```

Returns the provided password when doing HTTP authentication. (Key: `PHP_AUTH_PW`)

<hr class="blockspace">

```php
public function getContentType() : string
```

Returns the provided content type header on a POST or PUT request. (Key: `CONTENT_TYPE`)

<hr class="blockspace">

```php
public function getContentLength() : string
```

Returns the provided content length header on a POST or PUT request. (Key: `CONTENT_LENGTH`)

<hr class="blockspace">

```php
public function getPhpSelf() : string
```

Returns the filename of the currently executing script, relative to the document root. 
For instance `/foo/bar.php` for a script at the address `http://example.com/foo/bar.php`. (Key: `PHP_SELF`)

<hr class="blockspace">

```php
public function getGatewayInterface() : string
```

Returns what revision of the CGI specification the server is using. Example `FastCGI/1.0`. (Key: `GATEWAY_INTERFACE`)

<hr class="blockspace">

```php
public function getServerName() : string
```

Returns the name of the server host under which the current script is executing. (Key: `SERVER_NAME`)

<hr class="blockspace">

```php
public function getServerSoftware() : string
```

Returns the server identification string, given in the headers when responding to requests. (Key: `SERVER_SOFTWARE`)

<hr class="blockspace">

```php
public function getServerProtocol() : string
```

Returns the name and revision of the information protocol via which the page was requested. Example: `HTTP/1.1` (Key: `SERVER_PROTOCOL`)

<hr class="blockspace">

```php
public function getServerAdmin() : string
```

Returns the value given to the SERVER_ADMIN (for Apache) directive in the web server configuration file. If the script is running on a virtual host, 
this will be the value defined for that virtual host. (Key: `SERVER_ADMIN`)

<hr class="blockspace">

```php
public function getServerPort() : string
```

Returns the port on the server machine being used by the web server for communication. For default setups, 
this will be `80`; using SSL, for instance, will change this to whatever your defined secure HTTP port is. (Key: `SERVER_PORT`)

<hr class="blockspace">

```php
public function getServerSignature() : string
```

Returns a string containing the server version and virtual host name which are added to server-generated pages, if enabled. (Key: `SERVER_SIGNATURE`)

<hr class="blockspace">

```php
public function getPathTranslated() : string
```

Returns the Filesystem- (not document root-) based path to the current script, after the server has done any virtual-to-real mapping. (Key: `PATH_TRANSLATED`)

<hr class="blockspace">

```php
public function getDocumentRoot() : string
```

Returns the document root directory under which the current script is executing, as defined in the server's configuration file. (Key: `DOCUMENT_ROOT`)

<hr class="blockspace">

```php
public function getScriptName() : string
```

Returns the current script's path. This is useful for pages which need to point to themselves. (Key: `SCRIPT_NAME`)

<hr class="blockspace">

```php
public function getScriptFilename() : string
```

Returns the absolute pathname of the currently executing script. (Key: `SCRIPT_FILENAME`)

<hr class="blockspace">

```php
public function getPathInfo() : string
```

Returns any client-provided pathname information trailing the actual script filename but preceding the query string, if available. 
Example: `/some/stuff` for `http://www.example.com/php/path_info.php/some/stuff?foo=bar` (Key: `PATH_INFO`)

<hr class="blockspace">

```php
public function getOriginalPathInfo() : string
```

Returns the original version of `PATH_INFO` before processed by PHP. (Key: `PATH_INFO`)

<hr class="blockspace">

```php
public function getCustomValue( string $key ) : string;
```

Returns an optionally user-defined value. (Key: `<custom>`)
