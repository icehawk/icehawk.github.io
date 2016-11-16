# IceHawk Uploaded files 

This section covers the comfortable handling of uploaded files with the IceHawk component.

<hr class="blockspace">

## Transformation of $_FILES

If you have a form on your website containing one file input field and you submit it to the server, 
PHP unpacks the uploaded files to the `$_FILES` array in the following way:
 
**HTML form**
```html
<form action="/upload/to/server" method="post" enctype="multipart/form-data">
	<input type="file" name="uploadedFile">
	<input type="submit" value="Upload">
</form> 
```

**$_FILES array**
```php
array (size=1)
  'uploadedFile' => 
    array (size=5)
      'name' => string 'branding-circle.png' (length=19)
      'type' => string 'image/png' (length=9)
      'tmp_name' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpaX6BSv' (length=66)
      'error' => int 0
      'size' => int 14273
```

But if you upload multiple files in one submit, the structure of `$_FILES` changes from a 2-dimensional array to a 3-dimensional array or even a mixed one.
See this example:

**HTML form**
```html
<form action="/upload/to/server" method="post" enctype="multipart/form-data">
	<input type="file" name="uploadedFile">
	<input type="file" name="files[]">
	<input type="file" name="files[]">
	<input type="file" name="files[file4]">
	<input type="submit" value="Upload">
</form> 
```

**$_FILES array**
```php
array (size=2)
  'uploadedFile' => 
    array (size=5)
      'name' => string 'branding-circle.png' (length=19)
      'type' => string 'image/png' (length=9)
      'tmp_name' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpLVo9Fb' (length=66)
      'error' => int 0
      'size' => int 14273
  'files' => 
    array (size=5)
      'name' => 
        array (size=3)
          0 => string 'branding-circle.png' (length=19)
          1 => string 'branding-circle.png' (length=19)
          'file4' => string 'branding-circle.png' (length=19)
      'type' => 
        array (size=3)
          0 => string 'image/png' (length=9)
          1 => string 'image/png' (length=9)
          'file4' => string 'image/png' (length=9)
      'tmp_name' => 
        array (size=3)
          0 => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpBcPxjk' (length=66)
          1 => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpz3JrCY' (length=66)
          'file4' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpeM1qZn' (length=66)
      'error' => 
        array (size=3)
          0 => int 0
          1 => int 0
          'file4' => int 0
      'size' => 
        array (size=3)
          0 => int 14273
          1 => int 14273
          'file4' => int 14273
```

This is pretty confusing in the first place and leads to conditional code that checks e.g. on the "name" index of a file if its value is a string or an array.
  
That's why the IceHawk component normalizes the unpacked array to always be 3-dimensional in favour of providing _one_ simple API to access uploaded files.
Especially in the second example you can see that data bound to one uploaded file is distributed to multiple arrays.
 
In addition to normalize the unpacked `$_FILES` array, the IceHawk component also wraps an object of type `UploadedFile` around each file so that all file related data can be comfortably accessed.
 
Since file uploads only work with POST (write) requests, the list of uploaded files is attached to the request input data object, like shown in [this section](@baseUrl@/docs/icehawk/request-input-data.html).

When asking the write request input object for all uploaded files 

```php
var_dump( $requestInput->getAllFiles() );
```

it will give you this for the single file upload form:

```php
array (size=1)
  'uploadedFile' => 
    array (size=1)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[15]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpG4XHn5' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
```

And this for the multi-file upload form:

```php
array (size=2)
  'uploadedFile' => 
    array (size=1)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[15]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpL4xQLe' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
  'files' => 
    array (size=3)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[17]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpildUxI' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
      1 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[18]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpwh3DyL' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
      'file4' => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[19]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpGCUjgR' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
```

As you can see the array is always 3-dimensional and file-related data is kept together in an object.

<hr class="blockspace">

## How to access uploaded files

The array of uploaded files is normalized to a 3-dimensional array, so it's quiet obvious to provide 3 appropriate methods
to access each of these dimensions.

Given the multi-file upload form shown above was submitted.

### Get all uploaded files (1st dimension)
    
```php
$allUploadedFiles = $requestInput->getAllFiles();
var_dump( $allUploadedFiles );
```

Prints:

```php
array (size=2)
  'uploadedFile' => 
    array (size=1)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[15]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpL4xQLe' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
  'files' => 
    array (size=3)
      0 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[17]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpildUxI' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
      1 => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[18]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpwh3DyL' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
      'file4' => 
        object(IceHawk\IceHawk\Requests\UploadedFile)[19]
          private 'name' => string 'branding-circle.png' (length=19)
          private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpGCUjgR' (length=66)
          private 'error' => int 0
          private 'size' => int 14273
          private 'type' => string 'image/png' (length=9)
```

<hr class="blockspace">

