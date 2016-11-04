# IceHawk request input data 

You may noticed that so far the separated handling of read and write side does not differ that much. 
But when it comes to request input data, you'll see the big difference.

This documentation explains the way IceHawk provides you the request data on each side - read and write.

## Preamble

Before we discuss how to access request input data, let's briefly talk about data types in request input data.

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

As you can see `null` is defined by the `$default` parameter. If you like to use another value/type for non-existing keys/values, just pass it as the second parameter.

**Please note:** It's a `??` ([null coalescing operator](http://php.net/manual/en/language.operators.comparison.php#language.operators.comparison.coalesce)) in the return statement, not a `?:` ([ternary operator](http://php.net/manual/en/language.operators.comparison.php#language.operators.comparison.ternary)). So the value of `$default` is only applied, if the key does not exist in the data array.

## The read request input data

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
galleryId=4711&filters[price]=19900&filters[weight]=2.5&page=
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

# key doesn't exist, use FALSE as default value
$limit = $request->get( 'limit', false ); 
var_dump( $limit );
```

This will print:

```php
string(4) "4711"        # $galleryId

array(2) {              # $filters
  'price' => string(5) "19900"
  'weight' => string(3) "2.5"
}

string(0) ""            # $page

NULL                    # $limit

FALSE               	# $limit
```

### 2. Get all values

Assuming we have the following query string:

```http
galleryId=4711&filters[price]=19900&filters[weight]=2.5&page=
```

If you want to get all request input values, just use the `getData()` method on the `ReadRequestInput` object.

```php
$allData = $requestInput->getData();
var_dump( $allData );
```

This prints:

```php
array(3) {
  'galleryId' => string(4) "4711"
  'filters' => array(2) {
    'price' => string(5) "19900"
    'weight' => string(3) "2.5"
  }
  'page' => string(0) ""
}
```

**That's it for the read side.**

<hr class="blockspace">

## The write request input data

As you can see in the [request handlers section](@baseUrl@/docs/icehawk/request-handlers.html) the IceHawk component provides a request data object to the request handler.
This object carries the [request information](@baseUrl@/docs/icehawk/request-information.html) and the request input data.

This example shows how you can access the request input data inside a POST(/PUT/PATCH/DELETE) request handler:

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
[WriteRequestInput class](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Requests/WriteRequestInput.php).

This class is a wrapper of the superglobal `$_POST` variable and offers the following ways to access its data:

### 1. Get a single value
 
Assuming we have the following post data, submitted from a registration form:

```http
userId=0815&profile[username]=someguy&profile[password]=****&subscribeNewsletter=
```

```php
<?php

$userId = $requestInput->get( 'userId' );
var_dump( $userId );

$profile = $requestInput->get( 'profile' );
var_dump( $profile );

$subscribeNewsletter = $request->get( 'subscribeNewsletter' );
var_dump( $subscribeNewsletter );

$captcha = $request->get( 'captcha' ); # key doesn't exist!
var_dump( $captcha );

# key doesn't exist, use FALSE as default value
$captcha = $request->get( 'captcha', false );
var_dump( $captcha );
```

This will print:

```php
string(4) "0815"        # $userId

array(2) {              # $profile
  'username' => string(7) "someguy"
  'password' => string(4) "****"
}

string(0) ""            # $subscribeNewsletter

NULL                    # $captcha

FALSE               	# $captcha
```

### 2. Get all values

Assuming we have the following post data, submitted from a registration form:

```http
userId=0815&profile[username]=someguy&profile[password]=****&subscribeNewsletter=
```

If you want to get all request input values, just use the `getData()` method on the `WriteRequestInput` object.

```php
$allData = $requestInput->getData();
var_dump( $allData );
```

This prints:

```php
array(3) {
  'userId' => string(4) "0815"
  'profile' => array(2) {
    'username' => string(7) "someguy"
    'password' => string(4) "****"
  }
  'subscribeNewsletter' => string(0) ""
}
```

### 3. Get the raw request body

If the client submits a data in the request body, you can access it via the `getBody()` method.

Assuming the following JSON string is the body:

```json
{"message":"You look great today.","flowers":["red","yellow","orange"]}
```

```php
$body = $requestInput->getBody();
var_dump( $body );
```

Prints:

```php
string(71) "{"message":"You look great today.","flowers":["red","yellow","orange"]}"
```

**Please note:**

* The `getBody()` method always returns a `string`, which is empty if no content was submitted in the request body.
* The submitted content remains unchanged and is provided as submitted. If you need to parse it somehow, you need to take care of that yourself.  
Unlike other frameworks we intentionally decided to not parse the body automatically depending on a header that (maybe) was sent with the request.  

<hr class="blockspace">

### 3. Get uploaded files

The IceHawk component offers a comfortable way to access uploaded files, especially when you need to handle multiple files in one request. 
IceHawk transforms the confusing structured `$_FILES` array that PHP unpacks to a normalized array containing an object for each uploaded file.
  
An uploaded file is represented by the [UploadedFile class](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Requests/UploadedFile.php) with
its interface `IceHawk\IceHawk\Interfaces\ProvidesUploadedFileData`.

You can read more about this in the [uploaded files section](@baseUrl@/docs/icehawk/uploaded-files.html).

In the following example, we'll explain how to access the files that were uploaded by this form:

```html
<form action="/upload/to/server" method="post" enctype="multipart/form-data">
	<input type="file" name="file1">
	<input type="file" name="files[]">
	<input type="file" name="files[]">
	<input type="file" name="files[file4]">
	<input type="submit" value="upload">
