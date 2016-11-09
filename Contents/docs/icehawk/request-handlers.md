# IceHawk request handlers 

This section covers how to implement request handlers for the read and the write side.

<hr class="blockspace">

## Request handler interfaces

In order to implement a request handler for the IceHawk component you need to implement one or more of 
the following interfaces in your request handler class. These interfaces are more or less marker interfaces, all extending the base interfaces for 
handling a read or a write request.
 
Every marker interface defines what request method(s) are allowed for the implementing request handler class. 
So there is no need to add a request method string or constant to every defined route.
  
This is the full inheritance listing of the request handler interfaces:

```
- HandlesRequest (empty interface, may contain declarations in future releases)
  |
  |- HandlesReadRequest (defines the handle() method for read request handlers)
  |  |
  |  `- HandlesHeadRequest (marks a request handler to handle a HEAD request)
  |     |
  |     `- HandlesGetRequest (marks a request handler to handle a GET and a HEAD request)
  |
  `- HandlesWriteRequest (defines the handle() method for write request handlers)
     |
     |- HandlesPostRequest (marks a request handler to handle a POST request)
     |
     |- HandlesPutRequest (marks a request handler to handle a PUT request)
     |
     |- HandlesPatchRequest (marks a request handler to handle a PATCH request)
     |
     `- HandlesDeleteRequest (marks a request handler to handle a DELETE request)
```
 
These marker interfaces also allow the IceHawk component to respond automatically to an `OPTIONS` request, as it just needs
to check for the implemented interfaces of the request handlers the requested URI was mapped to. Note that both, read and write side, are considered before responding an `OPTIONS` request.
So you can have the same URL allowing for example a `GET` request on the read side and a `POST` request on the write side. 

You can read more about this in our [auto repsonding section](@baseUrl@/docs/icehawk/auto-responding.html).

<hr class="blockspace">

## Read side request handlers

For the read side HTTP offers only to verbs: `GET` and `HEAD`.
Where the `HEAD` request should be handled and responded to exectly the same as a `GET` request, except that the body is omitted in the response.

For this reason we decided to let `HandlesGetRequest` inherit from the `HandlesHeadRequest` interface. So if your request handler implements 
`HandlesGetRequest` it can respond to both, `GET` and `HEAD` requests.
 
Implenenting a handler for `GET` requests is pretty easy though:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesGetRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

final class ShowHomepageRequestHandler implements HandlesGetRequest
{
	public function handle( ProvidesReadRequestData $request )
	{
		# Read some data and respond
	}
}
```

As you can see the only contract for your request handler is to implement a handle method that gets passed the read request object. So the rest is up to you.

Another advantage of only implementing an interface instead of extending an abstract or base class is to have no dependencies to 
base logic or any other framework-tied implementation. So you can easily write own abstract/base classes, if needed, without to worry about 
inheritance / overwriting of framework code. [We discuss a use-case for abstract classes here.](@baseUrl@/docs/icehawk/dependency-injection.html)
  
For the sake of completeness, this is how a `HEAD`-only request handler is implemented:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesHeadRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

final class RessourceAvailableRequestHandler implements HandlesHeadRequest
{
	public function handle( ProvidesReadRequestData $request )
	{
		# Check for ressource existence and respond
	}
}
```
  
<hr class="blockspace">

## Write side request handlers

For the write side HTTP offers 4 verbs: `POST`, `PUT`, `PATCH` and `DELETE`. The most web applications use `PUT`/`PATCH`/`DELETE` very rarely or don't use them at all.
In most cases a remote write operation is represented by a `POST` request. Some webservers don't even support `PUT`, `PATCH` and `DELETE`.
  
Implementing a handler for each of the request methods is equal and as easy as the previous example for a `GET` request handler.

A `POST` request handler:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesPostRequest;
use IceHawk\IceHawk\Interfaces\ProvidesWriteRequestData;

final class PostNewCommentRequestHandler implements HandlesPostRequest
{
	public function handle( ProvidesWriteRequestData $request )
	{
		# Do some write actions here and respond
	}
}
```

A `PUT` request handler:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesPutRequest;
use IceHawk\IceHawk\Interfaces\ProvidesWriteRequestData;

final class AddRessourceRequestHandler implements HandlesPutRequest
{
	public function handle( ProvidesWriteRequestData $request )
	{
		# Do some write actions here and respond
	}
}
```

A `PATCH` request handler:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesPatchRequest;
use IceHawk\IceHawk\Interfaces\ProvidesWriteRequestData;

final class UpdateProfileRequestHandler implements HandlesPatchRequest
{
	public function handle( ProvidesWriteRequestData $request )
	{
		# Do some write actions here and respond
	}
}
```

A `DELETE` request handler:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesDeleteRequest;
use IceHawk\IceHawk\Interfaces\ProvidesWriteRequestData;

final class UnsubscribeNewsletterRequestHandler implements HandlesDeleteRequest
{
	public function handle( ProvidesWriteRequestData $request )
	{
		# Do some write actions here and respond
	}
}
```
