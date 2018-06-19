# Events

Whenever actions are carried out on your Paystack account, we trigger POSt events to your application. These actions include when a payment is successful, when a transfer is successful or fails, when a subscription is created, renewed or disabled, etc. We will list all the actions that trigger a webhook event in a section below.

### Why You Need This

Events provide a way for Paystack servers to communicate directly with your server whenever something significant happens. This is very important because for the payment and transfers actions, your API calls may not get the final statuses at the time the calls were made. When a customer is making a payment, an error can that causes a disconnection or the user's application or device may shut down or, in the case of transfers, the status can stay at pending. Paystack sends events to your webhook URL once there is a final status. Actions like subscriptions and invoices happen outside of your application. For example when a customer's subscription is due or when a customer pays an invoice, Paystack charges the customer's card without any activity from your application. Webhooks allows your application to be notified when these actions occur so your application can update the user's or transaction information. Without webhooks, Paystack will not be able to communicate to your server when these actions happen.

### Setting Up Webhooks

A webhook has to be a route/script on your server that can receive unauthenticated http POST requests. This means that you have to ensure the route does not require any form of authentication or token to be reached. If your environment restricts access to external IPs, you can whitelist the following Paystack IPs from which the events will be coming: 

-  52.31.139.75
- 52.49.173.169
- 52.214.14.220

