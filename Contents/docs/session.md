# Session component

This component is intended to wrap the super-global `$_SESSION` variable and give 
access to values by explicitly user-defined keys and value/return types 
and is therefor declared abstract.

Furthermore it provides the registration and interfaces for data mappers on all, several or single keys, e.g. to reduce data overhead stored in session.

<hr class="blockspace">

## Extend the class
 
```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\Session\AbstractSession;

final class Session extends AbstractSession
{
    const KEY_SOME_STRING_VALUE = 'someStringValue';
    
    public function getSomeStringValue() : string
    {
        return $this->get( self::KEY_SOME_STRING_VALUE );
    }

    public function setSomeStringValue( string $value )
    {
        $this->set( self::KEY_SOME_STRING_VALUE, $value );
    }

    public function isSomeStringValueSet() : bool
    {
        return $this->isset( self::KEY_SOME_STRING_VALUE );
    }

    public function unsetSomeStringValue()
    {
        $this->unset( self::KEY_SOME_STRING_VALUE );
    }
}
```

<hr class="blockspace">

## Work with values

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

$session = new Session( $_SESSION );

# Set
$session->setSomeStringValue( 'Hello world' );

# Isset
if ( $session->isSomeStringValueSet() )
{
    # Get
    $someString = $session->getSomeStringValue();
    
    echo $someString . ' was set.';
}

# Unset 
$session->unsetSomeStringValue();
```

<hr class="blockspace">

## Data mapping

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

use IceHawk\Session\Interfaces\MapsSessionData;

# Create a data mapper class
final class MyDataMapper implements MapsSessionData
{
	public function toSessionData( $value ) 
	{
        return base64_encode( $value );
	}
	
	public function fromSessionData( $sessionData ) 
	{
		return base64_decode( $sessionData );
	}
}

$session = new Session( $_SESSION );

# Add the data mapper for all keys in the registry
$session->addDataMapper( new MyDataMapper() );

# Add the data mapper for one specific key in the registry
$session->addDataMapper( new MyDataMapper(), [Session::KEY_SOME_STRING_VALUE] );

# Add the data mapper for multiple keys in the registry
$session->addDataMapper( new MyDataMapper(), [Session::KEY_SOME_STRING_VALUE, Session::KEY_SOME_OTHER_VALUE] );
```

- The data mapper's `toSessionData()` is called when the `AbstractSesion::set()` method gets invoked.
- The data mapper's `fromSessionData()` is called when the `AbstractSesion::get()` method gets invoked.

<hr class="blockspace">

## Clear all session data

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

$session = new Session( $_SESSION );

# ... put some data to the session ...

# Clear the session data
$session->clear();
```
