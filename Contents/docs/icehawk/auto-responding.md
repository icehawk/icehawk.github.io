# IceHawk auto responding 

This section shows the auto responding behaviour of the IceHawk component.

<hr class="blockquote">

## OPTIONS requests

As described in the [request handlers section](@baseUrl@/docs/icehawk/request-handlers.html) the IceHawk component checks all implented interfaces of 
request handlers mapped to the requested URI to determine the allowed request methods for this URI. The IceHawk component will then automatically 
serve an appropriate response.

Let's assume you configured a read route for the URI `/newsletter/subscribe` to present a newsletter subscription form and the mapped request handler 
implements the `HandlesGetRequest` interface.

The following `OPTIONS` request
```HTTP
OPTIONS: /newsletter/subscribe
```

will be responded with:

```HTTP
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Content-Length: 0
Allow: HEAD,GET
```

If you additionally configured a write route for the same URI to process the actual newsletter subscription with a `POST` request handler 
implementing the `HandlesPostRequest` interface, the response would look like this:
 
```HTTP
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Content-Length: 0
Allow: HEAD,GET,POST
```

And if you additionally allow `PUT`ting the newsletter subscription by also implementing the `HandlesPutRequest` interface in the same request handler, 
the response would be:
  
```HTTP
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Content-Length: 0
Allow: HEAD,GET,POST,PUT
```

You get the point.

<hr class="blockspace">

## Method not implemented

For the unlikely case you try to perform a request with an HTTP verb other than `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE` or `OPTIONS`, 
the IceHawk component will automatically respond.

For example a request like this:
```HTTP
FREEZE: /says/the/police
```
  
would be responded with:
  
```HTTP
HTTP/1.1 501 Not Implemented
Content-Type: text/plain; charset=utf-8
Content-Length: 37
501 - Method Not Implemented (FREEZE)
```
