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
The URI is provided by the [request info object](@baseUrl@/docs/icehawk/request-information.html).

That means you can not (directly) match against the server/host name or a query string.  

<hr class="blockspace">

## How is the URI provided?

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

A route basically consists of a URI pattern instance and a request handler instance.

A pattern is expressed as an object implementing the interface `IceHawk\IceHawk\Routing\Interfaces\ProvidesMatchResult`.

A request handler is expressed as an object implementing one of the following interfaces, each representing the allowed request method(s):

| Request type | Request method(s)   | Interface                                          |
|--------------|---------------------|----------------------------------------------------|
| read         | GET **incl.** HEAD  | `IceHawk\IceHawk\Interfaces\HandlesGetRequest`     |
| read         | HEAD                | `IceHawk\IceHawk\Interfaces\HandlesHeadRequest`    |
| write        | POST                | `IceHawk\IceHawk\Interfaces\HandlesPostRequest`    |
| write        | PUT                 | `IceHawk\IceHawk\Interfaces\HandlesPutRequest`     |
| write        | PATCH               | `IceHawk\IceHawk\Interfaces\HandlesPatchRequest`   |
| write        | DELETE              | `IceHawk\IceHawk\Interfaces\HandlesDeleteRequest`  |

A route then is expressed as an object implementing one of the following interfaces: 

* For read rouutes: `IceHawk\IceHawk\Routing\Interfaces\RoutesToReadHandler` 
* For write routes: `IceHawk\IceHawk\Routing\Interfaces\RoutesToWriteHandler`

The IceHawk component provides two appropriate ready-to-use classes: 

* [_ReadRoute_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/ReadRoute.php) 
* [_WriteRoute_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/WriteRoute.php) 

We also provide the following ready-to-use classes for URI patterns:

* [_Literal_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/Patterns/Literal.php)
* [_RegExp_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/Patterns/RegExp.php)
* [_NamedRegExp_](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/Patterns/NamedRegExp.php)

We will discuss them later in this section.

<hr class="blockspace">

## What is the most simple route?

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

<hr class="blockspace">

## Where to place routes?

As shown in the [configuration section](@baseUrl@/docs/icehawk/configuration.html) you need to configure and return your routes in 
the `getReadRoutes()` / `getWriteRoutes()` methods of the IceHawk config class.

Here is an example:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Interfaces\RoutesToReadHandler;
use IceHawk\IceHawk\Routing\ReadRoute;
use IceHawk\IceHawk\Routing\Patterns\Literal;
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
			
		];
	}
}
```

As you may noticed, we omitted the return type for this method, because we wanted to allow building a `\Generator` and `yield`ing the routes.  
In the next major release targeting PHP 7.1 we'll add the [iterable](https://github.com/php/php-src/pull/1941) return type.

This simple example shows how to provide routes by simply returning an array of route instances.
This is pretty OK for small applications with a small number of routes. In case you have a large number of routes you can source them out to 
a config file and only `yield` the current loop instance. That's why we didn't add a return type to the `getReadRoutes()` / `getWriteRoutes()` methods.

The following example shows how to `yield` route instances, defined in a config file:

### 1. The routes config file

```php
<?php declare(strict_types=1);

return [
	'/'         => YourVendor\YourProject\Application\Endpoints\ShowHomeRequestHandler::class,
	'/product'  => YourVendor\YourProject\Application\Endpoints\ShowProductRequestHandler::class,
	'/gallery'  => YourVendor\YourProject\Application\Endpoints\ShowGalleryRequestHandler::class,
	'/imprint'  => YourVendor\YourProject\Application\Endpoints\ShowImprintRequestHandler::class,
	'/privacy'  => YourVendor\YourProject\Application\Endpoints\ShowPrivacyRequestHandler::class,
];
```

### 2. The IceHawk Config

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

# ...

class IceHawkConfig implements ConfiguresIceHawk
{
	# ...
	
	/**
	 * @return array|\Traversable|RoutesToReadHandler[]
	 */
	public function getReadRoutes()
	{
		$routeDefinitions = require( 'ReadRoutes.php' );
		
		foreach ( $routeDefinitions as $uriPattern => $requestHandlerClass )
		{
			yield new ReadRoute( new Literal( $uriPattern ), new $requestHandlerClass() );
		}
	}
}
```