When you have setup this route on your server, copy the URL to this route or script and [paste them in the webhook fields on your Paystack Dashboard Developer Settings page][https://dashboard.paystack.com/#/settings/developer]. Paystack requires you to set up test webhook URL for test transactions and live webhook URL for live transactions, so please ensure you set them up accordingly on the dashboard. 

*Note: Events cannot be sent to your localhost server. Events can only be sent to a live-hosted server.*

### Receiving Paystack Events

As mentioned above, the events are sent as http POST requests. Since the route is open and anybody can send events to it, we hash the event body using the secret key of the transaction domain (test secret key for test transactions and live secret key for live transactions). We use Hmac Sha512 hash, and the value is stored in `x-paystack-signature` header in the request. To confirm that the request is from Paystack use the secret key of the transaction domain to hash the body of the request received and compare the result against the `x-paystack-signature` header. If it matches then you can be sure that the event came from Paystack.  See code implementation samples below.

Next you need to respond to the event. You are expected to respond to the event with status `200` at most 10 seconds after it gets to your server. This is to acknowledge to Paystack that you have gotten the event. ***If you do not respond within 10 seconds, or at all, Paystack will resend the event every hour for the next 72 hours***. You shoud respond before attempting to process the event. You can log the event body so that you can reference it later in case something breaks while trying to process the event. 

After this, check the type of event that was sent

Here is a recap of how to handle the event.

1. Make sure the route is expecting a POST request
2. When the request comes in use the secret key of the transaction domain and the event body to generate a HMAC SHA512 hash, and compare that against the `x-paystack-signature` header in the request.
3. If the hash matches, respond to the event with a `200` status immediately. 
4. Log the event.
5. Check what kind of event was sent and process accordingly.

###Code Samples

###Paystack Event Triggers

Here are the events we currently trigger. We would add more to this list as we hook into more actions in the future.

`subscription.create` - A subscription has been created

`subscription.disable` - A subscription on your account has been disabled

`subscription.enable` - A subscription on your account has been enabled

`invoice.create` - An invoice has been created for a subscription on your account. This usually happens 3 days before the subscription is due or whenever we send the customer their first pending invoice notification

`invoice.update` - An invoice has been updated. This usually means we were able to charge the customer successfully. You should inspect the invoice object returned and take necessary action

`charge.success` - A successful charge was made.

`transfer.success` - A successful transfer has been completed.

`transfer.failed` -  A transfer you attempted has failed.



### Sample Events

**Transaction**

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
      "log":{  
         "time_spent":16,
         "attempts":1,
         "authentication":"pin",
         "errors":0,
         "success":false,
         "mobile":false,
         "input":[  

         ],
         "channel":null,
         "history":[  
            {  
               "type":"input",
               "message":"Filled these fields: card number, card expiry, card cvv",
               "time":15
            },
            {  
               "type":"action",
               "message":"Attempted to pay",
               "time":15
            },
            {  
               "type":"auth",
               "message":"Authentication Required: pin",
               "time":16
            }
         ]
      },
      "fees":null,
      "customer":{  
         "id":68324,
         "first_name":"BoJack",
         "last_name":"Horseman",
         "email":"bojack@horseman.com",
         "customer_code":"CUS_qo38as2hpsgk2r0",
         "phone":null,
         "metadata":null,
         "risk_action":"default"
      },
      "authorization":{  
         "authorization_code":"AUTH_f5rnfq9p",
         "bin":"539999",
         "last4":"8877",
         "exp_month":"08",
         "exp_year":"2020",
         "card_type":"mastercard DEBIT",
         "bank":"Guaranty Trust Bank",
         "country_code":"NG",
         "brand":"mastercard"
      },
      "plan":{}
   }
}
```

**Transfer Success**:

```
{
  event: "transfer.success",
  data: {
    domain: "live",
    amount: 10000,
    currency: "NGN",
    source: "balance",
    source_details: null,
    reason: "Bless you",
    recipient: {
      domain: "live",
      type: "nuban",
      currency: "NGN",
      name: "Someone",
      details: {
        account_number: "0123456789",
        account_name: null,
        bank_code: "058",
        bank_name: "Guaranty Trust Bank"
      },
      description: null,
      metadata: null,
      recipient_code: "RCP_xoosxcjojnvronx",
      active: true
    },
    status: "success",
    transfer_code: "TRF_zy6w214r4aw9971",
    transferred_at: "2017-03-25T17:51:24.000Z",
    created_at: "2017-03-25T17:48:54.000Z"
  }
}
```

**Transfer Failed**:

```
{
  "event": "transfer.failed",
  "data": {
    "domain": "test",
    "amount": 10000,
    "currency": "NGN",
    "source": "balance",
    "source_details": null,
    "reason": "Test",
    "recipient": {
      "domain": "live",
      "type": "nuban",
      "currency": "NGN",
      "name": "Test account",
      "details": {
        "account_number": "0000000000",
        "account_name": null,
        "bank_code": "058",
        "bank_name": "Zenith Bank"
      },
      "description": null,
      "metadata": null,
      "recipient_code": "RCP_7um8q67gj0v4n1c",
      "active": true
    },
    "status": "failed",
    "transfer_code": "TRF_3g8pc1cfmn00x6u",
    "transferred_at": null,
    "created_at": "2017-12-01T08:51:37.000Z"
  }
}
```

**Subscription**

```
{
  "event": "subscription.create",
  "data": {
    "domain": "test",
    "status": "active",
    "subscription_code": "SUB_vsyqdmlzble3uii",
    "amount": 50000,
    "cron_expression": "0 0 28 * *",
    "next_payment_date": "2016-05-19T07:00:00.000Z",
    "open_invoice": null,
    "createdAt": "2016-03-20T00:23:24.000Z",
    "plan": {
      "name": "Monthly retainer",
      "plan_code": "PLN_gx2wn530m0i3w3m",
      "description": null,
      "amount": 50000,
      "interval": "monthly",
      "send_invoices": true,
      "send_sms": true,
      "currency": "NGN"
    },
    "authorization": {
      "authorization_code": "AUTH_96xphygz",
      "bin": "539983",
      "last4": "7357",
      "exp_month": "10",
      "exp_year": "2017",
      "card_type": "MASTERCARD DEBIT",
      "bank": "GTBANK",
      "country_code": "NG",
      "brand": "MASTERCARD"
    },
    "customer": {
      "first_name": "BoJack",
      "last_name": "Horseman",
      "email": "bojack@horsinaround.com",
      "customer_code": "CUS_xnxdt6s1zg1f4nx",
      "phone": "",
      "metadata": {},
      "risk_action": "default"
    },
    "created_at": "2016-10-01T10:59:59.000Z"
  }
}
```

### Webhook Best Practices

- Ensure that your webhook route is not a protected route or require authentication
- Respond to the webhook before processing. To avoid the webhook being resent multiple times, and also avoid the possibility of a code error responding with a `500` status.
- When you receive any `success` event, always check to ensure that value hasn't been delivered during the API call that triggered the event or by a previous event. 
- Ensure your environment configurations are not blocking or truncating the events.
- Make sure the event body is not preprocessed in anyway before doing the hash check to avoid hash mismatch. Preprocessing may change some parts of the event, like URL encoding, and this will lead to the hash producing different values.