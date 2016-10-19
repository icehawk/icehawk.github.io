# IceHawk routing

This documentation shows you the routing abilities of the IceHawk component.
 
<div class="alert alert-warning">
<b>Please note:</b> The routing of read requests and write requests is equally implemented. 
That's why we will only describe how to configure read routes here. 
Just replace the <code>*Read*</code> classnames with <code>*Write*</code> classnames and you'll do write request routing.
</div>

<hr class="blockspace">

## Routing by URI only

The routing is done by matching patterns against the current request URI. 
The URI is provided by the [request info object](/docs/icehawk/request-information.html).

That means you can not (directly) match against the server/host name or a query string.  

<hr class="blockspace">

## How the URI is provided

Almost every application has a "Home(page)" represented by a root URI like `/`. 
The IceHawk component makes sure that you'll always get at least a `/` (slash) 
as the requested URI, even if it is not part of the requested URL.

That means:

* For the URL https://www.your-domain.com IceHawk provides the URI `/`.
* For the URL https://www.your-domain.com/ IceHawk provides the URI `/`.

So you always have a reliable root URI string.

Of course your "Home(page)" must not be represented by the URI `/`, but if you have a base path 
for all your URIs like `/shop/` IceHawk will give you the URI as is.

That means:

* For the URL https://www.your-domain.com/shop IceHawk provides the URI `/shop`.
* For the URL https://www.your-domain.com/shop/ IceHawk provides the URI `/shop/`.

If you always want trailing slashes in your URI you need to take care of that yourself, e.g. by adding an appropriate rewrite rule in your webserver's host config or `.htaccess` file.

<hr class="blockspace">

## What is a route?

A route basically consists of an URI pattern instance and an request handler instance.

A pattern is expressed as an object implementing the interface `IceHawk\IceHawk\Routing\Interfaces\ProvidesMatchResult`.

A request handler is expressed as an object implementing one of the following interfaces, each representing the allowed request method(s):

| Request type | Request method(s)   | Interface                                          |
|--------------|---------------------|----------------------------------------------------|
| read         | GET **incl.** HEAD  | `IceHawk\IceHawk\Interfaces\HandlesGetRequest`     |
| write        | POST                | `IceHawk\IceHawk\Interfaces\HandlesPostRequest`    |
| write        | PUT                 | `IceHawk\IceHawk\Interfaces\HandlesPutRequest`     |
| write        | PATCH               | `IceHawk\IceHawk\Interfaces\HandlesPatchRequest`   |
| write        | DELETE              | `IceHawk\IceHawk\Interfaces\HandlesDeleteRequest`  |
_&dash; request method interfaces_ 


A route then is expressed as an object implementing on of the following interfaces: 

* For read rouutes: `IceHawk\IceHawk\Routing\Interfaces\RoutesToReadHandler` 
* For write routes: `IceHawk\IceHawk\Routing\Interfaces\RoutesToWriteHandler`

The IceHawk component ships with two appropriate ready-to-use classes: 

* [_ReadRoute_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/ReadRoute.php) 
* [_WriteRoute_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/WriteRoute.php) 

We also ship the following ready-to-use classes for URI patterns:

* [_Literal_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/Patterns/Literal.php)
* [_RegExp_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/Patterns/RegExp.php)
* [_NamedRegExp_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/Patterns/NamedRegExp.php)

We will discuss them later in this documentation.

So the most simple route would look like this:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesGetRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;
use IceHawk\IceHawk\Routing\Patterns\Literal;
use IceHawk\IceHawk\Routing\ReadRoute;

$uriPattern = new Literal( '/' );
$requestHandler = new class implements HandlesGetRequest
{
	public function handle( ProvidesReadRequestData $request ) 
	{
	    echo "Hello World!";
	}	
};

$mostSimpleRoute = new ReadRoute( $uriPattern, $requestHandler );
```

**Please note:** The request handler is implemented as an anonymous class here for better readability of the implemented interface. In most cases you would implement a real class of course. 
