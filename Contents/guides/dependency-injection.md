# Dependency injection guide

This guide shows you how you _can_ implement a dependency injection container (DIC) with IceHawk.

**Please note:** This is just a proposal - we do it like this and have made good (or better - no bad) experiences.

<hr class="blockspace">

When you were reading the docs you may noticed that the IceHawk component does not support a generic dependency injection container per se.
We intentionally decided to design a framework that requires concrete implementation and tried to avoid as much magic as possible.

Offering a generic dependency injeciton container such as suggested by [PSR-11](https://github.com/container-interop/fig-standards/blob/master/proposed/container.md)
would be completely the opposite of what we tried to achieve - a typed object API for your project.

Furthermore we wanted to keep our business logic clean from infrastructure dependencies, so we applied the 
[dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle) to the business and data layer and injected a 
project-specific infrastructure container. The application then can access the needed infrastructure via typed getters on that container.
 
The container is designed as a variant of an object pool offering the ability to create shared instances without using singleton objects.

The names for such containers vary a lot nowadays: Dependency injection container, Service locator, Interop container and so on.
We tried to find a short name and simply called it `Env`, because from a business domain view that's what we do - we inject infrastructure from the environment.

As we don't want to confuse you about `Env` and `$_ENV` or whatever you think the word "Env" means, we'll simply use "container" in the following paragraphs.

<hr class="blockspace">

## The object pool

As a basis for our container we implemented a very simply object pool class, that allows us to create shared instances. This class looks like this:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourNamespace;

abstract class AbstractObjectPool
{
	/** @var array */
	private $pool;
	
	public function __construct() 
	{
		$this->pool = [];
	}
	
	final protected function getSharedObject( string $key, \Closure $createFunction )
	{
		if ( !isset($this->pool[$key]) )
		{
			$this->pool[$key] = $createFunction->call($this);
		}
		
		return $this->pool[$key];
	}
}
```

As you can see we initialize an empty object pool on construction and do not create any dependency objects, also we hide the untyped getter for new 
objects inside the abstract class, so we can provide public getters with explicit return type declarations in the final container implementation. 

Let's take the common use-case of creating a database manager to see how that works:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

final class Container extends AbstractObjectPool
{
	public function getDbManager() : \PDO
	{
		return $this->getSharedObject(
			'dbManager', 	               # The unique key for the object pool
			function ()                    # The \Closure to create the object if it's not already in the pool 
			{
				$pdo = new \PDO(
					'mysql:host=127.0.0.1;port=3306;dbname=database', 
					'user', 
					'password',
					[
						\PDO::ATTR_CURSOR                   => \PDO::CURSOR_FWDONLY,
                        \PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => true,
					]
				);
				
				return $pdo;
			}
		);
	}
}
```

For more complex configured objects you can of course delegate the construction (inside the closure) to an appropriate factory or builder class.
We omitted that here for better readability and a smaller scope to comprehend.

<hr class="blockspace">

## Injecting the container to request handlers

In the first place we will want to use the container in our request handlers where we call our buisiness logic.
As described [in the documentation](@baseUrl@/docs/icehawk/configuration.html) request handlers are constructed in the IceHawk configuration class.
 
So we will first inject our container into the IceHawk config. Let's do this in the `index.php`:
 
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

$container       = new Container();
$iceHawkConfig   = new IceHawkConfig( $container );
$iceHawkDelegate = new IceHawkDelegate();
$iceHawk         = new IceHawk( $iceHawkConfig, $iceHawkDelegate );

$iceHawk->init();
$iceHawk->handleRequest();
```

Therefor we need to add the container object as a custructor parameter to the IceHawk config. Since the IceHawk config or its interface does 
not define a constructor, we don't have to care about overwriting a base constructor.
 
The altered IceHawk config class  could then look like this:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourNamespace;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;

final class IceHawkConfig implements ConfiguresIceHawk
{
	/** @var Container */
	private $container;
	
	public function __construct( Container $container ) 
	{
		$this->container = $container;
	}
	
	# ...
}
```

Now we are able to pass the container object to our defined request handlers like this:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourNamespace;

use IceHawk\IceHawk\Interfaces\ConfiguresIceHawk;
use IceHawk\IceHawk\Routing\ReadRoute;
use IceHawk\IceHawk\Routing\Patterns\Literal;

final class IceHawkConfig implements ConfiguresIceHawk
{
	/** @var Container */
	private $container;

	# ...
	
	public function getReadRoutes()
	{
		return [
			new ReadRoute( 
				new Literal( '/users/list' ),
				new ListUsersRequestHandler( $this->container ) # container injection
			),
			
			# ...
		];
	}
}
```
