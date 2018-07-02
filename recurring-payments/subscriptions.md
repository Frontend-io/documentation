# Subscriptions

Subscriptions allow customers to pay a specific amount every hour, day, week, month, or year depending on the recurring interval you set. With subscriptions, you only need to initialize the first payment, and Paystack will handle the renewals when they are due.

Here is how to set up a subscription:

- Step 1: Create a Plan
- Step 2: Initiate a subscription payment
- Step 3: Listen for subscription payments

## Create a Plan

A **plan** is a framework for a subscription. This is where you decide the amount, interval of the subscription and currency in which the subscription will be paid. Every subscription needs to be on a plan.

```
curl https://api.paystack.co/plan \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"name": "Monthly retainer", "interval": "monthly", "amount": 500000}' \
-X POST
```

You can also create a Plan from the [Paystack Dashboard](https://dashboard.paystack.com/#/plans).

When a plan is created, the response contains a **plan code** that is used to create a subscription.

The plan intervals we have are `daily`, `weekly`,  `monthly`, `quarterly` and `annually`. You can also set invoice limits to determine how many times a user can be charged for this plan. If you set `invoice_limit: 5` to the request above, the user will be charged only 5 times.

## Initiate a subscription payment

There are 2 ways to create a subscription:

The first way is to include the plan code in the [transaction initialization](https://developers.paystack.co/docs/recievingmoney) object when the user is about to make the first payment. So your transaction initialization object will look like:

```
{
    "email" : "customer@email.com",
    "reference": "your_transaction_reference"
    "plan": "PLN_sampleplancode"
}
```

Here are some sample code

_In Paystack Inline_

```javascript
<form >
  <script src="https://js.paystack.co/v1/inline.js"></script>
  <button type="button" onclick="payWithPaystack()"> Pay </button> 
</form>

<script>
var paymentForm = document.getElementById('paymentForm');
paymentForm.addEventListener("submit", payWithPaystack, false);

function payWithPaystack(e) {  
  e.preventDefault();
  var config = {
    key: 'pk_test_xxxxxxxxxxxxxx', // Replace with your public key
    email: document.getElementById("email-address").value,
    amount: document.getElementById("amount").value * 100,
    .
    .
    .
    plan: 'PLN_xxxxxxxxxxx'
    onClose: function(){
      alert('Window closed.');
    },
    callback: function(response){
      var message = 'Payment complete! Reference: ' + response.reference;
      alert(message);
    }
  };
  
  var paystackPopup = new Popup(config);
  paystackPopup.open();
}
```

A second way to create subscriptions is to run the [create subscription endpoint](https://developers.paystack.co/v2.0/reference#create-subscription) via the Paystack API. This can only be used when the customer must have paid on your business before as it requires a `customer` code and `authorisation`. If an `authorisation` exists but is not passed as a parameter, Paystack picks the most recent `authorization` to charge.

```
curl https://api.paystack.co/subscription \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"customer": "CUS_xnxdt6s1zg1f4nx", "plan": "PLN_gx2wn530m0i3w3m"}' \
     -X POST
```
The response returned include `subscription_code`, `next_payment_date` and `email_token`. The email token and subscription are used to [cancel the subscription](https://developers.paystack.co/v2.0/reference#disable-subscription).

Subscriptions are not retried. When a payment attempt fails, it will not be attempted again. This makes subscriptions more ideal for situations where value is delivered after payment such as payment for internet service or a streaming service. Where if payment fails, value is held till customer pays successfully via a one-time payment.

## Listen for subscription payments

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