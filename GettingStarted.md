## Installing GoRest ##

GoRest is just a package that you can use with standard Go packages.

It has been tested on release [go1](http://golang.org/doc/devel/release.html#go1) of Go, so make sure you have this or above.




**Install:**
```
go get code.google.com/p/gorest
```


## Hello World/Who? ##

Below is a sample "Hello World!" or "Hello Who?" service:

```
package main
import (
    "code.google.com/p/gorest"
	"net/http"
)
func main() {
    gorest.RegisterService(new(HelloService)) //Register our service
    http.Handle("/",gorest.Handle())	
    http.ListenAndServe(":8787",nil)
}

//Service Definition
type HelloService struct {
    gorest.RestService `root:"/tutorial/"`
    helloWorld  gorest.EndPoint `method:"GET" path:"/hello-world/" output:"string"`
    sayHello    gorest.EndPoint `method:"GET" path:"/hello/{name:string}" output:"string"`
}
func(serv HelloService) HelloWorld() string{
    return "Hello World"
}
func(serv HelloService) SayHello(name string) string{
    return "Hello " + name
}
```



When the above is run and called from a browser with :
```
http://localhost:8787/tutorial/hello/James
```
the service responds with an http entity: **“Hello James”**.


P.S: _Play around with this service by adding more end-points, GoRest will guide with descriptive errors until you get it right._




## Service Definition ##

A service (gorest.RestService) is a collection of end-points (gorest.EndPoint).

A service is simply a struct which has an anonymous field of type gorest.RestService.
This field may have tags defined in the tags list below.

When we define a service we give it a root path, that can also be ommited (“/tutorial”, in this case), then every end-point that belongs to this service will have a path that falls under the services path. In our case above, the “SayHello” end-point will have the path “/tutorial/hello/{string}”.


### Service Level Tags ###


|root|The root path of the service in the URL namespace.|
|:---|:-------------------------------------------------|
|consumes|A mime type like “application/json”. This is used to marshall the entities posted to the service by the client.|
|produces|A mime type like “application/json”. This is used to marshall the entities return back to the client.|
|realm|Authorization realm that users of this service must be registered in, to be able to call any end-points in this service.|


## Endpoints ##
An end-point is an unambiguous, destination of an http request.  More like the definitive implementation of a remote procedure in RPC.

An end-point definition is two part. First the signature part (field in the struct), then the implementation (method of the same struct).

For our “SayHello” end-point, the signature is:
```
sayHello    gorest.EndPoint `method:"GET" path:"/hello/{name:string}" output:"string"`
```

Key factors:
  1. The end-points struct field name must be not be exported (starts with lowercase)
  1. The corresponding method name must be the same but exported (starts with uppercase)
  1. The struct field type is gorest.EndPoint
  1. The “method” and “path” tags are compulsory.
  1. In the path tag one can define parts of the path that will be submitted as arguments to the implementation method. These are identified with curly braces {} in the path and consists of the variable name and the type {name:string}. The type of the variable can be any one of [string,int,bool,float32,float64]. These type are allowable on HTTP url’s.
  1. The type in the “output” tag must be the same as the return type of the implementation method. The “output” is what goes into the HTTP entity on the response our get method. More on this on the section for method types.


### EndPoint Level Tags: ###
| **Name** | **Tag Description** |
|:---------|:--------------------|
|method    |The endpoint HTTP method of request. Supported: HEAD, GET, POST, PUT, OPTIONS, DELETE|
|path      |The path used in the HTTP request to get to this end-point. The path also contains the parameter definitions that are passed to the implementation method for the end-point.|
|output    |For an end-point of “method” type GET, this is the “stuff” that is written back to the HTTP responses entity.|
|postdata  |For an end-point of “method” types POST and PUT, this is what is read from the HTTP requests entity, and passed into the first parameter or the implementation method.|
|role      |Authorization role that users of this end-point must be have, to be able to call this end-point. This is checked after the user has been cleared to be in this service realm as required by tag “realm” in the services tags.|


## URL Pattern Matching ##

GoRest currently allows 3 types of url pattern matching.
  * Prefixed:  path:"/person/{name:string}/{age:int}/{weight:float32}"
  * Mixed: path:"/person/{name:string}/country/state/{weight:float32}/parent/"
  * Variable: path:"/countries/ {...:string}"

The system is designed to ensure that paths will never clash at runtime. When you register your endpoints, GoRest will panic if there is a possibility for clashes, end will not load.

**Prefixed** works on prefix and signature length basis. For a prefix “person”, only one end-point with a signature length of 2 can be registered. You can  register more end-points with signature lengths of 3, 4, 5, etc, as long as there is one of each for the predefined prefix.

**Mixed** works the same as Prefixed. In addition, it ensures that the parts of the URL that are not path-parameters do correspond to what’s on the signature (name and position, e.g: country in position 3).

**Variable-Length** allows you to take the whole namespace under a path and have the parts of the url sent to your implementation method as a slice of the specified type.


# HTTP Method structure: #

Each end-point uses only one of the HTTP methods below. Depending on the type of the method, the rules below apply for what type of parameters and return types the implementing method may have.

| **Method** | **Input** | **Output** |
|:-----------|:----------|:-----------|
|HEAD        |Path+query params only|None        |
|GET         |Path+query only|Any serializable. E.g: struct, slice,string|
|PUT         |Posted entity + Path+query params|None        |
|POST        |Posted entity + Path+query params|None        |
|OPTIONS     |Path+query params only|None        |
|DELETE      |Path+query params only|None        |

### Example valid endpoint signature pairs: ###

GET:
```
 sayHello gorest.EndPoint `method:"GET" path:"/hello/{name:string}" output:"string"`
 func(serv HelloService) SayHello(name string) string
```

```
getMapStruct   gorest.EndPoint `method:"GET" path:"/mapstruct/{Bool:bool}/{Int:int}" output:"map[string]User"`
func (serv Service) GetMapStruct(Bool bool, Int int) map[string]User
```

HEAD:
```
sayHello gorest.EndPoint `method:"HEAD" path:"/hello/{name:string}"`
func(serv HelloService) SayHello(name string)  
```
```
head gorest.EndPoint `method:"HEAD" path:"/bool/{Bool:bool}/{Int:int}"`
func (serv Service) Head(Bool bool, Int int)
```
PUT:

```
put gorest.EndPoint `method:"PUT" path:"/put/"  postdata:"User"`
func(serv HelloService) Put(posted User) 
```

POST:

```
posted gorest.EndPoint method:"POST" path:"/post/"  postdata:"User" 
func(serv HelloService) Posted(posted User)
```

```
postArrayStruct gorest.EndPoint `method:"POST" path:"/arraystruct/{Bool:bool}/{Int:int}" postdata:"[]User"`
func (serv Service) PostArrayStruct(posted []User, Bool bool, Int int) 
```

OPTIONS:

```
options			gorest.EndPoint `method:"OPTIONS" path:"/bool/{Bool:bool}/{Int:int}"`
func (serv Service) Options(Bool bool, Int int)
```

DELETE:

```
doDelete			gorest.EndPoint `method:"DELETE" path:"/bool/{Bool:bool}/{Int:int}"`
func (serv Service) DoDelete(Bool bool, Int int)
```

## Marshal/UnMarshal ##

GoRest has a pluggable system to allow the transomation from go data types to HTTP entities(text-based encodings). It comes with JSON and XML “Marshallers”, but marshallers of different types can be registered on the system.

These registered marshaller are shared for server side and client side usage of GoRest, all depending on which part you are using.

### Register a Marshaller: ###

We use the function: gorest.RegisterMarshaller(mime  string, m **Marshaller)
```
gorest.RegisterMarshaller(“application/json”, gorest.NewJSONMarshaller())
```**

The type Marshaller  is just a struct that has two function fields for the marshal and unmarhal functions.  gorest.NewJSONMarshaller() creates a new instance of this, from the default JSON one that’s predefined.


## Authorization: ##