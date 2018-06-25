# Paystack Redirect

Paystack Redirect payment method provides an authorization URL which can be used to complete a payment. It is called by directly calling the [initialize transaction](https://developers.paystack.co/v1.0/reference#initialize-a-transaction) endpoint. The Authorization URL can only be used for one transaction. You will need to generate a new authorization url for a new transaction.

When the payment is successful, we will call your callback URL (as setup in your dashboard or while initializing the transaction) and return the reference sent in the first step as a query parameter.

If you use a test secret key, we will call your test callback url, otherwise, we'll call your live callback url.

## Implementation

1. **Prerequisites**: Confirm that your server can conclude a TLSv1.2 connection to Paystack's servers. Most up-to-date software have this capability. Contact your service provider for guidance if you have any SSL errors.

2. **Prepare your parameters**:

   `email` and `amount` are the most common compulsory parameters. Do send a unique email per customer. If your customers do not provide a unique email, please devise a strategy to set one for each of them. For example, using their phone numbers 08001234567@yoursite.com. The amount we accept on all endpoint are in kobo and must be an integer value. For instance, to accept 456 naira, 78 kobo, please send 45678 as the `amount`.

   ```php
   curl https://api.paystack.co/transaction/initialize \
   -H "Authorization: Bearer SECRET_KEY" \
   -H "Content-Type: application/json" \
   -d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com"}' \
   -X POST
   ```

3. Initialize a transaction by making a POST request to our API. For more info on this, see [initializing transactions](https://developers.paystack.co/v1.0/reference#initialize-a-transaction).

   If the transaction is successful, Paystack will

   - Redirect back to a `callback_url` set when initializing the transaction or on your dashboard at: <https://dashboard.paystack.co/#/settings/developer> . If neither is set, Customers see a "Transaction was successful" message.
   - Send a `charge.success` event to your Webhook URL set at: <https://dashboard.paystack.co/#/settings/developer>
   - If receipts are not turned off, an HTML receipt will be sent to the customer's email.

   Before you give value to the customer, please make a server-side call to our verification endpoint (which should be your callback URL) to confirm the status and properties of the transaction.

4. Handle `charge.success` event: We will post a `charge.success` event to the webhook URL set for your transaction's domain. If it was a live transaction, we will post to your live webhook url and if it is a test transaction, we will post to your test webhook url.

   For more information on this, please see [Events](https://developers.paystack.co/v1.0/docs/events).