# Receiving Money

## Typical Payment Request.

Here is the typical API request to initialize a transaction.

```
curl https://api.paystack.co/transaction/initialize \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"amount": 500000, "email": "customer@email.com"}' \
-X POST
```

### Payment Parameters

The most important fields in the request body are the amount to be charged and the customer email.

If your application does not require customer emails, you can generate an email for the transaction with any unique identifier from your system like phone number or customer id: 0801245679@yoursite.com or j239dj3hn9393@yourapp.com.

It is helpful if the identifier is unique to the customer for all transactions by the customer so that you can track the customer transaction history on your paystack dashboard.

You can add a reference to the request body which you use to identify this transaction on your own system. If you don't send a reference, Paystack will generate one for you for that transaction. Also transaction references must be unique for each transaction, otherwise you will get a Duplicate transaction error.

The amount values that you pass to Paystack must be in kobo. To convert a Naira value to kobo, multiply by 100. So if you wanted to charge N100, the amount value to send will be 10000.

###Adding Extra Information to the Payment Request

Sometimes you want to pass extra information to your transaction. The initialization request body accepts a metadata object in which you can add other information you want. You can pass the object directly into the metadata. You can also use custom_fields array within the metadata. Custom fields is an array of objects. The values you pass in the custom_fields array will displayed in the transaction details on the Paystack dashboard.

A transaction initialization object with metadata will look like this:

```JSON
{
  email: "me@you.us",
  amount: 100000,
  metadata: {
    account_id: "i923d923",
    reg_number: "101290",
    custom_fields: [
      {
        display_name: "Favourite Movie",
        variable_name: "favourite_movie",
        value: "Pirates of the Carribeans"
      },
      {
        display_name: "Worst Movie",
        variable_name: "worst_movie",
        value: "La la Land"
      }
    ]
  }
}
```



### Response Structure

A typical Paystack Response object (response) contains:

- status
- message
- data

**Status** is true or false depending on if the API call is was processed or not. If you something was wrong with your request (no authorization object, no/wrong secret key, invalid/duplicate transaction reference) the status will be false. The status is NOT the status of the transaction, it is the status of the API call.

If the response status is true, the response object will contain a **data** field. This contains the response for the request you made. The data object has a status field `response->data->status` that contains the status of the transaction.

For example, if a payment is successful, the response status will be true, and the response data status will be success. If the payment fails, the response status will be true and the response data status will be false. The data also contains the rest of the response for the request you made.



### Common Errors

- The most common mistake people make is not including the Authorization Header. All API requests on Paystack are authenticated with your secret key. Format is `Authorization: "Bearer YOUR_SECRET_KEY"`  Don’t forget the `Bearer` keyword
- Amount value is in Kobo. Always remember to multiply the Naira value of the amount by 100. So if you want to charge N100, you will pass amount as `100*100 = 10000`.
- Calls to the Paystack API should not be made from your frontend, only from your server. This is because your secret key should never be exposed publicly. It can be used to access sensitive information on your Paystack account. The public key can only be used to initialize transactions on the Javascript inline.
- Ensure to call the verify function from your callback function (for frontend implementation) or your callback URL/route (for standard redirect implementation). Only when control has been passed to your callback can you be certain that the user has completed payment. If you try to do a long pool to the verify endpoint, you will keep getting abandoned status until the user has completed the payment.

##USECASE  

###Simple online Store



## Payment Channels

Paystack provides various payment channels to allow merchant receive payment from customers.

Currently we have

- Pay with Card
- Pay with bank - this allows users complete payments using their Nigerian bank accounts
- Pay with USSD
- Pay with Ghana mobile money.

The purpose of multiple channels is to allow the users complete payments using any of the channels in case any of them is down.

You can enable or disable any of these Payment channels on your **Dashboard >> Settings >> Preferences**.

### Best Practices

* If you intend to charge recurring payments, it's best to enable only card payments, as other payment channels don't currently support reccuring billing.
* If your transaction amounts are always high, it's best to only allow card payment as other channels may not be able to processs very high transaction amounts. Their failures may affect your payment success rate numbers.  



## Split Payments

###Why?

Paystack has a Split Payments feature that allows you to split a transaction between your main Paystack account and a sub-account using a predefined split percentage, and Paystack automatically settles both the bank account of the main account and the sub-account.

For example, if your business has a multiple products and you want a percentage of payments for each product to go into a account for that product while the rest goes into a central account, you can create a sub-account for each of the products, and include the sub-account code for each product in the transaction for that product.

