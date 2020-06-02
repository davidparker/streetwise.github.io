---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---
## Using the provided library Streetwise.Api.Connect ( recommended )

The library is a .net standard 2.0 library.  and should work in almost any platform
Install as a reference to NOP or into a nop plugin

An example below shows how to login and send data to the API using the library

<script src="https://gist.github.com/davidparker/1ff154061d715cfd7636f4ac8b935834.js" height="200"></script>

### Endpoints

```cs
// Your clientid, your token, the store url ending with /slash
_authService.GetLogin(string clientId, string clientSecret, string storeUrl)
// example
_authService.GetLogin("hudnoewihrnl.lcidjoukjfhrwrug", "asjhajshajshjaskjhuwjb7213tevdyuy7ewru", "https:127.0.0.1:8858/")
// Refresh token  ( TTL is only 10 mins )  if you require access for more than 10 mins, you must refresh the token
_authService.RefreshLogin(string existingToken, string storeUrl);
// existingToken is your existing token from the AccessToken property in GetLogin Response ( see models )
// getting data from the API
        /// <summary>
        /// Get items from the api
        /// </summary>
        /// <param name="requestModel">request model ( see models )</param>
        /// <param name="storeUrl">Store URL</param>
        /// <param name="endpoint">ApiEndpoint</param>
        /// <returns>ApiResponse object</returns>
        public async Task<ApiResponse> GetData(RequestModel requestModel, string storeUrl, string endpoint)
// Posting / sending data to the API
        /// <summary>
        /// Send an item for insert or update
        /// </summary>
        /// <param name="item">object must have a base of BAseModel</param>
        /// <param name="storeUrl">URL for the store, usually in app.config</param>
        /// <param name="endpoint">ApiEndpoint</param>
        /// <returns>ApiResponse object</returns>
        public async Task<ApiResponse> SendData(RequestModel requestModel, string storeUrl, string endpoint)
```

## Data

The client provides some models that provide information directly, if needed.


```cs
  // Gets the available endponts
  Streetwise.Api.Connect.Models.ApiAuthEndpoints
  // example
  string loginUrl = Streetwise.Api.Connect.Models.ApiAuthEndpoints.Login  // output => "api/ApiAuthenticate/Login"
  // Details of error messages
  Streetwise.Api.Connect.Models.ApiErrorMessages
  // example
  var error = Streetwise.Api.Connect.Models.ApiErrorMessages.ExpiredLogin // output => "Token expired, please login again"
  // these are the standard messages we send.
  // we provide a list of the message incase you want to check if error.contains(ApiErrorMessages)
  Streetwise.Api.Connect.Models.ApiErrorMessages.GetAll(); // returns list<string>() 

  //For the GetData and SendData endpoints. The urls are from ApiEndpoints
  // rather than the ApiAuthEndpoints, here are the details

  namespace Streetwise.Api.Connect.Models
  {
    public class ApiEndpoints
    {
        public const string OnlineOrderCreate = "api/OnlineOrderDetail/Create";
        public const string CreateOrder = "api/Orders/Create";
        public const string UpdateOrder = "api/Orders/Update";
        public const string UpdateOrderStatus = "api/Orders/UpdateStatus";

        public const string UpdateOrderItem = "api/OrderItem/Update";
        public const string CreateOrderItem = "api/OrderItem/Create";

        public const string AddOrderAddress = "api/OrderAddress/Create";
        public const string UpdateOrderAddress = "api/OrderAddress/Update";
    }
  }
  
```

## Models

#### Login Response
We return this model when you call the Login endpoint
this is the type returned on good or bad login

This is also the response from RefreshLogin
```cs
    public class LoginResponse
    {
        /// <summary>
        /// Your access token for futher authentification
        /// </summary>
        public string AccessToken { get; set; }

        /// <summary>
        /// The datetime your token expires ( usually now + 10 mins ) 
        /// local server time
        /// </summary>
        public DateTime Expires { get; set; }

        /// <summary>
        /// Error message, from ApiErrorMessages object
        /// </summary>
        public string ErrorMessage { get; set; }

        /// <summary>
        /// Check if true login success, false means check the error message
        /// </summary>
        public bool IsSuccess { get; set; }
    }
```

#### Request Model

For request, the library uses a generic handler.

```cs
    public class BaseModel
    {
        /// <summary>
        /// A valid acccess token, used to authenticate your request
        /// </summary>
        public string AccessToken { get; set; }
    }

    public class RequestModel : BaseModel
    {
        /// <summary>
        /// Object of a single item.  be that of class, string int  etc
        /// </summary>
        public object Item { get; set; }

        /// <summary>
        /// List of items to send. i.e Orders  or string messages
        /// </summary>
        public List<object> Items { get; set; }
    }
```

#### Response Model
And We also always respond with the same model.  Check status of success
if you get a false,  You can check the ValidationErrors for property based issues
and you can check the ErrorMessage property for other message types.

The Errors in ErrorMessage are detailed in ApiErrorMessages model.
The ValidationErrors will be subject to change, and are not yet set as const and in a model.
As soon as validation is settled.  I will create a constants for this, and update the library.

```cs
    public class ApiResponse
    {
        /// <summary>
        /// True if all ok, false if errorMessage needs to be checked
        /// </summary>
        public bool Success { get; set; }

        /// <summary>
        /// An error message, if Success = false
        /// from ApiErrorMessages object
        /// </summary>
        public string ErrorMessage { get; set; }

	/// <summary>
        /// List errors from property validation
        /// </summary>
        public List<ValidationError> ValidationErrors { get; set; }

        /// <summary>
        /// Object of a single item.  be that of class, string int  etc
        /// used on getdata
        /// </summary>
        public object Item { get; set; }

        /// <summary>
        /// List of items to send. i.e Orders  or string messages
        /// used on getdata
        /// </summary>
        public List<object> Items { get; set; }
    }

    public class ValidationError
    {
	/// <summary>
	/// The name of the property.
	/// </summary>
	public string PropertyName { get; set; }

	/// <summary>
	/// The error message
	/// </summary>
	public string ErrorMessage { get; set; }

	/// <summary>
	/// The property value that caused the failure.
	/// </summary>
	public object AttemptedValue { get; set; }
   }
```



