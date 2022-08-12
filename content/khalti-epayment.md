# Web Checkout (New E-Payment Version)

This documentation details the process of implementing the latest e-Payment Checkout platform released by Khalti.The latest version is accessible through [pay.khalti.com](https://pay.khalti.com)

<!-- ## Demo
To get the feel of how Khalti checkout looks click the button below.

<button id="payment-button" style="background-color: #5C2D91; cursor: pointer; color: #fff; border: none; padding: 5px 10px; border-radius: 2px">Pay with Khalti</button>
<button id="ebanking-button" style="background-color: #FAA61A; cursor: pointer; color: #fff; border: none; padding: 5px 10px; border-radius: 2px">Pay with Ebanking</button> -->

## How it works?  


- User visits the merchant's website to make some purchase
- A unique `purchase_order_id` is generated at merchant's system
- Payment request is made to Khalti providing the `purchase_order_id`, `amount` in paisa and `return_url`
- User is redirected to the epayment portal (eg. https://pay.khalti.com)
- After payment is made by the user, a successful callback is made to the `return_url`
- The merchant website can optionally confirm the payment received
- The merchant website then proceeds other steps after payment confirmation

## Getting Started

There is no special installation plugin or SDK required for this provided you are able to make a POST request from your web application. However, we shall come up with handy plugins in coming days.

> A merchant account is required if you haven't created at. 

> __For sandbox access__ <br />Signup from [https://a.khalti.com/join/merchant/](https://a.khalti.com/join/merchant/) as a merchant

> __For Production access__ <br />Please visit [https://admin.khalti.com](https://admin.khalti.com)

## API Authorization
HTTP Authorization for api requests is done using Auth Keys. Auth Key must be passed in the  header for authorization in the following format 

```
{
 	"Authorization": "Key <LIVE_SECRET_KEY>"  
}  
```

> PS : Use `live_secret_key` from __a.khalti.com__ during sandbox testing and use `live_secret_key` from __khalti.com__ for production environments

## API Endpoints
> __Live__ <br /> [https://khalti.com/api/v2/](https://khalti.com/api/v2/)

> __Sandbox__ <br /> [https://a.khalti.com/api/v2/](https://a.khalti.com/api/v2/)

## Initiating a Payment request
Every payment request should be first initiated from the merchant as a server side `POST` request. Upon success, a unique request identifier is provided called `pidx` that should be used for any future references

| URL | Method | Authorization | Format |
| --- | --- | --- | --- |
| /epayment/initiate/ | POST | Required | application/json |

### JSON Payload Details
| Field |  Required |  Description | 
| --- | --- | --- |
| return_url | Yes |  <ul> <li>Landing page after the transaction. </li><li>Field must contain a URL.</li></ul> | 
| website_url | Yes | <ul> <li>The URL of the website.  </li><li>Field must contain a URL.</li></ul> |
| amount | Yes | <ul> <li>Total payable amount excluding the service charge. </li><li> Amount must be passed in Paisa </li></ul> | 
| purchase_order_id | Yes | Unique identifier for the transaction generated by merchant
| purchase_order_name | Yes | This is the name of the product. 
| customer_info | No | This field represents to whom the txn is going to be billed to. 
| amount_breakdown | No | Any number of labels and amounts can be passed. 
| product_details | No | No of set is unlimited

### Sample Request Payload
```
{
  "return_url": "https://example.com/",
  "website_url": "https://example.com/",
  "amount": 1300,
  "purchase_order_id": "test12",
  "purchase_order_name": "test",
  "customer_info": {
      "name": "Ashim Upadhaya",
      "email": "example@gmail.com",
      "phone": "9811496763"
  },
  "amount_breakdown": [
      {
          "label": "Mark Price",
          "amount": 1000
      },
      {
          "label": "VAT",
          "amount": 300
      }
  ],
  "product_details": [
      {
          "identity": "1234567890",
          "name": "Khalti logo",
          "total_price": 1300,
          "quantity": 1,
   "unit_price": 1300
      }
  ]
}
```

### Success Response
```
{
   "pidx": "S8QJg2VALZGTJRkKqVxjqB",
   "payment_url": "https://test-pay.khalti.com/?pidx=S8QJg2VALZGTJRkKqVxjqB/"
}
```

After getting the success response, the user should be redirected to the `payment_url` obtained in the success response.

### Error Responses
__return_url is blank__
```
{
   "return_url": [
       "This field may not be blank."
   ],
   "error_key": "validation_error"
}
```

__return_url is invalid__
````
{
   "return_url": [
       "Enter a valid URL."
   ],
   "error_key": "validation_error"
}
````
__website_url is blank__
```
{
   "website_url": [
       "This field may not be blank."
   ],
   "error_key": "validation_error"
}
```

__website_url is invalid)__
```
{
   "website_url": [
       "Enter a valid URL."
   ],
   "error_key": "validation_error"
}
```

__Amount is less than 10__
```
{
   "amount": [
       "Amount should be greater than Rs. 1, that is 100 paisa."
   ],
   "error_key": "validation_error"
}
```

__Amount is invalid__
```
{
   "amount": [
       "A valid integer is required."
   ],
   "error_key": "validation_error"
}
```

__purchase_order_id is blank__
```
{
   "purchase_order_id": [
       "This field may not be blank."
   ],
   "error_key": "validation_error"
}
```

__purchase_order_name is blank__
```
{
   "purchase_order_name": [
       "This field may not be blank."
   ],
   "error_key": "validation_error"
}
```

__Amount breakdown doesn't total to the amount passed__
```
{
   "amount": [
       "Amount Breakdown mismatch."
   ],
   "error_key": "validation_error"
}
```

<!-- **Additionally** Configuration also accepts attribute starting with `merchant_` that can be used to pass additional (meta) data.

- `merchant_name`: This is merchant name

- `merchant_extra`: This is extra data

The additional data starting with `merchant_` is returned in success response payload. -->

## Payment Success Callback
After the user completes the payment, the success response is obtained in the return URL specified during payment initiate.
Sample of success response return URL. 

- The callback url `return_url` should support `GET` method
- User shall be redirected to the `return_url` with following parameters for confirmation
    + pidx - _The initial payment identifier_
    + transaction_id - _The transaction identifier at Khalti after successful payment
    + amount - _Amount paid in paisa_
    + mobile - _Payer Mobile_
    + purchase_order_id - _The initial purchase_order_id provided during payment initiate_
    + purchase_order_name - _The initial purchase_order_name provided during payment initiate_
- There is no further step required to complete the payment, however merchant can process with their own validation and confirmation steps if required
- It's recommended that during implementation, payment lookup API is checked for confirmation after the redirect callback is received

### Sample Callback Request
```
https://example.com/?pidx=EwGKrbdaYLTQ4rmWtNAMEJ
	&amount=1300
	&mobile=98XXXXX403
	&purchase_order_id=test12
	&purchase_order_name=test
	&transaction_id=MJbBJDKYziWqgvkgjxhS2W
```

## Payment Verification (Lookup)
After a callback is received, You can use the `pidx` provided earlier, to lookup and reassure the payment status.

| URL | Method | Authorization | Format | 
| -- | -- | -- |  -- | 
/epayment/lookup/ |  POST | Required | application/json

#### Request Data
`
{
   "pidx": "HT6o6PEZRWFJ5ygavzHWd5"
}
`
#### Success Response
```
{
   "pidx": "HT6o6PEZRWFJ5ygavzHWd5",
   "total_amount": 1000,
   "status": "Completed",
   "transaction_id": "GFq9PFS7b2iYvL8Lir9oXe",
   "fee": 0,
   "refunded": false
}
```
 
#### Failed / Pending  Response
```
{
   "pidx": "HT6o6PEZRWFJ5ygavzHWd5",
   "total_amount": 1000,
   "status": "Pending",
   "transaction_id": "GFq9PFS7b2iYvL8Lir9oXe",
   "fee": 0,
   "refunded": false
}
```

#### Refunded  Response
```
{
   "pidx": "HT6o6PEZRWFJ5ygavzHWd5",
   "total_amount": 1000,
   "status": "Refunded",
   "transaction_id": "GFq9PFS7b2iYvL8Lir9oXe",
   "fee": 0,
   "refunded": true
}
```
 
#### Expired Response 
```
{
   "pidx": "H889Er9gq4j92oCrePrDwf",
   "total_amount": 1000,
   "status": "Expired",
   "transaction_id": null,
   "fee": 0,
   "refunded": false }
```
> PS : Links expire in 60 minutes in production


### Lookup Payload Details
| Field | Description | -- 
| -- | -- | -- | 
| pidx | This is the payment id of the transaction. 
| total_amount | This is the total amount of the transaction
| status | `Completed` - Transaction is success <br />`Pending` - Transaction is failed or is in pending state <br />`Refunded` - Transaction has been refunded
| transaction_id | This is the transaction id for the transaction. <br />This is the unique identifier. 
| fee | The fee that has been set for the merchant.
| refunded | `True` - The transaction has been refunded. <br />`False` - The transaction has not been refunded.

## Generic Errors
#### When an incorrect Authorization key is passed. 
`{
   "detail": "Invalid token.",
   "status_code": 401
}`

#### If incorrect payment_id is passed. 
`{
   "detail": "Not found.",
   "error_key": "validation_error"
}`


&nbsp; 