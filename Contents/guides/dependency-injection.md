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
We tried to find a short name and simply called it `Env`, because from a business domain perspective that's what we do - we inject infrastructure from the environment.

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

Therefor we need to add the container object as a constructor parameter to the IceHawk config. Since the IceHawk config or its interface does 
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

<hr class="blockspace">

## Request handler base class(es)

Depending on your choice how you want to access the container inside a request handler, there are at least three ways of implementation.

### 1. Add container injection to every request handler

The easiest variant is obviously to give each request handler a contructor that gets passed the container object and the usage via private member access.

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesGetRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

final class ListUsersRequestHandler implements HandlesGetRequest
{
	/** @var Container */
	private $container;
	
	public function __construct( Container $container )
	{
		$this->container = $container;
	}
	
	public function handle( ProvidesReadRequestData $request )
	{
		$dbManager = $this->container->getDbManager();
		$userList  = $dbManager->query( "SELECT * FROM users WHERE 1" );
		 
		echo json_encode( $userList->fetchAll( \PDO::FETCH_ASSOC ), JSON_PRETTY_PRINT );
		flush();
	}
}
```

Adding a constructor like this to every request handler is a lot of code redundancy and definitely a violation of 
[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). We wouldn't recommend to do this.

### 2. Let the request handlers inherit from _one_ abstract class

The following second variant uses one abstract class to implement the container injection only once for all request handlers and 
gives access to the container via a final and protected getter.
 
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

abstract class AbstractRequestHandler
{
	/** @var Container */
	private $container;
	
	public function __construct( Container $container )
	{
		$this->container = $container;
	}
	
	final protected function getContainer() : Container
	{
		return $this->container;
	}
}
```

Now you can let all your request handlers extend this `AbstractRequestHandler`. 
**Note:** This variant works for both types of request handlers - read and write.
  
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesGetRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

final class ListUsersRequestHandler extends AbstractRequestHandler implements HandlesGetRequest
{
	public function handle( ProvidesReadRequestData $request )
	{
		$dbManager = $this->getContainer()->getDbManager();
		$userList  = $dbManager->query( "SELECT * FROM users WHERE 1" );
		 
		echo json_encode( $userList->fetchAll( \PDO::FETCH_ASSOC ), JSON_PRETTY_PRINT );
		flush();
	}
}
```

So all you need to add to your request handler is an `extends AbstractRequestHandler`.

<hr class="blockspace">

### 3. Add abstract request handlers for each side

This third variant adds an abstract request handler for read and the write request handlers and passes the container object to a `handleRequest()` method.

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

abstract class AbstractReadRequestHandler implements HandlesReadRequest
{
	/** @var Container */
	private $container;
	
	public function __construct( Container $container )
	{
		$this->container = $container;
	}
	
	public function handle( ProvidesReadRequestData $request )
	{
		$this->handleRequest( $request, $this->container );
	}
	
	abstract public function handleRequest( ProvidesReadRequestData $request, Container $container );
}
```

Now the request handler would look like this:

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesGetRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

final class ListUsersRequestHandler extends AbstractReadRequestHandler implements HandlesGetRequest
{
	public function handleRequest( ProvidesReadRequestData $request, Container $container )
	{
		$dbManager = $container->getDbManager();
		$userList  = $dbManager->query( "SELECT * FROM users WHERE 1" );
		 
		echo json_encode( $userList->fetchAll( \PDO::FETCH_ASSOC ), JSON_PRETTY_PRINT );
		flush();
	}
}
```

Even though this variant probably offers the best readability, it has some downsides:
 
* You need to add two abstract classes containing almost the same code
* The previous intended interface of the request handlers change: `handle()` vs. `handleRequest()`. 

You choose! ;)

<hr class="blockspace">

## Further injection

In the same way you brought the container into the request handlers, you now can inject it to all the other application specific objects 
in the [IceHawk config](@baseUrl@/docs/icehawk/configuration.html), such as event subscribers and final responders. If you need it.