### Get specific uploaded files (2nd dimension)

```php
$specificFiles = $requestInput->getFiles( 'files' );
var_dump( $specificFiles );
```

Prints:

```php
array (size=3)
  0 => 
	object(IceHawk\IceHawk\Requests\UploadedFile)[17]
	  private 'name' => string 'branding-circle.png' (length=19)
	  private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpildUxI' (length=66)
	  private 'error' => int 0
	  private 'size' => int 14273
	  private 'type' => string 'image/png' (length=9)
  1 => 
	object(IceHawk\IceHawk\Requests\UploadedFile)[18]
	  private 'name' => string 'branding-circle.png' (length=19)
	  private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpwh3DyL' (length=66)
	  private 'error' => int 0
	  private 'size' => int 14273
	  private 'type' => string 'image/png' (length=9)
  'file4' => 
	object(IceHawk\IceHawk\Requests\UploadedFile)[19]
	  private 'name' => string 'branding-circle.png' (length=19)
	  private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpGCUjgR' (length=66)
	  private 'error' => int 0
	  private 'size' => int 14273
	  private 'type' => string 'image/png' (length=9)
```

<hr class="blockspace">

### Get one uploaded file (3rd dimension)

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
# file at default index 0
object(IceHawk\IceHawk\Requests\UploadedFile)[17]
  private 'name' => string 'branding-circle.png' (length=19)
  private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpildUxI' (length=66)
  private 'error' => int 0
  private 'size' => int 14273
  private 'type' => string 'image/png' (length=9)

# file at a string index "file4"
object(IceHawk\IceHawk\Requests\UploadedFile)[19]
  private 'name' => string 'branding-circle.png' (length=19)
  private 'tmpName' => string '/private/var/folders/dq/883sl5qj2td_wp17hv8_mx3m0000gn/T/phpGCUjgR' (length=66)
  private 'error' => int 0
  private 'size' => int 14273
  private 'type' => string 'image/png' (length=9)

# not existing file
NULL
```

**Please note:** 
[We will change the return value for not-existing files in `v2.1.0` to an empty `UploadedFile` object with the error code `UPLOAD_ERR_NO_FILE`.](https://github.com/icehawk/icehawk/issues/22)
And will then add a return type declaration to the `getOneFile()` method.

<hr class="blockspace">

## The UploadedFile object

We already mentioned that the meta data of an uploaded file is wrapped in an `UploadedFile` object.
This object implements the [ProvidesUploadedFileData](https://github.com/icehawk/icehawk/blob/@icehawk/icehawk-version@/src/Interfaces/ProvidesUploadedFileData.php) 
interface and offers the following methods for access to the meta data.

```php
public function getError() : int
```

This method returns one of [PHP's predefined upload error codes](http://php.net/manual/en/features.file-upload.errors.php). 

```php
public function getName() : string
```

This method returns the original filename of the uploaded file.

```php
public function getSize() : int
```

This method returns in bytes the filesize of the uploaded file.
  
```php
public function getTmpName() : string
```

This method returns the temporary filepath on the server of the uploaded file.

```php
public function getType() : string
```

This method retuns the mime type of the uploaded file, _if the browser provided this information._
An example would be `image/gif`.

**Note from the PHP docs:**
> This mime type is however not checked on the PHP side and therefore don't take its value for granted.

```php
public function getRealType() : string
```

The method uses the PHP extension [fileinfo](http://php.net/fileinfo) to determine a more precise mime type of the uploaded file on the server side.

```php
public function getEncoding() : string
```

This method uses the PHP extension [fileinfo](http://php.net/fileinfo) to determine the mime encoding of the uploaded file and returns it. 
For example: `utf-8`.

```php
public function didUploadSucceed() : bool
```

This method return whether the upload of the file succeeded or not. It checks for the error code to be `UPLOAD_ERR_OK`.

```php
public function getErrorMessage() : string
```

This method returns an appropriate error message for the upload error code:
 
| Error code             | Value | Message                                           |
|------------------------|-------|---------------------------------------------------|
| UPLOAD_ERR_OK          | 0     | _empty string_                                    |
| UPLOAD_ERR_INI_SIZE    | 1     | Filesize exceeded max size allowed by server.     |
| UPLOAD_ERR_FORM_SIZE   | 2     | Filesize exceeded max size allowed by input form. |
| UPLOAD_ERR_PARTIAL     | 3     | File was uploaded partially.                      |
| UPLOAD_ERR_NO_FILE     | 4     | No file uploaded.                                 |
| UPLOAD_ERR_NO_TMP_DIR  | 6     | No upload temp directory available.               |
| UPLOAD_ERR_CANT_WRITE  | 7     | Upload canceled by PHP extension.                 |
| UPLOAD_ERR_EXTENSION   | 8     | Cannot write file.                                |
