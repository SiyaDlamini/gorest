# GoRest #
A RESTful style web-services framework for the [Go](http://golang.org) language. Creating services in Go is straight forward, GoRest takes this a step further by adding a layer that makes tedious tasks much more automated and avoids regular pitfalls. This gives you the opportunity to focus more on the task at hand... minor low-level http handling.

Features:

  * **Type safe** conversion of URL parameters into method arguments.
  * Automated **marshal/unmarshall** of HTTP entities.
  * Highly configurable end-point system that makes grouping of end-points into services easy.
  * Tag(Annotations) based configuration of end-points and services.
  * Easilly pluggable URL system to allow various multiplexing configurations.
  * Configured as per "mostly accepted" RESTful style behaviour (e.g: behaviour of a POST or GET). Ability to overide the default behaviour.
  * Many planned features coming soon.







# Example Usage: #

**Install:** _go get code.google.com/p/gorest_



A full workin example is provided with the sources.

```
func main(){
    gorest.RegisterService(new(OrderService))
    http.Handle("/",gorest.Handle())	
    http.ListenAndServe(":8787",nil)

}

//************************Define Service***************************

type OrderService struct{
    //Service level config
    gorest.RestService    `root:"/orders-service/" consumes:"application/json" produces:"application/json"`

    //End-Point level configs: Field names must be the same as the corresponding method names,
    // but not-exported (starts with lowercase)

    userDetails gorest.EndPoint `method:"GET" path:"/users/{Id:int}" output:"User"`
    listItems   gorest.EndPoint `method:"GET" path:"/items/" output:"[]Item"`
    addItem     gorest.EndPoint `method:"POST" path:"/items/" postdata:"Item"`

    //On a real app for placeOrder below, the POST URL would probably be just /orders/, this is just to
    // demo the ability of mixing post-data parameters with URL mapped parameters.
    placeOrder  gorest.EndPoint `method:"POST" path:"/orders/new/{UserId:int}/{RequestDiscount:bool}" postdata:"Order"`
    viewOrder     gorest.EndPoint `method:"GET" path:"/orders/{OrderId:int}" output:"Order"`
    deleteOrder     gorest.EndPoint `method:"DELETE" path:"/orders/{OrderId:int}"`


}

//Handler Methods: Method names must be the same as in config, but exported (starts with uppercase)



func(serv OrderService) UserDetails(Id int) (u User){
    if user,found:=userStore[Id];found{
        u =user
        return
    }
    serv.ResponseBuilder().SetResponseCode(404).Overide(true)  //Overide causes the entity returned by the method to be ignored. Other wise it would send back zeroed object
    return
}

func(serv OrderService) ListItems()[]Item{
    serv.ResponseBuilder().CacheMaxAge(60*60*24) //List cacheable for a day. More work to come on this, Etag, etc
    return itemStore
}

func(serv OrderService) AddItem(i Item){

    for _,item:=range itemStore{
        if item.Id == i.Id{
            item=i
            serv.ResponseBuilder().SetResponseCode(200) //Updated http 200, or you could just return without setting this. 200 is the default for POST
            return
        }
    }

    //Item Id not in database, so create new
    i.Id = len(itemStore)
    itemStore=append(itemStore,i)

    serv.ResponseBuilder().Created("http://localhost:8787/orders-service/items/"+string(i.Id)) //Created, http 201
}

//On the method parameters, the posted data(http-entity) is always first, followed by the URL mapped parameters
func(serv OrderService) PlaceOrder(order Order,UserId int,AskForDiscount bool){
    order.Id = len(orderStore)

    if user,found:= userStore[UserId];found{
          if item,exists:=findItem(order.ItemId);exists{
                itemStore[item.Id].AvailableStock--

                if AskForDiscount && order.Amount >5{
                    order.Discount = 2.5
                }
                order.Id=len(orderStore)
                order.UserId=UserId
                order.Cancelled=false
                orderStore=append(orderStore,order)
                user.OrderIds=append(user.OrderIds,order.Id)

                userStore[user.Id]=user

                serv.ResponseBuilder().SetResponseCode(201).Location("http://localhost:8787/orders-service/orders/"+string(order.Id))//Created
                return

          } else{
              serv.ResponseBuilder().SetResponseCode(404).WriteAndOveride([]byte("Item not found"))//You can still manually place an entity on the response, even on a POST
              return
          }
    }

    serv.ResponseBuilder().SetResponseCode(404).WriteAndOveride([]byte("User not found"))
    return
}
func(serv OrderService) ViewOrder(id int) (retOrder Order){
     for _,order:=range orderStore{
        if order.Id == id{
            retOrder = order
            return
        }
     }
     serv.ResponseBuilder().SetResponseCode(404).Overide(true)
     return
}
func(serv OrderService) DeleteOrder(id int) {
     for pos,order:=range orderStore{
        if order.Id == id{
            order.Cancelled =true
            orderStore[pos]=order
            return               //Default http code for DELETE is 200
        }
     }
     serv.ResponseBuilder().SetResponseCode(404).Overide(true)
     return
}
```