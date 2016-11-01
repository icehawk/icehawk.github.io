# IceHawk request data 

You may noticed that so far the separated handling of read and write side does not differ that much. 
But when it comes to request data, you'll see the big difference.

This documentation explains the way IceHawk provides you the request data on each side - read and write.

## Preamble

Before we discuss how to access request input data, lets briefly talk about data types in request input data.

### What data types do I get?

If we break down what kind of data types a request input value can be, we will result in the following three types:

* `string`: for all scalar values  
Yes, also a number (int or float) is a string in the query *string* in the first place. There is no such thing like autocasting request input values in PHP.
* `array`: for structured values  
An array is the only data structure that is automatically unpacked from a query string in PHP. All other structures (e.g. JSON) remain strings and must be unpacked manually.  
* `null`: for not existing keys/values  
Well, technically you can not submit `null` as a request input value, but it makes sense to use this "type" for not existing request input keys/values, 
because it should not be an empty string or array, which can be valid values for an existing key. 
We actually don't have any information about the value at all. So `null` is the absence of everything and indicates that the key and value you want to access do not exist.  

This behaviour is reflected by the basic method for accessing a request input value:

```php
/**
 * @param string            $key
 * @param string|array|null $default
 *
 * @return string|array|null
 */
public function get( string $key, $default = null )
{
	return $this->data[ $key ] ?? $default;
}
```

## The read request data

As you can see in the [request handlers section](@baseUrl@/docs/icehawk/request-handlers.html) the IceHawk component provides a request data object to the request handler.
This object carries the [request information](@baseUrl@/docs/icehawk/request-information.html) and the request input data.

This example shows how you can access the request input data inside a GET request handler:

```php
class YourGetRequestHandler implements HandlesGetRequest
{
	public function handle( ProvidesReadRequestData $request )
	{
		# Get the request input data
		$requestInput = $request->getInput();
	}
}
```

The `$requestInput` object retrieved in the example above is an instance of the 
[ReadRequestInput class](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Requests/ReadRequestInput.php).

This class is a wrapper of the superglobal `$_GET` variable and offers two ways to access its data:
 
### 1. Get a single value

Assuming we have the following query string:

```http
?galleryId=4711&filters[price]=19900&filters[weight]=2.5&page=
```

PHP will unpack this query string to the following `$_GET` array:

```php
Array
(
    [galleryId] => 4711
    [filters] => Array
        (
            [price] => 19900
            [weight] => 2.5
        )

    [page] =>
)
```

Now let's access the single values:

```php
<?php

$galleryId = $requestInput->get( 'galleryId' );
var_dump( $galleryId );

$filers = $requestInput->get( 'filters' );
var_dump( $filters );

$page = $request->get( 'page' );
var_dump( $page );

$limit = $request->get( 'limit' ); # key doesn't exist!
var_dump( $limit );
```

This will print:

```php
string(4) "4711"        # $galleryId

array(2) {              # $filters
  'price' =>
  string(5) "19900"
  'weight' =>
  string(3) "2.5"
}

string(0) ""            # $page

NULL                    # $limit
```
