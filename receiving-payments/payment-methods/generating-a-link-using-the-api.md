# Generating a link using the API

This payment method provides an authorization URL which can be used to complete a payment. It is implemented by directly calling the [initialize transaction](https://developers.paystack.co/v2.0/reference#initialize-a-transaction) endpoint. The authorization URL can only be used for one transaction so you will need to generate a new authorization url for a new transaction.

When the payment is successful, we will call your callback URL (as setup in your dashboard or while initializing the transaction) and return the reference sent in the first step as a query parameter.

If you use a test secret key, we will call your test callback url, otherwise, we'll call your live callback url.

To implement this payment method,

- [Step 1: Collect customer information](https://developers.paystack.co/v2.0/docs/paystack-redirect#section-collect-customer-information)
- [Step 2: Initialize transaction](https://developers.paystack.co/v2.0/docs/paystack-redirect#section-initialize-transaction)
- [Step 3: Verify transaction](https://developers.paystack.co/v2.0/docs/verifying-transaction)
- Step 4: Handle webhook

## Prerequisites
Confirm that your server can conclude a TLSv1.2 connection to Paystack's servers. Most up-to-date software have this capability. Contact your service provider for guidance if you have any SSL errors.

## Collect customer information
`email` and `amount` are the required parameters. Send a unique email per customer. The amount we accept on all endpoints are in kobo and must be an integer value. For instance, to accept 456 naira, 78 kobo, please send 45678 as the `amount`.  

<p class="callout info"><b>Tip:</b>
If you don’t collect email addresses from your customers, you can generate an email address by combining the information you have with your website url. For example, if you only collect phone numbers from your customers, you can create email addresses like 08030000000@mybusiness.com.</p>

If you service is one where users are logged in and you already have user information, you can pass email address and amount to Paystack's servers for a charge from your database or user session. If it is not, you can create a form where the customer information is collected.

```
<form action="/save-order-and-pay" method="POST">
 <input type="hidden" name="email_prepared_for_paystack" value="<?php echo $email; ?>"> 
 <input type="hidden" name="amount" value="<?php echo $amount_in_kobo; ?>"> 
 <input type="hidden" name="cartid" value="<?php echo $cartid; ?>"> 
 <button type="submit" name="pay_now" id="pay-now" title="Pay now">Pay now</button>
</form>
```

## Initialize transaction
When a customer clicks the "Pay Now" button, save the order details on your server and initialize a transaction by making a POST request to our API. Pass the email, amount and any other parameters to the [initialize endpoint](https://developers.paystack.co/v2.0/reference#initialize-a-transaction). 

If the API call is successful, we will return an authorization URL which you will redirect to for the customer to input their payment information to complete the transaction.

```
curl https://api.paystack.co/transaction/initialize \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com"}' \
-X POST
```
If the transaction is successful, Paystack will

   - Redirect back to a `callback_url` set when initializing the transaction or on your dashboard at: <https://dashboard.paystack.co/#/settings/developer> . If neither is set, Customers see a "Transaction was successful" message.
   - Send a `charge.success` event to your Webhook URL set at: <https://dashboard.paystack.co/#/settings/developer>
   - If receipts are not turned off, a HTML receipt will be sent to the customer's email.

Before you give value to the customer, please make a server-side call to our verification endpoint (which should be your callback URL) to confirm the status and properties of the transaction.