Another great use case is marketplace sites with multiple vendors. Subaccounts can be used to represent each vendor and their products.

###Sub-accounts 

As the name implies, a sub-account is an account within your main Paystack account that represent a separate class of payment. Whenever a sub-account is created, a sub-account code is generated. Including that sub-account code in any transaction instructs Paystack that this sub-account gets some or all of the payment made for that transaction, as defined in the split percentage while creating the sub-account. Sometimes no money is paid into the sub-account, when the sub-accounts are simply used to separate transactions into different sub-accounts and all the money still goes into same bank account.  On your Paystack Dashboard, you can filter transactions by each sub-account to see all the payments made for each sub-account product.

####**Creating a Subaccount**

You can create a sub-account on [your Paystack dashboard](https://dashboard.paystack.com/#/subaccounts) or [using the API.](https://developers.paystack.co/v1.0/reference#create-subaccount) You can just use the dashboard if you’re only creating a few sub-accounts. If you’ll be needing multiple sub-accounts, say for the marketplace example, then you will need to create a sub-account for each user as they register, in this case you will need to use the create subaccount API. 

Here is what the API request looks like

```
curl https://api.paystack.co/subaccount \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"business_name": "Slay Shoes Store", "settlement_bank": "Access Bank", "account_number": "0728212242", "percentage_charge": 12.5}' \
-X POST
```

The response includes a subaccount code that you can store against the subaccount’s store or product on your database. Whenever someone wants to buy a product associated with that subaccout, you include the subaccount code to the transaction initialization object.

While creating the sub-account you will need to specify 

- the name of the subaccount
- the settlement bank
- settlement account number
- and percentage charge.

#### Split Percentage

The **percentage charge** is how much of the payment the main account is taking from any transaction for that sub-account. If you specify a **percentage charge** of 10%, it means that every time a payment is made for a transaction that includes this subaccount’s code, 10% of the money goes to the main Paystack account and the rest of the money goes into the sub-account’s settlement bank account.

####Including Subaccounts in Transactions

When you want to add a subaccount to a transaction, you simply add the subaccount code to the transaction object like so  `subaccount: "ACCT_4hl4xenwpjy5wb"`.  We are going to use a N2000 transaction to demonstrate, and we will assume that the split percentage is 10% for the main account and 90% for the sub-account. When the transaction is completed, Paystack will settle: N200 to the main account and N1800 to the sub-account.

#####Overriding split percentages with fixed cost

Sometimes you may want to override the split percentage with a fixed amount, such that instead of the having the main account receive a percentage of the payment like 10%, it receives a fixed amount for all transaction irrespective of amount. To do this you need to add a transaction_charge value to the initialization object. The fixed amount value has to be in kobo too. So for a N500 fixed amount, you set transaction_charge: 50000. Using our sample N2000 transaction, this means that N500 goes to the main account and N1500 to the subaccount.

##### Who bears Paystack transaction fees

Finally there is the question of who bears Paystack transaction fees for split payment transactions. By default, Paystack takes the transaction fee from the main account’s share. (Paystack transaction fee is 1.5% of the amount below N2500 and 1.5% + N100 for transactions above N2500). In this N2000 sample transaction, Paystack fee will be N30. 

So by default, using the split percentage for our sample transaction, subaccount will get N1800, main account will get N170 and Paystack will get N30

If you want the sub-account to bear the fees, you can specify that in the initialization object like so: bearer: 'subaccount'.  In this case, subaccount gets N1770, main account gets N200 and Paystack gets N30.

Whichever account is set to bear the Paystack fees, if the fees is greater than the amount meant for the bearer, Paystack takes the fees from the other account. For example, in our sample N2000 transaction, assuming the split percentage is 1% this means that the main account should get N20 and subaccount gets N1980.

But Paystack transaction fee is N30. So if the main Paystack account is supposed to bear the transaction fees, the fees are greater than the amount meant for the main account so we take the transaction fee from the subaccount instead. So Paystack gets N30, main account gets N20, and subaccount gets N1950.

So a typical split payment transaction initialization object will look something:

```
{
  "email": "customer@email.com",
  "amount": 200000,
  "subaccount": "ACCT_4hl4xenwpjy5wb",
  "transaction_charge": 30000,
  "bearer": "subaccount"
}
```



####Settlement

Paystack settles subaccounts automatically everyday into the settlement bank account provided when creating the subaccounts. The amount for the main accounts are settled according to the settlement schedule set on the settlement account (Next day settlement/manual settlement).

####Best Practices

Subaccounts are mostly used by companies that have separate branches/outlets and would like to know which transactions are coming from which outlets. It is not compulsory to split the payments, you can set the split percentage to pay 100% of the payment into the main account, and still use subaccount codes to differentiate the payments.

You cannot use subaccounts created in test mode with your live keys and vice versa.

###USE CASE - MULTIVENDOR STORE

## Recurring Payments

Paystack allows you to charge a card subsequently using our APIs such the user doesn’t need the enter the card details again after a first successful payment. This allows you to debit the user automatically for subcriptions, in-app payments, thrift collections, or simply charge returning customers. A common misconception is that you need to collect and tokenize user’s card details in order to do this. You don’t need to do that. In this section we will explain how to get and use Paystack card authorization tokens.

After a transaction is completed, Paystack generates an authorization token for the payment instrument (customer’s card in this case). That token allows the user to charge that customer for subsequent transactions. The token is returned when you call the verify in an authorization object.

###How to get authorization token for recurring payments

When the first transaction is made (either using inline on the frontend or redirecting using standard), you **verify** the transaction from your server. Part of the response for the [verify endpoint][https://developers.paystack.co/v1.0/reference#verify-transaction] is an authorization object

That authorization object contains the card details the user paid with and an authorization_code. You can use that authorization code to charge the user subsequently

So here is what to do when you verify:

- Check if transaction was successful. The transaction status is inside response object at `response->data->status`
-  Check for the authorization object in `response->data->authorization`. You need to store this authorization object agains the user so you can use it to charge the card subsequently. In the authorization object, check if the payment was made with a card, so `response->data->authorization->channel` should be card. 
- If it is, check if it is `reusable`. Some cards cannot be charged recurrently. So `response->data->authorization->reusable` should be true. 
- Finally, check if you have stored that card against that user before so you don't store it again. So check if any of the card has the same signature. The card signature is at `response->data->authorization->signature`. If it isn't you can now store the whole card against that user.

Other details in the authorization object include the `bank`,`card type`, `last 4 digits`, `expiry date` and `bin`. So when the user wants to pay again, you can display the card for the user as **Access Bank Visa card ending with 1234**

When they select the card for a new transaction (or when you want to charge them subsequently), you then send the authorization_code, user's email and amount you want to charge to the charge_authorization endpoint: <https://developers.paystack.co/v1.0/reference#charge-authorization> and the card will be charged directly.

It’s important to note that to charge the card authorization, you must use the email address that was used for the initial charge. So it is advisable to store each card authorization with the email used to generate it, incase the user changes his email on your system in the future.

## Receiving Subscription Payment

Paystack allows businesses receive subscription payment at intervals - hourly, daily, weekly, monthly, quarterly and annually. To start receiving subscriptions, you need to create a plan.

### Plans

A Plan is the service for which a subscription payment can be made. An example of a plan can be **Monthly Rent Payment** or **Weekly Piano Lessons.** 

You can create a plan on your dashboard or using the API. 

```
curl https://api.paystack.co/plan \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"name": "Monthly retainer", "interval": "monthly", "amount": 500000}' \
-X POST
```

When a Plan is created, Paystack returns a `plan_code`.

###Subscription

A subscription is a payment for a plan. To charge a subscription, you can simply add the plan code to the transaction initialization object like so: plan: `PLN_gx2wn530m0i3w3m` and the user will make the first payment and will be charged automatically subsequently for the plan.

####Trial Period

You may want to offer users a trial period and then charge them subsequently. To do this, you will have to tokenize the customer’s card during registration and use the process described in the **Recurring Payment** section, then when you are ready to start charging the user for the subscription, you send the card token and plan code to the create subscription endpoint.

```
curl https://api.paystack.co/subscription \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"customer": "CUS_xnxdt6s1zg1f4nx", "plan": "PLN_gx2wn530m0i3w3m"}' \
-X POST
```

The response includes a subscription code, the next payment date and an email token. You use the subscription code and email token when you want to cancel the subscription.

####Subscription Renewals

When a subscription is due, Paystack sends a notification email to the customer 3 days to the due date. The email also contains a link to cancel the subscription if the customer wants to opt out. If the subscription is not cancelled before the due date, Paystack charges the customer’s card and sends a event to your webhook url to notify your application of the charge. You can then update the user's account on your end.

### USE CASE - Simple investment app. 