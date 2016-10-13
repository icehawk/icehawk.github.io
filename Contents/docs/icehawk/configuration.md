# IceHawk configuration

This documentation shows you which configuration abilities IceHawk offers.

<hr class="blockspace">

## Defaults

IceHawk makes use of a ["traitful config"](https://phpind.de/posts/traitful-configs?locale=en_US) and offers an interface for its configuration, 
which can be implemented by simply using several traits covering all interface methods.

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

By the way, this is (pretty much) the [_IceHawkConfig.php_](https://github.com/icehawk/icehawk/blob/v2.0.0/src/Defaults/IceHawkConfig.php) that the IceHawk component already ships.

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
   
<hr class="blockspace">

## Get read routes

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

## Get write routes

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
