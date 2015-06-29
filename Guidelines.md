# Introduction #

For more compatible/interoperable end-points, I made most of the design decissions based on the list below.



# GET #
## Typical Usage: ##
  * retrieve a representation
  * retrieve a representation if modified (caching)

## Typical Status Codes: ##
  * 200 (OK) ‐ the representation is sent in the response
  * 204 (no content) ‐ the resource has an empty representation
  * 301 (Moved Permanently) ‐ the resource URI has been updated
  * 303 (See Other) ‐ e.g. load balancing
  * 304 (not modified) ‐ the resource has not been modified (caching)
  * 400 (bad request) ‐ indicates a bad request (e.g. wrong parameter)
  * 404 (not found) ‐ the resource does not exits
  * 406 (not acceptable) ‐ the server does not support the required  representation
  * 500 (internal server error) ‐ generic error response
  * 503 (Service Unavailable) ‐ The server is currently unable to handle the request

---

# DELETE #
## Typical Usage: ##
  * delete aresource

## Typical Status Codes: ##
  * 200 (OK) ‐ the resource has been deleted
  * 301 (Moved Permanently) – the resource URI has been updated
  * 303 (See Other) ‐ e.g. load balancing
  * 400 (bad request) ‐ indicates a bad request
  * 404 (not found) ‐ the resource does not exits
  * 409 (conflict) ‐ general conflict
  * 500 (internal server error) – generic error response
  * 503 (Service Unavailable) ‐ The server is currently unable to handle the request


# PUT #
## Typical Usage: ##
  * create a resource with client‐side managed instance id
  * update a resource by replacing
  * update a resource by replacing if not modified (optimistic locking)
  * 

## Typical Status Codes: ##
  * 200 (OK) ‐ if an existing resource has been updated
  * 201 (created) ‐ if a new resource is created
  * 301 (Moved Permanently) ‐ the resource URI has been updated
  * 303 (See Other) ‐ e.g. load balancing
  * 400 (bad request) ‐ indicates a bad request
  * 404 (not found) ‐ the resource does not exits
  * 406 (not acceptable) ‐ the server does not support the required represetation
  * 409 (conflict) ‐ general conflict
  * 412 (Precondition Failed) e.g. conflict by performing conditional update
  * 415 (unsupported media type) ‐ received representation is not supported
  * 500 (internal server error) – generic error response
  * 503 (Service Unavailable) ‐ Theserver is currently unable to handle the request


# POST #
## Typical Usage: ##
  * create a resource with server‐side managed(auto generated) instance id
  * create a sub‐resource
  * partial update of a resource
  * partial update a resource if not modified (optimistic locking)
  * 

## Typical Status Codes: ##
  * 200 (OK) ‐ if an existing resource has been updated
  * 201 (created) ‐ if a new resource is created
  * 202 (accepted) ‐ accepted for processing but not been completed (Async processing)
  * 301 (Moved Permanently) ‐ the resource URI has been updated
  * 303 (See Other) ‐ e.g. load balancing
  * 400 (bad request) ‐ indicates a bad request
  * 404 (not found) ‐ the resource does not exits
  * 406 (not acceptable) ‐ the server does not support the required representation
  * 409 (conflict) ‐ general conflict
  * 412 (Precondition Failed) e.g. conflict by performing conditional updat
  * 415 (unsupported media type) ‐ received representation is not supported
  * 500 (internal server error) ‐ generic error response
  * 503 (Service Unavailable) ‐ The server is currently unable to handle the request
  * 