</form>
```

<hr class="blockspace">

#### 3.1 Get all uploaded files

```php
$allUploadedFiles = $requestInput->getAllFiles();
var_dump( $allUploadedFiles );
```

This prints:

```php
array (size=2)
  'file1' => 
    array (size=1)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[15]
          private 'name' => string '...' (length=24)
          private 'tmpName' => string '...' (length=66)
          private 'error' => int 0
          private 'size' => int 40506
          private 'type' => string 'image/png' (length=9)
  'files' => 
    array (size=3)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[17]
          private 'name' => string '...' (length=24)
		  private 'tmpName' => string '...' (length=66)
		  private 'error' => int 0
		  private 'size' => int 40506
		  private 'type' => string 'image/png' (length=9)
      1 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[18]
          private 'name' => string '...' (length=24)
		  private 'tmpName' => string '...' (length=66)
		  private 'error' => int 0
		  private 'size' => int 40506
		  private 'type' => string 'image/png' (length=9)
      'file4' => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[19]
          private 'name' => string '...' (length=24)
		  private 'tmpName' => string '...' (length=66)
		  private 'error' => int 0
		  private 'size' => int 40506
		  private 'type' => string 'image/png' (length=9)
```

<hr class="blockspace">

#### 3.2 Get specific uploaded files

```php
$specificFiles = $requestInput->getFiles( 'files' );
var_dump( $specificFiles );
```

This prints:

```php
array (size=3)
  0 => 
	object(IceHawk\IceHawk\Requests\UploadedFile)[17]
	  private 'name' => string '...' (length=24)
	  private 'tmpName' => string '...' (length=66)
	  private 'error' => int 0
	  private 'size' => int 40506
	  private 'type' => string 'image/png' (length=9)
  1 => 
	object(IceHawk\IceHawk\Requests\UploadedFile)[18]
	  private 'name' => string '...' (length=24)
	  private 'tmpName' => string '...' (length=66)
	  private 'error' => int 0
	  private 'size' => int 40506
	  private 'type' => string 'image/png' (length=9)
  'file4' => 
	object(IceHawk\IceHawk\Requests\UploadedFile)[19]
	  private 'name' => string '...' (length=24)
	  private 'tmpName' => string '...' (length=66)
	  private 'error' => int 0
	  private 'size' => int 40506
	  private 'type' => string 'image/png' (length=9)
```

<hr class="blockspace">

#### 3.3 Get one uploaded file

```php
# access a file using default index 0
$uploadedFile0 = $requestInput->getOneFile( 'files' );
var_dump( $uploadedFile0 );

# access a file at a string index
$uploadedFile4 = $requestInput->getOneFile( 'files', 'file4' );
var_dump( $uploadedFile4 );

# access a not existing file index
$notUploadedFile = $requestInput->getOneFile( 'files', 'not-existing' );
var_dump( $notUploadedFile );
```

This prints:

```php
object(IceHawk\IceHawk\Requests\UploadedFile)[17]
  private 'name' => string '...' (length=24)
  private 'tmpName' => string '...' (length=66)
  private 'error' => int 0
  private 'size' => int 40506
  private 'type' => string 'image/png' (length=9)

object(IceHawk\IceHawk\Requests\UploadedFile)[19]
  private 'name' => string '...' (length=24)
  private 'tmpName' => string '...' (length=66)
  private 'error' => int 0
  private 'size' => int 40506
  private 'type' => string 'image/png' (length=9)

NULL
```
