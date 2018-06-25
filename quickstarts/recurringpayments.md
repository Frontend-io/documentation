# Recurring Payment and Subscriptions

You can receive recurring payments using Paystack. There are 2 types of recurring payments in Paystack. 

- Charging Authorizations
- Subscriptions

The difference between them is that with **Subscriptions**, Paystack handles the recurrent charges at your Plan's predefined intervals but with **Authorizations**, you control the recurrent charges, and can charge any time you want.

##Charging Authorizations 

An authorization is a representation of a payment instrument that you can use to charge a customer subsequently.

Here are the steps for charging authorization.

1. Perform a transaction
2. Confirm that the authorization is reusable
3. Store the authorization and email against the user
4. Charge the Authorization

### Step 1 - Initiate A Transaction

First you have to initialize a transaction where the user will make a payment for the first time. [If you don't know to initialize a Paystack transaction, learn more here](https://developers.paystack.co/docs/quickstarts/receivingmoney). This first charge is to ensure that the card or account is valid and can be charged. This initial transaction has to be successful. The transaction status is inside response object at `response->data->status`.

When a transaction is completed, you verify the transaction from your server. Part of the response for the verify endpoint is an `authorization` object. It's in the response object at `response->data->authorization`. That `authorization` object contains details of the payment instrument (card or account number) that the user paid with and an `authorization_code`. It is this authorization code that you use to charge the user recurrently.

###Step 2 - Confirm that the Authorization is reusable

Only `reusable` payment instruments can be charged recurrently, so you need to confirm that instrument is reusable.  So `response->data->authorization->reusable` should be `true`.  If it isn't `true`, you cannot charge the authorization.  So if your application needs the authorizations to be charged recurrently, like a loan collection app, then you must ensure that the initial transaction only allows reusable payment channels. You can set that in the custom filters in the `metadata` field of the transaction initialization object. [Learn more about reusable and non-reusable payment channels here](https://developers.paystack.co/docs/guides/receivingpayments#reusable).

### Step 3 - Store Authorization and email against the user

Next, you need to store this authorization object against the user so you can use it to charge the card subsequently. Every payment instrument that is used on your site/app has a unique signature, so you can check if you have stored that payment instrument against the user before so you don't store it again. The signature is at `response->data->authorization->signature`. If this authorization hasn't been stored before, you can store it now.

You also need to store the email used for this transaction along with the authorization object. This is important because only the email used to create an authorization can be used to charge it. If rely on the user's email stored on your system and the user changes it, the authorization can no longer be charged.

Other details in the authorization object include the **bank**,**card type**, **last 4 digits**, **card expiry date** and **bin**. So when the user wants to pay again, you can display the card for the user as **Access Bank Visa card ending with 1234**.

### Step 4 - Charge the Authorization

When they select the card for a new transaction or when you want to charge them subsequently, you then send the `authorization_code`, user's `email` and `amount` you want to charge to the [charge_authorization endpoint]( https://developers.paystack.co/v1.0/reference#charge-authorization) and the instrument will be charged .

```
curl https://api.paystack.co/transaction/charge_authorization \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"authorization_code": "AUTH_72btv547", "email": "bojack@horsinaround.com", "amount": 500000}' \
-X POST
```

If your application needs to charge the authorizations at certain intervals, it means your server needs to have a cron job that runs at particular interval and picks all the authorizations that needs to be charged.

## Subscriptions

Subscription allows you charge users for various subscription plans at set intervals. With subscriptions, you only need to initialize the first payment, and Paystack will handle the renewals when they are due. 

Here are the steps to set up a subscription.

1. Create a Plan
2. Initiate a subscription payment
3. Listen out for renewals 

### Step 1 Create A Plan

To create a subscription plan, send the name of the plan, the amount and the interval to the [Plan endpoint](https://developers.paystack.co/v1.0/reference#create-plan). Here's an API request for creating a subsciption plan.

```
curl https://api.paystack.co/plan \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"name": "Monthly retainer", "interval": "monthly", "amount": 500000}' \
-X POST
```

The response contains a **Plan Code** that is used to create a subscription.

The plan intervals we have are `daily`, `weekly`,  `monthly`, `quarterly` and `annually`. You can also set invoice limits to determine how many times a user can be charged for this plan. If you set `invoice_limit: 5` to the request above, the user will be charged only 5 times.



### Step 2 - Initiate a Subscription Payment

There are 2 ways to do this. 

The first way is to include the plan code in the [transaction initialization](https://developers.paystack.co/docs/recievingmoney) object when the user is about to make the first payment. So your transaction initialization object will look like:

```
{
    "email" : "customer@email.com",
    "reference": "your_transaction_reference"
    "plan": "PLN_sampleplancode"
}
```

The second way assumes that you have already charged the user before with a reusable payment instrument. You can use either the user's authorization code or customer code (both returned when you verify any successful transactions) to create a subscription using the plan code.

```
curl https://api.paystack.co/subscription \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"customer": "CUS_xnxdt6s1zg1f4nx", "plan": "PLN_gx2wn530m0i3w3m"}' \
-X POST
```

The response returned include `subscription_code`, `next_payment_date` and `email_token`. The email token and subscription are used to cancel the subscription.

Here is how to cancel a subscription

```
curl https://api.paystack.co/subscription/disable \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"code": "SUB_vsyqdmlzble3uii", "token": "d7gofp6yppn3qz7"}' \
-X POST
```

### Step 3 Listen Out for Renewals

When a subscription is due, Paystack sends a webhook notification to your webhook URL to notify you of the renewal. We also send events when a subscription is cancelled. [Learn more about Paystack webhook and events here](https://developers.paystack.co/docs/events).