## The Streetwise.Api.Connect.Models

In the request model, you will return an object or many objects in the item or items fields
as it stands we are covering order information.  Below is the model you will use and set to the item property
I have also included a breakdown of the types used in that model for clarity.

#### Streetwise.Api.Connect.Models.OnlineOrders
```cs
    /// <summary>
    /// Basket / Online order information model
    /// </summary>
    public class OnlineOrderDetail
    {
        /// <summary>
        /// OnlineOrder object
        /// </summary>
        public OnlineOrder OrderDetails { get; set; }
        
        /// <summary>
        /// List of OnlineOrderItems
        /// </summary>
        public ICollection<OnlineOrderItems> OrderItems { get; set; }

        /// <summary>
        /// Delivery address for this order
        /// </summary>
        public OnlineOrderDeliveryAddress DeliveryAddress { get; set; }

    }
```

#### Streetwise.Api.Connect.Models.OnlineOrder
```cs
public class OnlineOrder : BaseIntIdModel
    {
        /// <summary>
        /// The location order be dispatched from
        /// </summary>
        public string LocationCode { get; set; }
        
        /// <summary>
        /// The order number / NOP Order GUID
        /// </summary>
        public string OrderNo { get; set; }
        
        /// <summary>
        /// The store members code
        /// </summary>
        public int MemberCode { get; set; }
        
        /// <summary>
        /// The date the order was created
        /// </summary>
        public DateTime OrderDate { get; set; }
        
        /// <summary>
        /// Customer GUID in nop   or ID to string in others
        /// </summary>
        public string CustomerGuid { get; set; }
        
        /// <summary>
        /// Ability to extend with notes or other messages
        /// use the field for delivery notes too
        /// </summary>
        public string Notes { get; set; }

        /// <summary>
        /// If customer is actually a member of staff
        /// </summary>
        public bool IsStaffMember { get; set; }

        /// <summary>
        /// Delivery charge, if none set 0
        /// </summary>
        public decimal DeliveryCharge { get; set; }

        /// <summary>
        /// The total and final value for the order. i.e Amount customer has paid
        /// </summary>
        public decimal TotalOrderValue { get; set; }
    }
``` 

#### Streetwise.Api.Connect.Models.OnlineOrderItems
```cs
public class OnlineOrderItems : BaseIntIdModel
    {
        /// <summary>
        /// The order number field in OnlineOrders   NOP order GUID
        /// </summary>
        public string OrderNo { get; set; }

        /// <summary>
        /// Streetwise Product Code
        /// </summary>
        public string ProductCode { get; set; }
        
        /// <summary>
        /// Qty requested in the order row
        /// </summary>
        public int QtyRequired { get; set; }
                        
        /// <summary>
        /// Price the customer paid
        /// </summary>
        public decimal PurchasePrice { get; set; }
        
        /// <summary>
        /// The ID of the order row item, in relation to sending store
        /// </summary>
        public string OrderRowId { get; set; }
        
        /// <summary>
        /// The price it would normally sell for
        /// </summary>
        public decimal StandardSellingPrice { get; set; }
        
        /// <summary>
        /// The promotion used on this order item
        /// </summary>
        public string PromitionId { get; set; }
    }
```

#### Streetwise.Api.Connect.Models.OnlineOrderDeliveryAddress

```cs
public class OnlineOrderDeliveryAddress : BaseIntIdModel
    {
        /// <summary>
        /// Gets or sets the first name
        /// </summary>
        public string FirstName { get; set; }

        /// <summary>
        /// Gets or sets the last name
        /// </summary>
        public string LastName { get; set; }

        /// <summary>
        /// Gets or sets the email
        /// </summary>
        public string Email { get; set; }

        /// <summary>
        /// Gets or sets the company
        /// </summary>
        public string Company { get; set; }
        
        /// <summary>
        /// Gets or sets the county
        /// </summary>
        public string County { get; set; }

        /// <summary>
        /// Gets or sets the city
        /// </summary>
        public string City { get; set; }

        /// <summary>
        /// Gets or sets the address 1
        /// </summary>
        public string Address1 { get; set; }

        /// <summary>
        /// Gets or sets the address 2
        /// </summary>
        public string Address2 { get; set; }

        /// <summary>
        /// Gets or sets the zip/postal code
        /// </summary>
        public string ZipPostalCode { get; set; }

        /// <summary>
        /// Gets or sets the country
        /// </summary>
        public string Country { get; set; }

        /// <summary>
        /// Gets or sets the phone number
        /// </summary>
        public string PhoneNumber { get; set; }       
    }
```

## Validation

Note that we sanitize all data coming into the API.
the general rule is this regex: 

```cs
@"[^\w\.@-]"
```

It allows letters, number and normal punctuation like spaces.   It is also extended to allow @ . and -
We do not warn or validate if the property contains an unwanted char, we just sanitize. It is up to the sender
to make sure that data is not going to loose context, by having invalid charactors.

It is also worth a mention that any email address fields, will be checked, and as such, if the email field
is not null then the value of that field must validate as an email address string. We will warn about this.

It would be best practice to log all failed responses from the API, in order to ensure that a checker can be created
on your side, in order to make sure that no orders have been missed.  If the order is corrected, You can re-post for re-validation.

So it would also be prundent to add in the facility to re-send / re-try an order notification.
