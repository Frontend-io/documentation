# Subscriptions

Subscriptions allow customers to pay a specific amount every hour, day, week, month, or year depending on the recurring interval you set. With recurring payments on Paystack, the customer enters their card details once, and they're dutifully debited without them having to input their card details in future.

## Implementation

1. [Create a plan](https://developers.paystack.co/v1.0/reference#create-plan): A **plan** is a framework for a subscription. This is where you decide the amount, interval of the subscription and currency in which the subscription will be paid. Every subscription needs to be on a plan.

   ```
   curl https://api.paystack.co/plan \
   -H "Authorization: Bearer SECRET_KEY" \
   -H "Content-Type: application/json" \
   -d '{"name": "Monthly retainer", "interval": "monthly", "amount": 500000}' \
   -X POST
   ```

   You can also create a Plan from the [Paystack Dashboard](https://dashboard.paystack.com/#/plans).

2. [Create a subscription](https://developers.paystack.co/v1.0/reference#create-subscription): Subscriptions can be created in two ways. 

   - The most popular method is to create a subscription by simply passing in a plan code as a parameter. 

     _In Paystack Inline_

     ```javascript
     <form >
       <script src="https://js.paystack.co/v1/inline.js"></script>
       <button type="button" onclick="payWithPaystack()"> Pay </button> 
     </form>
      
     <script>
       function payWithPaystack(){
         var handler = PaystackPop.setup({
           key: 'pk_test_86d32aa1nV4l1da7120ce530f0b221c3cb97cbcc',
           email: 'customer@email.com',
           amount: 10000,
           .
           .
           .
           plan: 'PLN_JIFIJJ',
           callback: function(response){
               alert('success. transaction ref is ' + response.reference);
           },
           onClose: function(){
               alert('window closed');
           }
         });
         handler.openIframe();
       }
     </script>
     ```

     _In Paystack Standard_

     ```curl
     curl https://api.paystack.co/transaction/initialize \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com", "plan": "PLN_JIFIJJ"}' \
     -X POST
     ```

   - A second way to create subscriptions is to run the [create subscription endpoint](https://developers.paystack.co/v1.0/reference#create-subscription) via the Paystack API. This can only be used when the customer must have paid on your business before as it requires a `customer` code and `authorisation`. If an `authorisation` exists but is not passed as a parameter, Paystack picks the most recent `authorization.

     ```
     curl https://api.paystack.co/subscription \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"customer": "CUS_xnxdt6s1zg1f4nx", "plan": "PLN_gx2wn530m0i3w3m"}' \
     -X POST
     ```

## Other Information

Subscriptions are not retried. When a payment attempt fails, it will not be attempted again. This makes subscriptions more ideal for situations where value is delivered after payment such as payment for internet service or a streaming service. Where if payment fails, value is held till customer pays successfully via a one-time payment.

Subscriptions are disabled via the [Disable Subscription](https://developers.paystack.co/v1.0/reference#disable-subscription) endpoint. Customers are also sent emails before a subscription is charged to allow them cancel if they so wish.

```
curl https://api.paystack.co/subscription/disable \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"code": "SUB_vsyqdmlzble3uii", "token": "d7gofp6yppn3qz7"}' \
-X POST
```

[Events](https://developers.paystack.co/v1.0/docs/events) are used to track subscriptions. When a subscription is created, a *create.subscription* event is sent to your webhook URL. To track subscription payments, watch for the *charge.success* event sent for successful subscriptions.

```
{  
   "event":"charge.success",
   "data":{  
      "id":302961,
      "domain":"live",
      "status":"success",
      "reference":"qTPrJoy9Bx",
      "amount":10000,
      "message":null,
      "gateway_response":"Approved by Financial Institution",
      "paid_at":"2016-09-30T21:10:19.000Z",
      "created_at":"2016-09-30T21:09:56.000Z",
      "channel":"card",
      "currency":"NGN",
      "ip_address":"41.242.49.37",
      "metadata":0,
      "fees":null,
      "customer": {
            "id": 902735,
            "first_name": null,
            "last_name": null,
            "email": "customer@email.com",
            "customer_code": "CUS_1psu1flkeh1dzm8",
            "phone": null,
            "metadata": null,
            "risk_action": "default"
        },
        "plan": "PLN_tq8ur7pqz80fbpi",
        "paidAt": "2018-06-10T18:00:25.000Z",
        "createdAt": "2018-06-10T17:59:59.000Z",
        "transaction_date": "2018-06-10T17:59:59.000Z",
        "plan_object": {
            "id": 6743,
            "name": "Test Plans",
            "plan_code": "PLN_tq8ur7pqz80fbpi",
            "description": "This is to test listing plans, etc etc",
            "amount": 200000,
            "interval": "hourly",
            "send_invoices": true,
            "send_sms": true,
            "currency": "NGN"
        },
        "subaccount": {}
   }
}
```

