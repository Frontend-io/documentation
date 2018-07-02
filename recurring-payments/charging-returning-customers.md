# Charging returning customers

Paystack offers a simple way to take recurring debits from cards.  

Once a customer has paid you successfully, we send you an authorization code that you can save and use to send payments anytime you need to charge the customer. 

Here are the steps for charging an authorization to make a recurring debit happen:

- Step 1: Initialize first transaction
- Step 2: Confirm that the authorization is reusable
- Step 3: Store the authorization and email against the user
- Step 4: Charge the authorization

## Initialize first transaction
Initialize a transaction where the user will make a payment for the first time. 

This first charge is to ensure that the card or account is valid and can be charged. This initial transaction has to be successful. The transaction status is inside the response object at response→data→status.

If the transaction is successful, and you have verified the transaction by calling the verify transaction endpoint, part of the response will be an authorization object. 

```
{  
   "status":true,
   "message":"Verification successful",
   "data":{  
      "amount":27000,
      "currency":"NGN",
      "transaction_date":"2016-10-01T11:03:09.000Z",
      "status":"success",
      "reference":"DG4uishudoq90LD",
      "domain":"test",
      "metadata":0,
      "gateway_response":"Successful",
      "message":null,
      "channel":"card",
      "ip_address":"41.1.25.1",
      "log":{  
         "time_spent":9,
         "attempts":1,
         "authentication":null,
         "errors":0,
         "success":true,
         "mobile":false,
         "input":[ ],
         "channel":null,
         "history":[  
            {  
               "type":"input",
               "message":"Filled these fields: card number, card expiry, card cvv",
               "time":7
            },
            {  
               "type":"action",
               "message":"Attempted to pay",
               "time":7
            },
            {  
               "type":"success",
               "message":"Successfully paid",
               "time":8
            },
            {  
               "type":"close",
               "message":"Page closed",
               "time":9
            }
         ]
      },
      "fees":null,
      "authorization":{  
         "authorization_code":"AUTH_8dfhjjdt",
         "card_type":"visa",
         "last4":"1381",
         "exp_month":"08",
         "exp_year":"2018",
         "bin":"412345",
         "bank":"TEST BANK",
         "channel":"card",
         "signature": "SIG_idyuhgd87dUYSHO92D",
         "reusable":true,
         "country_code":"NG"
      },
      "customer":{  
         "id":84312,
         "customer_code":"CUS_hdhye17yj8qd2tx",
         "first_name":"BoJack",
         "last_name":"Horseman",
         "email":"bojack@horseman.com"
      },
      "plan":"PLN_0as2m9n02cl0kp6"
   }
}
```

The authorization object contains details of the payment instrument that the user paid with in which there is an authorization_code. It is this authorization code that you will use to charge the user recurrently.

We understand that some use-cases do not require an initial charge, (i.e when you want a customer to just add a card), but we strongly suggest that you make an initial of about NGN 50 charge since zero debits are not supported yet. 

## Confirm that the authorization is reusable
Only payment instruments that are tagged `reusable` can be charged recurrently, so you need to confirm that instrument is reusable before attempting a recurring charge on it.  You can find if a payment can recur by checking if `response→data→authorization→reusable` is `true`.  If it isn't `true`, you cannot charge the authorization.  

So if your application https://dash.readme.io/project/paystack/v2.0/docs/recurring-debitsneeds the authorizations to be charged recurrently e.g a loan collection app, then you must ensure that the initial transaction only allows reusable payment channels. You can set that in the custom filters in the `metadata` field of the transaction initialization object.

## Store the authorization and email against the user
Next, you need to store this authorization object against the user so you can use it to charge the card subsequently. Every payment instrument that is used on your site/app has a unique signature, so you can check if you have stored that payment instrument against the user before so you don't store it again. The signature is at `response→data→authorization→signature`. If this authorization hasn't been stored before and is reusable, it means it is a new payment instrument and you can save it in your database.

You also need to store the email used for this transaction along with the authorization object. This is important because only the email used to create an authorization can be used to charge it. If you rely on the user's email stored on your system and the user changes it, the authorization can no longer be charged.

Other details in the authorization object include the **bank**,**card type**, **last 4 digits**, **card expiry date** and **bin**. When you have the whole authorization object saved, you can display customer payment details at the point of payment to charge recurrently. For example, when the user wants to pay again, you can display the card for the user as **Access Bank Visa card ending with 1234**.

## Charge the Authorization

When they select the card for a new transaction or when you want to charge them subsequently, you then send the `authorization_code`, user's `email` and `amount` you want to charge to the [charge_authorization endpoint]( https://developers.paystack.co/v2.0/reference#charge-authorization) and the instrument will be charged .

```
curl https://api.paystack.co/transaction/charge_authorization \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"authorization_code": "AUTH_72btv547", "email": "bojack@horsinaround.com", "amount": 500000}' \
-X POST
```

If your application needs to charge the authorizations at certain intervals, it means your server needs to have a cron job that runs at particular interval and picks all the authorizations that needs to be charged.