The advantage of this approach is that not all routes, patterns and request handlers will be instantiated on bootstrap and that only the matching route instance will remain.

<hr class="blockspace">

## What patterns to use for matching?

As already mentioned above, the IceHawk component provides 3 ready-to-use pattern classes. We'd like to show you examples for each of them.

A pattern class needs to implement the interface `IceHawk\IceHawk\Routing\Interfaces\ProvidesMatchResult`, that looks like this:

```php
<?php declare(strict_types=1);

namespace IceHawk\IceHawk\Routing\Interfaces;

interface ProvidesMatchResult
{
	public function matches( string $other ) : bool;
	
	public function getMatches() : array;
}
```

The `matches()` method is called with the current URI as the only parameter and shall return whether the URI matches or not.
 
The `getMatches()` method shall return any values extracted from the URI as an array, if any. These values will then be merged with the current request values.

So it is pretty simple to implement an own pattern class. This example shows how the `Literal` pattern class would be implemented as an anonymous class:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Routing\Interfaces\ProvidesMatchingResult;

$literalPattern = new class implements ProvidesMatchingResult
{
	private $uri;

	public function __construct( string $uri ) 
	{
		$this->uri = $uri;
	}

	public function matches( string $other ) : bool
	{
		return ($other == $this->uri);		
	}
	
	public function getMatches() : array
	{
		return [];
	}
};
```

### 1. The Literal pattern

The `Literal` pattern matches - like the name says - a URI literally. So you have no option for placeholders or regular expressions here.

```php
<?php declare(strict_types=1);

$uri = '/product';

$pattern = new Literal( '/product' );

echo $pattern->matches( $uri ) ? 'Pattern matched' : 'Pattern not matched';

# Prints: Pattern matched

# Note: The Literal pattern compares case sensitive!

echo $pattern->matches( '/Product' ) ? 'Pattern matched' : 'Pattern matched not';

# Prints: Pattern matched not
```

### 2. The RegExp pattern

The `RegExp` pattern matches against the URI with a user-provided full regular expression and enables the user to map the regular expression matches to names.
 
```php
<?php declare(strict_types=1);

$uri = '/product/98361723';

$pattern = new RegExp( '#^/product/([0-9]+)$#', [ 'productId' ] );

echo $pattern->matches( $uri ) ? 'Pattern matched' : 'Pattern not matched';
print_r( $pattern->getMatches() );

# Prints:
# Pattern matched
# array( productId => 98361723 )
```

**Please note:** 

* The first match of the regular expression (at index 0) is always the full URI, which is useless and therefor is removed before the mapping takes place.
* The names are mapped in the order of matches from the regular expression (from left to right).
* You can have fewer names than matches in the regular expression.
* Only mapped values will be merged with the current request data.
* You can use any regular expression that can be interpreted as a pattern by [preg_match()](https://php.net/preg_match).
* The `RegExp` class makes sure the match is executed only once.

### 3. The NamedRegExp pattern
 
To avoid some redundant stuff when working with regular expression patterns for your routes, we also offer the `NamedRegExp` pattern class.
This one allows you to put the names for your variable values directly in the pattern string and you can omit the delimiters.


```php
<?php declare(strict_types=1);

$uri = '/product/98361723';

$pattern = new NamedRegExp( '^/product/(?<productId>[0-9]+)$' );

echo $pattern->matches( $uri ) ? 'Pattern matched' : 'Pattern not matched';
print_r( $pattern->getMatches() );

# Prints:
# Pattern matched
# array( productId => 98361723 )

# Note: You can add pattern string flags as the second parameter

$uri = '/Product/98361723';

$pattern = new NamedRegExp( '^/product/(?<productId>[0-9]+)$', 'i' ); # match case-insensitive!

echo $pattern->matches( $uri ) ? 'Pattern matched' : 'Pattern not matched';
print_r( $pattern->getMatches() );

