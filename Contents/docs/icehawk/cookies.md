# Cookies
 
This section covers the accessability of values set to cookies.

<hr class="blockspace">

## The Cookies object

Since version 2.1.0 IceHawk provides a wrapper object for cookie values via `$request` data object. This wrapper is defined by the interface
`IceHawk\IceHawk\Interfaces\ProvidesCookieData` and implemented as the default class 
[Cookies](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Defaults/Cookies.php).

As you can see in the [request handlers section](@baseUrl@/docs/icehawk/request-handlers.html) the IceHawk component provides a request data object to the request handler.
This object carries the [request information](@baseUrl@/docs/icehawk/request-information.html), [request input](@baseUrl@/docs/icehawk/request-input-data.html) and the cookie data.

This example shows how you can access the cookie data inside a GET request handler:

```php
class YourGetRequestHandler implements HandlesGetRequest
{
	public function handle( ProvidesReadRequestData $request )
	{
		# Get the cookie data
		$requestInput = $request->getCookies();
	}
}
```

<hr class="blockspace">

## Where to instantiate the object?

As shown in the [configuration section](@baseUrl@/docs/icehawk/configuration.html) you need to provide an instance of the `Cookies` class 
(or an implementation of the `ProvidesCookieData` interface) once in the `getCookies()` method. IceHawk will then inject it to all subsequent objects.

<hr class="blockspace">

### 1. Get a single value

Assuming your cookie data array looks like this:

```php
Array
(
    [language] => en
    [lastPage] =>
)
```

You can now access these values by the `get(string $key)` method like this:

```php
<?php

$language = $cookies->get( 'language' );
var_dump( $language );

$lastPage = $cookies->get( 'lastPage' );
var_dump( $lastPage );

$affiliate = $cookies->get( 'affiliate' ); # key doesn't exist!
var_dump( $affiliate );

# key doesn't exist, use FALSE as default value
$affiliate = $cookies->get( 'affiliate', false ); 
var_dump( $affiliate );
```

This will print:

```php
string(2) "en"          # $language

string(0) ""            # $lastPpage

NULL                    # $affiliate

FALSE                   # $affiliate
```

<hr class="blockspace">

### 2. Get all values

Assuming your cookie data array looks like this:

```php
Array
(
    [language] => en
    [lastPage] =>
)
```

You can now access these values by the `getData()` method like this:

```php
<?php

var_dump($cookies->getData());
```

This prints:

```php
array(2) {
  'language' => string(2) "en"
  'lastPage' => string(0) ""
}
```
