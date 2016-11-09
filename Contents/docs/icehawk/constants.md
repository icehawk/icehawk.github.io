# IceHawk constants

The IceHawk component ships with two handy abstract classes which provide public constants you can use:

<hr class="blockspace">
 
## HTTP methods

If you want to check what HTTP method was used for the current request, you can ask the [request info object](@baseUrl@/docs/icehawk/request-information.html) 
and can use the following class constants to compare it against:

```php
use IceHawk\IceHawk\Constants\HttpMethod;

HttpMethod::HEAD; 
HttpMethod::GET; 
HttpMethod::POST; 
HttpMethod::PUT; 
HttpMethod::PATCH; 
HttpMethod::DELETE; 
HttpMethod::OPTIONS; 
```

Additionally you can get all methods in one constant array:

```php
use IceHawk\IceHawk\Constants\HttpMethod;

echo "Request method is: ";

if ( in_array( $requestInfo->getMethod(), HttpMethod::ALL_METHODS ) )
{
	echo "valid"
}
else
{
	echo "invalid"
}
```

And we also provide a constant array for the methods of each side - read and write:

```php
use IceHawk\IceHawk\Constants\HttpMethod;

echo "Request method is: ";

if ( in_array( $requestInfo->getMethod(), HttpMethod::READ_METHODS ) )
{
	echo "for the read side"
}
elseif ( in_array( $requestInfo->getMethod(), HttpMethod::WRITE_METHODS ) )
{
	echo "for the write side"
}
else
{
	echo "OPTIONS or invalid";
}
```

<hr class="blockspace">

## HTTP codes

To help you respond requests with appropriate HTTP status codes we provide an abstract class with all available HTTP status codes, defined in 
[Hypertext Transfer Protocol - HTTP/1.1](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
 
```php
use IceHawk\IceHawk\Constants\HttpCode;

HttpCode::CONTINUE; # 100
HttpCode::SWITCHING_PROTOCOLS; # 101
HttpCode::PROCESSING; # 102
HttpCode::OK; # 200
HttpCode::CREATED; # 201
HttpCode::ACCEPTED; # 202
HttpCode::NON_AUTHORITATIVE_INFORMATION; # 203
HttpCode::NO_CONTENT; # 204
HttpCode::RESET_CONTENT; # 205
HttpCode::PARTIAL_CONTENT; # 206
HttpCode::MULTI_STATUS; # 207
HttpCode::ALREADY_REPORTED; # 208
HttpCode::IM_USED; # 226
HttpCode::MULTIPLE_CHOICES; # 300
HttpCode::MOVED_PERMANENTLY; # 301
HttpCode::FOUND; # 302
HttpCode::SEE_OTHER; # 303
HttpCode::NOT_MODIFIED; # 304
HttpCode::USE_PROXY; # 305
HttpCode::SWITCH_PROXY; # 306
HttpCode::TEMPORARY_REDIRECT; # 307
HttpCode::PERMANENT_REDIRECT; # 308
HttpCode::BAD_REQUEST; # 400
HttpCode::UNAUTHORIZED; # 401
HttpCode::PAYMENT_REQUIRED; # 402
HttpCode::FORBIDDEN; # 403
HttpCode::NOT_FOUND; # 404
HttpCode::METHOD_NOT_ALLOWED; # 405
HttpCode::NOT_ACCEPTABLE; # 406
HttpCode::PROXY_AUTHENTICATION_REQUIRED; # 407
HttpCode::REQUEST_TIMEOUT; # 408
HttpCode::CONFLICT; # 409
HttpCode::GONE; # 410
HttpCode::LENGTH_REQUIRED; # 411
HttpCode::PRECONDITION_FAILED; # 412
HttpCode::REQUEST_ENTITY_TOO_LARGE; # 413
HttpCode::REQUEST_URI_TOO_LONG; # 414
HttpCode::UNSUPPORTED_MEDIA_TYPE; # 415
HttpCode::REQUESTED_RANGE_NOT_SATISFIABLE; # 416
HttpCode::EXPECTATION_FAILED; # 417
HttpCode::I_AM_A_TEAPOT; # 418
HttpCode::AUTHENTICATION_TIMEOUT; # 419
HttpCode::METHOD_FAILURE; # 420
HttpCode::UNPROCESSABLE_ENTITY; # 422
HttpCode::LOCKED; # 423
HttpCode::FAILED_DEPENDENCY; # 424
HttpCode::UNORDERED_COLLECTION; # 425
HttpCode::UPGRADE_REQUIRED; # 426
HttpCode::PRECONDITION_REQUIRED; # 428
HttpCode::TOO_MANY_REQUESTS; # 429
HttpCode::REQUEST_HEADER_FIELDS_TOO_LARGE; # 431
HttpCode::NO_RESPONSE; # 444
HttpCode::RETRY_WITH; # 449
HttpCode::BLOCKED_BY_WINDOWS_PARENTAL_CONTROLS; # 450
HttpCode::UNAVAILABLE_FOR_LEGAL_REASONS; # 451
HttpCode::REQUEST_HEADER_TOO_LARGE; # 494
HttpCode::CERT_ERROR; # 495
HttpCode::NO_CERT; # 496
HttpCode::HTTP_TO_HTTPS; # 497
HttpCode::CLIENT_CLOSED_REQUEST; # 499
HttpCode::INTERNAL_SERVER_ERROR; # 500
HttpCode::NOT_IMPLEMENTED; # 501
HttpCode::BAD_GATEWAY; # 502
HttpCode::SERVICE_UNAVAILABLE; # 503
HttpCode::GATEWAY_TIMEOUT; # 504
HttpCode::HTTP_VERSION_NOT_SUPPORTED; # 505
HttpCode::VARIANT_ALSO_NEGOTIATES; # 506
HttpCode::INSUFFICIENT_STORAGE; # 507
HttpCode::LOOP_DETECTED; # 508
HttpCode::BANDWIDTH_LIMIT_EXCEEDED; # 509
HttpCode::NOT_EXTENDED; # 510
HttpCode::NETWORK_AUTHENTICATION_REQUIRED; # 511
HttpCode::NETWORK_READ_TIMEOUT_ERROR; # 598
HttpCode::NETWORK_CONNECT_TIMEOUT_ERROR; # 599
```