# Prints:
# Pattern matched
# array( productId => 98361723 )
```

**Please note:**

* The `NamedRegExp` class uses the `!` (exclamation mark) as the delimiter for the pattern string. So the last example would be concatenated to the following string: 
`"!^/product/(?<productId>[0-9]+)$!i"`.
* You can provide the regular expression flags as the second argument.
* The `NamedRegExp` class makes sure the match is executed only once.

<hr class="blockspace">

## Route groups

If you have a large number of routes in your project, it might be useful to group them by base paths.

The IceHawk component provides 2 ready-to-use classes to accomplish this approach:

* [ReadRouteGroup](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/ReadRouteGroup.php)
* [WriteRouteGroup](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Routing/WriteRouteGroup.php)

Like `ReadRoute` and `WriteRoute` these classes also implement the interfaces `IceHawk\IceHawk\Routing\Interfaces\RoutesToReadHandler` / `IceHawk\IceHawk\Routing\Interfaces\RoutesToWriteHandler`.
So they can simply replace a normal route instance.

This example shows how to group multiple read routes:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

# ...

class IceHawkConfig implements ConfiguresIceHawk
{
	# ...
	
	/**
	 * @return array|\Traversable|RoutesToReadHandler[]
	 */
	public function getReadRoutes()
	{
		return [
			new ReadRouteGroup(
				new RegExp('#^/product#'),
				[
					new ReadRoute( 
						new RegExp('#/advisor$#'), 
						new ShowProductAdvisorRequestHandler()
					),
					new ReadRoute( 
						new NamedRegExp('/(?<productId>[0-9]+)$'), 
						new ShowProductRequestHandler()
					),
					new ReadRoute( 
						new NamedRegExp('/(?<productId>[0-9]+)/(?<ean>[0-9]{13})$'), 
						new ShowProductDetailsRequestHandler()
					),
				]
			),
		];
	}
}
```

**Please note:** The pattern strings in the subsequent read routes are relative to the group's pattern string, to avoid redundancy.

### Get matches out of a route group

Depending on the pattern you use to match the base path of a group, you can also get values out of this base path.

This example shows how these values get merged:

```php
<?php declare(strict_types=1):

# We want to read the file "profile.png" of user with ID "4711"

$uri = '/user/4711/files/profile.png';

$userRouteGroup = new ReadRouteGroup(
	new NamedRegExp( '^/user/(?<userId>[0-9]+)' ),
	[
		new ReadRoute(
			new NamedRegExp( '/files/(?<fileName>[^/]+)$' ),
			new ShowUserFileRequestHandler()
		),
		
		# ...
	]
);

echo $userRouteGroup->matches( $uri ) ? 'Pattern matched' : 'Pattern matched not';
print_r( $userRouteGroup->getUriParams() );

# Prints:
# Pattern matched
# array( userId => 4711, fileName => profile.png )
```

<hr class="blockspace">

## Generate route groups

The approach of `yield`ing route instances also works for route groups. The following example shows how. 
To make the configuration as easy as possible we'll use always the `NamedRegExp` pattern class.

### 1. The routes config file

```php
<?php declare(strict_types=1);

return [
	'^/user/(?<userId>[0-9]+)'  => [
		'/files/(?<fileName>[^/]+)$' => YourVendor\YourProject\ShowUserFileRequestHandler::class,
		'/orders/(?<orderId>[0-9]+)$' => YourVendor\YourProject\ShowUserOrderRequestHandler::class,
	],
];
```

### 2. The IceHawk Config

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

# ...

class IceHawkConfig implements ConfiguresIceHawk
{
	# ...
	
	/**
	 * @return array|\Traversable|RoutesToReadHandler[]
	 */
	public function getReadRoutes()
	{
		$routeGroups = require( 'ReadRoutes.php' );
		
		foreach ( $routeGroups as $groupPattern => $routeDefinitions )
		{
			$routeGroup = new ReadRouteGroup( new NamedRegExp( $groupPattern ) );
			
			foreach ($routeDefinitions as $routePattern => $requestHandlerClass )
			{
				$routeGroup->addRoute( 
					new ReadRoute(
						new NamedRegExp( $routePattern ),
						new $requestHandlerClass()
					)
				);
			}
			
			yield $routeGroup;
		}
	}
}
```
